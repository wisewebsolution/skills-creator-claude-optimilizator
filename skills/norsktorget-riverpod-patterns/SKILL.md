---
name: norsktorget-riverpod-patterns
description: Use when implementing state management in the Norsktorget project. This skill provides the required patterns for Riverpod 3.0 with code generation (@riverpod), including Notifier, AsyncNotifier, family providers, and testing with ProviderContainer. It enforces project-specific standards.
---

# Riverpod 3.0 Patterns for Norsktorget

## Overview

This skill defines the **required** state management patterns for the Norsktorget project, using Riverpod 3.0 with code generation (`@riverpod`). It ensures type-safe, scalable, and testable state management.

**Core Principle:** All state management logic MUST be implemented using the patterns described here. Legacy patterns like `StateNotifierProvider` are forbidden.

## When to Use

- When creating any new state management logic.
- When refactoring existing UI to handle state.
- When fetching, caching, or mutating data from an API or local database.
- When implementing dependency injection for services or repositories.
- When writing tests for widgets or business logic that involve state.

## 1. The `@riverpod` Annotation (The Norsktorget Way)

All providers MUST be created using the `@riverpod` annotation and code generation. Manual provider creation is not allowed.

### Simple Immutable Data (Services, Repositories)

Use a plain function for dependency injection.

```dart
// lib/features/auth/data/providers.dart

import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'repositories/auth_repository.dart';

part 'providers.g.dart';

// REQUIRED PATTERN for providing services
@riverpod
AuthRepository authRepository(AuthRepositoryRef ref) {
  // This allows swapping implementations for tests
  return FakeAuthRepository();
}
```

## 2. Modern State Management: Notifier and AsyncNotifier

`StateNotifier` is deprecated in this project. Use `Notifier` for synchronous state and `AsyncNotifier` for asynchronous state.

### Notifier for Synchronous State

Use for simple, synchronous state changes (e.g., UI state, form management).

```dart
// lib/features/theme/presentation/providers/theme_provider.dart

import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'theme_provider.g.dart';

// REQUIRED PATTERN for synchronous state
@riverpod
class ThemeMode extends _$ThemeMode {
  @override
  // build() sets the initial state
  bool build() {
    return false; // Initial state: light mode
  }

  void setDarkTheme() {
    state = true;
  }

  void setLightTheme() {
    state = false;
  }

  void toggle() {
    state = !state;
  }
}
```

### AsyncNotifier for Asynchronous State (CRITICAL PATTERN)

Use for any state that involves asynchronous operations like API calls or database queries. It automatically handles loading, data, and error states.

```dart
// lib/features/ads/presentation/providers/ad_list_provider.dart

import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'package:norsktorget/features/ads/domain/entities/ad.dart';
import 'package:norsktorget/features/ads/data/providers.dart';

part 'ad_list_provider.g.dart';

// REQUIRED PATTERN for asynchronous data and mutations
@riverpod
class AdList extends _$AdList {
  // build() fetches the initial data
  @override
  Future<List<Ad>> build() async {
    // Read repository using ref
    final adRepository = ref.watch(adRepositoryProvider);
    return adRepository.fetchAds();
  }

  // Method to add a new item
  Future<void> addAd(String title) async {
    // 1. Set state to loading
    state = const AsyncValue.loading();

    // 2. Use AsyncValue.guard to automatically handle errors
    state = await AsyncValue.guard(() async {
      final adRepository = ref.read(adRepositoryProvider);
      await adRepository.addAd(title: title);
      // 3. Refetch the list to get the new state
      return adRepository.fetchAds();
    });
  }

  // Method to delete an item
  Future<void> deleteAd(String id) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      final adRepository = ref.read(adRepositoryProvider);
      await adRepository.deleteAd(id: id);
      return adRepository.fetchAds();
    });
  }
}
```

## 3. Parameterized State with family

The `family` modifier is handled automatically by the generator when you add parameters to the `build` method.

```dart
// lib/features/ads/presentation/providers/ad_provider.dart

import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'ad_provider.g.dart';

// REQUIRED PATTERN for providers that need an argument
@riverpod
class Ad extends _$Ad {
  // The argument is passed to the build method
  @override
  Future<AdData> build(String adId) async {
    final adRepository = ref.watch(adRepositoryProvider);
    return adRepository.fetchAd(adId);
  }

  Future<void> updateTitle(String newTitle) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      final adRepository = ref.read(adRepositoryProvider);
      // The adId is available via `arg`
      return adRepository.updateAd(id: arg, newTitle: newTitle);
    });
  }
}

// In the UI:
final adAsyncValue = ref.watch(adProvider('some-ad-id'));
```

## 4. Consuming State in Widgets

Always use `ConsumerWidget` or `ConsumerStatefulWidget`.

```dart
// lib/features/ads/presentation/widgets/ad_list_view.dart

class AdListView extends ConsumerWidget {
  const AdListView({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Use ref.watch to listen to a provider
    final adListAsyncValue = ref.watch(adListProvider);

    // Use .when() to handle loading/error/data states
    return adListAsyncValue.when(
      data: (ads) => ListView.builder(
        itemCount: ads.length,
        itemBuilder: (context, index) => Text(ads[index].title),
      ),
      loading: () => const Center(child: CircularProgressIndicator()),
      error: (err, stack) => Center(child: Text('Error: $err')),
    );
  }
}
```

### Calling Methods on Notifiers

Use `ref.read` with the `.notifier` property to call methods.

```dart
// Inside a button's onPressed callback
onPressed: () {
  // Use ref.read inside callbacks
  ref.read(adListProvider.notifier).addAd('My New Awesome Ad');
}
```

## 5. Testing Providers: The Complete Guide

All provider tests MUST use `ProviderContainer`.

```dart
// test/features/ads/ad_list_provider_test.dart

import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:mocktail/mocktail.dart';

// 1. Mock your repository
class MockAdRepository extends Mock implements AdRepository {}

void main() {
  group('AdList Notifier Test', () {
    late ProviderContainer container;
    late MockAdRepository mockAdRepository;

    setUp(() {
      mockAdRepository = MockAdRepository();
      // 2. Create a ProviderContainer with overrides
      container = ProviderContainer(
        overrides: [
          // Override the repository provider with the mock
          adRepositoryProvider.overrideWithValue(mockAdRepository),
        ],
      );
    });

    tearDown(() {
      container.dispose();
    });

    test('Initial state is loading, then fetches data successfully', () async {
      // Arrange
      final ads = [Ad(id: '1', title: 'Test Ad')];
      when(() => mockAdRepository.fetchAds()).thenAnswer((_) async => ads);

      // 3. Create a listener to observe state changes
      final listener = Listener<AsyncValue<List<Ad>>>();
      container.listen(
        adListProvider,
        listener.call,
        fireImmediately: true,
      );

      // Assert initial state is loading
      verify(() => listener(
        null,
        const AsyncLoading<List<Ad>>(),
      )).called(1);

      // Wait for the build method to complete
      await container.read(adListProvider.future);

      // Assert final state is data
      verify(() => listener(
        const AsyncLoading<List<Ad>>(),
        AsyncData<List<Ad>>(ads),
      )).called(1);

      verifyNoMoreInteractions(listener);
      verify(() => mockAdRepository.fetchAds()).called(1);
    });

    test('addAd method updates state correctly', () async {
      // Arrange
      final initialAds = [Ad(id: '1', title: 'First Ad')];
      final newAd = Ad(id: '2', title: 'Second Ad');
      when(() => mockAdRepository.fetchAds()).thenAnswer((_) async => initialAds);
      when(() => mockAdRepository.addAd(title: 'Second Ad')).thenAnswer((_) async {});
      // Make the second fetchAds call return the updated list
      when(() => mockAdRepository.fetchAds()).thenAnswer((_) async => [...initialAds, newAd]);


      // Wait for initial build
      await container.read(adListProvider.future);

      // Act
      await container.read(adListProvider.notifier).addAd('Second Ad');

      // Assert
      final finalState = container.read(adListProvider);
      expect(finalState, isA<AsyncData>());
      expect(finalState.asData!.value.length, 2);
      expect(finalState.asData!.value.last.title, 'Second Ad');
    });
  });
}
```

## Common Mistakes / Anti-Patterns

| ❌ Anti-Pattern | ✅ Required Norsktorget Pattern |
|:---|:---|
| Using StateNotifier and StateNotifierProvider | ALWAYS use Notifier/AsyncNotifier with @riverpod. |
| Manual provider definitions (final myProvider = ...) | ALWAYS use @riverpod for code generation. |
| Mixing FutureProvider with manual state mutations in the UI. | ALWAYS use AsyncNotifier to encapsulate all async logic. |
| Testing notifiers by creating a direct instance. | ALWAYS use ProviderContainer to test providers in a realistic environment. |
