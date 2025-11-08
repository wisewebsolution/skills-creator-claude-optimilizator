# Norsktorget Quick Reference Card

## Essential Rules

1. **Max 300 lines per file** - Split if exceeded
2. **Relative imports only** - Never `package:` imports
3. **Hive CE** - `import 'package:hive_ce/hive_ce.dart';`
4. **Freezed** - All data classes use `@freezed`
5. **Riverpod 3.0** - Use `@riverpod` annotation style
6. **Result pattern** - `Success<T>` and `Failure<T>` for error handling

## File Naming

| Type | Convention | Example |
|------|-----------|---------|
| Dart files | snake_case | `ad_creation_screen.dart` |
| TypeScript files | kebab-case | `ad-creation.service.ts` |
| Classes | PascalCase | `AdCreationScreen` |
| Variables | camelCase | `userName` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_FILE_SIZE` |
| Widget constants | k prefix | `kPrimaryColor` |

## Import Order

```dart
// 1. Dart SDK
import 'dart:async';

// 2. Flutter
import 'package:flutter/material.dart';

// 3. Third-party
import 'package:riverpod_annotation/riverpod_annotation.dart';

// 4. Project (relative!)
import '../../core/constants.dart';

// 5. Part files
part 'file.freezed.dart';
```

## Architecture Layers

```
domain/         → Pure business logic (entities, repositories, use cases)
  ├── entities/
  ├── repositories/
  └── usecases/

data/          → Implementation (models, datasources, repo implementations)
  ├── datasources/
  ├── models/
  └── repositories/

presentation/  → UI and state (providers, screens, widgets)
  ├── providers/
  ├── screens/
  └── widgets/
```

## Quick Patterns

### Freezed Data Class
```dart
@freezed
class AdDraft with _$AdDraft {
  const factory AdDraft({
    required String id,
    required String title,
  }) = _AdDraft;
  
  factory AdDraft.fromJson(Map<String, dynamic> json) =>
      _$AdDraftFromJson(json);
}
```

### Riverpod Provider
```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
  
  void increment() => state++;
}
```

### Result Pattern
```dart
Future<Result<T>> operation() async {
  try {
    final data = await repository.fetch();
    return Success(data);
  } catch (e) {
    return Failure(AppException(e.toString()));
  }
}
```

### Repository Interface
```dart
abstract class ItemRepository {
  Future<Result<List<Item>>> getAll();
  Future<Result<Item>> getById(String id);
  Future<Result<void>> save(Item item);
}
```

### Screen Widget
```dart
class MyScreen extends ConsumerWidget {
  const MyScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(myProvider);
    
    return Scaffold(
      appBar: AppBar(title: const Text('Title')),
      body: state.when(
        loading: () => const CircularProgressIndicator(),
        data: (data) => _buildContent(context, data),
        error: (error, _) => ErrorView(error.toString()),
      ),
    );
  }
}
```

## Common Mistakes

❌ **DON'T**:
```dart
// Absolute imports
import 'package:norsktorget_mobile/features/ads/...';

// Old Hive
import 'package:hive/hive.dart';

// Old Riverpod
final provider = StateNotifierProvider(...);

// Direct exceptions
throw Exception('error');

// Files > 300 lines
class HugeWidget { // 500+ lines
```

✅ **DO**:
```dart
// Relative imports
import '../../features/ads/...';

// Hive CE
import 'package:hive_ce/hive_ce.dart';

// Riverpod 3.0
@riverpod
class MyNotifier extends _$MyNotifier { }

// Result pattern
return Failure(AppException('error'));

// Split large files
class Widget1 { } // 200 lines
class Widget2 { } // 200 lines (separate file)
```

## Testing Checklist

- [ ] >80% code coverage
- [ ] Unit tests for services/repos/use cases
- [ ] Widget tests for screens
- [ ] Integration tests for critical flows
- [ ] Mock external dependencies
- [ ] Test error cases

## Before Committing

- [ ] `flutter analyze` passes (no errors)
- [ ] `flutter test` passes (all tests green)
- [ ] No files exceed 300 lines
- [ ] All imports are relative
- [ ] Using Hive CE, not old Hive
- [ ] Semantic commit message: `type(scope): message`

## Need Help?

- **Architecture**: Read `references/project_architecture.md`
- **Flutter patterns**: Read `references/flutter_standards.md`
- **TypeScript patterns**: Read `references/typescript_standards.md`
- **Existing code**: Look for similar features in the codebase
