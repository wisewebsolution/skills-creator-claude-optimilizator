# Flutter/Dart Coding Standards - Norsktorget.no

## File Organization

### Import Order
```dart
// 1. Dart SDK imports (alphabetically)
import 'dart:async';
import 'dart:convert';
import 'dart:io';

// 2. Flutter imports
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

// 3. Third-party packages (alphabetically)
import 'package:dio/dio.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:hive_ce/hive_ce.dart';

// 4. Project imports (relative paths)
import '../../core/constants/app_constants.dart';
import '../../core/errors/exceptions.dart';
import '../models/ad_model.dart';

// 5. Part files
part 'ad_state.freezed.dart';
part 'ad_state.g.dart';
```

## Naming Conventions

### Files
- **snake_case** for all files
- Be descriptive: `image_compression_service.dart` not `service.dart`

### Classes and Enums
- **PascalCase** for classes
- **PascalCase** for enums with **camelCase** values

### Variables and Functions
- **camelCase** for variables and functions
- Descriptive boolean names: `isLoading`, `hasPermission`, `canEdit`
- Private members with underscore: `_privateField`, `_privateMethod()`

### Constants
- **SCREAMING_SNAKE_CASE** for constants
- Use `k` prefix for widget constants: `kPrimaryColor`, `kDefaultPadding`

## Architecture

### Feature-First Structure
```
lib/
├── core/
│   ├── constants/
│   ├── errors/
│   ├── theme/
│   └── utils/
├── features/
│   ├── ad_creation/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   ├── models/
│   │   │   └── repositories/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   ├── repositories/
│   │   │   └── usecases/
│   │   └── presentation/
│   │       ├── providers/
│   │       ├── screens/
│   │       └── widgets/
```

## Key Rules

1. **Max 300 lines per file** - Split larger files
2. **Use relative imports** - Never absolute `package:` imports
3. **Hive CE imports** - Use `package:hive_ce/hive_ce.dart` not old Hive
4. **Freezed for models** - All data classes use Freezed
5. **Riverpod 3.0** - Use `@riverpod` annotation style
6. **Null safety** - Explicit null handling everywhere

## Widget Structure

```dart
class MyWidget extends ConsumerWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 1. Watch providers
    final state = ref.watch(myProvider);
    
    // 2. Early returns for loading/error
    if (state.isLoading) {
      return const Center(child: CircularProgressIndicator());
    }
    
    // 3. Build UI
    return Scaffold(
      appBar: AppBar(title: const Text('Title')),
      body: _buildBody(context, state),
    );
  }
  
  // 4. Extract complex widgets
  Widget _buildBody(BuildContext context, MyState state) {
    return ListView.builder(
      itemCount: state.items.length,
      itemBuilder: (context, index) => _buildItem(state.items[index]),
    );
  }
  
  Widget _buildItem(Item item) {
    return ListTile(title: Text(item.name));
  }
}
```

## State Management (Riverpod 3.0)

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'my_provider.g.dart';

// Simple provider
@riverpod
String greeting(GreetingRef ref) {
  return 'Hello World';
}

// Async provider
@riverpod
Future<User> user(UserRef ref, String id) async {
  final repository = ref.watch(userRepositoryProvider);
  return repository.getUser(id);
}

// Notifier for mutable state
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
  
  void increment() => state++;
  void decrement() => state--;
}
```

## Error Handling

```dart
// Use Result pattern
sealed class Result<T> {
  const Result();
}

class Success<T> extends Result<T> {
  final T data;
  const Success(this.data);
}

class Failure<T> extends Result<T> {
  final AppException exception;
  const Failure(this.exception);
}

// Usage
Future<Result<Ad>> createAd(AdDraft draft) async {
  try {
    final ad = await repository.create(draft);
    return Success(ad);
  } on NetworkException catch (e) {
    return Failure(NetworkException(e.message));
  } catch (e) {
    return Failure(UnknownException(e.toString()));
  }
}
```

## Testing Requirements

- **>80% coverage** for all features
- Unit tests for services, repositories, providers
- Widget tests for screens and complex widgets
- Integration tests for critical flows

## Performance

- Use `const` constructors wherever possible
- Avoid rebuilds with `select` in Riverpod
- Lazy load heavy widgets
- Image compression before display
- Debounce text inputs

## Common Patterns

### Form Validation
```dart
String? validatePhone(String? value) {
  if (value == null || value.isEmpty) {
    return 'Telefonnummer er påkrevd';
  }
  final regex = RegExp(r'^(\+47)?[2-9]\d{7}$');
  if (!regex.hasMatch(value.replaceAll(' ', ''))) {
    return 'Ugyldig norsk telefonnummer';
  }
  return null;
}
```

### Image Compression
```dart
Future<Uint8List> compressImage(Uint8List bytes) async {
  final originalSize = bytes.length;
  if (originalSize <= MAX_SIZE) return bytes;
  
  int quality = 95;
  while (quality > 20) {
    final compressed = await FlutterImageCompress.compressWithList(
      bytes,
      quality: quality,
      format: CompressFormat.jpeg,
    );
    
    if (compressed.length <= MAX_SIZE) return compressed;
    quality -= 10;
  }
  
  return bytes;
}
```

### Navigation with Go Router
```dart
// Definition
@TypedGoRoute<AdCreationRoute>(path: '/ad/create')
class AdCreationRoute extends GoRouteData {
  const AdCreationRoute();
  
  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const AdCreationScreen();
  }
}

// Usage
context.push(const AdCreationRoute());
```
