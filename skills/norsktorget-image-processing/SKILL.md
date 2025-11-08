---
name: norsktorget-image-processing
description: Use when working with image operations in the Norsktorget project, especially when integrating with Gemini Vision API. This skill provides best practices for image pre-processing, optimization for AI analysis, caching strategies, and efficient image handling in Flutter.
---

# Image Processing for Norsktorget

## Overview

This skill defines the best practices for image processing in the Norsktorget project, with special focus on optimizing images for Gemini Vision API analysis and efficient storage/retrieval.

**Core Principle:** All images MUST be optimized before sending to AI services to minimize costs and improve response times. Local caching MUST be implemented to avoid redundant API calls.

## When to Use

- When implementing image upload and processing for vehicle ads.
- When preparing images for Gemini Vision API analysis.
- When implementing image caching strategies.
- When optimizing image storage in S3 or local storage.
- When handling image compression and format conversion.

## 1. Image Pre-Processing for Gemini Vision API

Before sending images to Gemini, apply these optimizations:

### Required Pre-Processing Steps

```dart
// lib/features/ai/services/image_preprocessor.dart

import 'package:image/image.dart' as img;
import 'dart:typed_data';

class ImagePreprocessor {
  // REQUIRED: Maximum dimensions for Gemini Vision API
  static const int maxWidth = 2048;
  static const int maxHeight = 2048;
  static const int targetQuality = 85;

  /// Optimizes an image for Gemini Vision API analysis
  /// Returns the processed image bytes and a hash for caching
  static Future<ProcessedImage> optimizeForAI(Uint8List imageBytes) async {
    // 1. Decode the image
    final image = img.decodeImage(imageBytes);
    if (image == null) throw Exception('Failed to decode image');

    // 2. Resize if needed (maintain aspect ratio)
    img.Image processed = image;
    if (image.width > maxWidth || image.height > maxHeight) {
      processed = img.copyResize(
        image,
        width: image.width > maxWidth ? maxWidth : null,
        height: image.height > maxHeight ? maxHeight : null,
        interpolation: img.Interpolation.cubic,
      );
    }

    // 3. Convert to JPEG with optimized quality
    final optimizedBytes = Uint8List.fromList(
      img.encodeJpg(processed, quality: targetQuality)
    );

    // 4. Generate hash for caching
    final hash = _generateImageHash(optimizedBytes);

    return ProcessedImage(
      bytes: optimizedBytes,
      hash: hash,
      width: processed.width,
      height: processed.height,
    );
  }

  static String _generateImageHash(Uint8List bytes) {
    // Use a fast hash function for caching
    return bytes.fold(0, (prev, element) => prev + element).toString();
  }
}

class ProcessedImage {
  final Uint8List bytes;
  final String hash;
  final int width;
  final int height;

  ProcessedImage({
    required this.bytes,
    required this.hash,
    required this.width,
    required this.height,
  });
}
```

## 2. Caching Strategy for AI Responses

Implement a two-level caching system:

### Level 1: In-Memory Cache (Fast)

```dart
// lib/features/ai/services/ai_cache_service.dart

class AiCacheService {
  // In-memory cache for current session
  final Map<String, AiAnalysisResult> _memoryCache = {};

  // Cache TTL: 24 hours
  static const Duration cacheTTL = Duration(hours: 24);

  /// Generates a cache key from image hash and analysis type
  String _getCacheKey(String imageHash, String analysisType) {
    return '$imageHash:$analysisType';
  }

  /// Checks if a cached result exists and is still valid
  AiAnalysisResult? getCachedResult(String imageHash, String analysisType) {
    final key = _getCacheKey(imageHash, analysisType);
    final cached = _memoryCache[key];

    if (cached == null) return null;

    // Check if cache is still valid
    if (DateTime.now().difference(cached.timestamp) > cacheTTL) {
      _memoryCache.remove(key);
      return null;
    }

    return cached;
  }

  /// Stores a result in cache
  void cacheResult(String imageHash, String analysisType, AiAnalysisResult result) {
    final key = _getCacheKey(imageHash, analysisType);
    _memoryCache[key] = result;
  }
}
```

### Level 2: Persistent Cache (Hive)

```dart
// lib/features/ai/data/repositories/ai_cache_repository.dart

import 'package:hive_ce/hive_ce.dart';

class AiCacheRepository {
  late Box<AiAnalysisCache> _cacheBox;

  Future<void> init() async {
    _cacheBox = await Hive.openBox<AiAnalysisCache>('ai_cache');
  }

  Future<AiAnalysisResult?> getCachedResult(String imageHash) async {
    final cached = _cacheBox.get(imageHash);

    if (cached == null) return null;

    // Check if cache is still valid (24 hours)
    if (DateTime.now().difference(cached.timestamp) > Duration(hours: 24)) {
      await _cacheBox.delete(imageHash);
      return null;
    }

    return cached.result;
  }

  Future<void> cacheResult(String imageHash, AiAnalysisResult result) async {
    await _cacheBox.put(
      imageHash,
      AiAnalysisCache(
        imageHash: imageHash,
        result: result,
        timestamp: DateTime.now(),
      ),
    );
  }
}
```

## 3. Image Upload and Storage Workflow

### Complete Workflow for Vehicle Photo Upload

```dart
// lib/features/ads/presentation/providers/image_upload_provider.dart

import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'image_upload_provider.g.dart';

@riverpod
class ImageUpload extends _$ImageUpload {
  @override
  AsyncValue<List<UploadedImage>> build() {
    return const AsyncValue.data([]);
  }

  /// Complete workflow: Pick → Optimize → Upload → Analyze
  Future<void> uploadAndAnalyze(String adId) async {
    state = const AsyncValue.loading();

    state = await AsyncValue.guard(() async {
      // 1. Pick image from gallery/camera
      final imagePicker = ref.read(imagePickerProvider);
      final pickedFile = await imagePicker.pickImage(source: ImageSource.gallery);

      if (pickedFile == null) throw Exception('No image selected');

      final rawBytes = await pickedFile.readAsBytes();

      // 2. Pre-process and optimize
      final preprocessor = ImagePreprocessor();
      final processed = await preprocessor.optimizeForAI(rawBytes);

      // 3. Check cache first
      final cacheService = ref.read(aiCacheServiceProvider);
      final cachedAnalysis = cacheService.getCachedResult(processed.hash, 'vehicle');

      AiAnalysisResult analysis;
      if (cachedAnalysis != null) {
        // Use cached result - saves API cost!
        analysis = cachedAnalysis;
      } else {
        // 4. Send to Gemini Vision API
        final geminiService = ref.read(geminiServiceProvider);
        analysis = await geminiService.analyzeVehicleImage(processed.bytes);

        // 5. Cache the result
        cacheService.cacheResult(processed.hash, 'vehicle', analysis);
      }

      // 6. Upload to S3 (in parallel with AI analysis if needed)
      final s3Service = ref.read(s3ServiceProvider);
      final imageUrl = await s3Service.uploadImage(processed.bytes, adId);

      // 7. Return uploaded image with AI metadata
      return [
        ...state.value ?? [],
        UploadedImage(
          url: imageUrl,
          hash: processed.hash,
          aiMetadata: analysis,
        ),
      ];
    });
  }
}
```

## 4. Image Optimization Best Practices

### Required Optimizations

| Aspect | Requirement | Reason |
|:---|:---|:---|
| **Max Dimensions** | 2048x2048 px | Gemini API limit; reduces costs |
| **Format** | JPEG (not PNG) | Smaller file size for same quality |
| **Quality** | 85% | Sweet spot for size vs. quality |
| **Color Space** | RGB | Ensure compatibility |
| **Compression** | Progressive JPEG | Better for web display |

### Cost Optimization Formula

```dart
// Estimated cost reduction from optimization
// Before optimization: ~5MB image = higher API cost
// After optimization: ~300KB image = 94% cost reduction

class CostOptimizer {
  static double estimateCostReduction(int originalSize, int optimizedSize) {
    return ((originalSize - optimizedSize) / originalSize) * 100;
  }
}
```

## 5. Error Handling for Image Operations

### Comprehensive Error Strategy

```dart
enum ImageProcessingError {
  unsupportedFormat,
  fileTooLarge,
  compressionFailed,
  uploadFailed,
  aiAnalysisFailed,
}

class ImageErrorHandler {
  static String getUserMessage(ImageProcessingError error) {
    switch (error) {
      case ImageProcessingError.unsupportedFormat:
        return 'Vennligst velg et bilde i JPEG, PNG eller HEIC format.';
      case ImageProcessingError.fileTooLarge:
        return 'Bildet er for stort. Maksimal størrelse er 10 MB.';
      case ImageProcessingError.compressionFailed:
        return 'Kunne ikke behandle bildet. Prøv et annet bilde.';
      case ImageProcessingError.uploadFailed:
        return 'Opplasting feilet. Sjekk internettforbindelsen.';
      case ImageProcessingError.aiAnalysisFailed:
        return 'AI-analyse feilet. Du kan fortsatt legge til bildet manuelt.';
    }
  }
}
```

## Common Mistakes / Anti-Patterns

| ❌ Anti-Pattern | ✅ Required Norsktorget Pattern |
|:---|:---|
| Sending raw, uncompressed images to Gemini API | ALWAYS pre-process and optimize images using ImagePreprocessor. |
| Not implementing caching for AI results | ALWAYS use two-level caching (memory + Hive) to avoid redundant API calls. |
| Blocking UI during image upload | ALWAYS use AsyncNotifier and show loading states during processing. |
| Ignoring image hash for cache keys | ALWAYS generate and use image hash for reliable caching. |
| Using PNG for all images | ALWAYS convert to JPEG with 85% quality for optimal size/quality ratio. |
