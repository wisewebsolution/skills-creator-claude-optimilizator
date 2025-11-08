---
name: norsktorget-coding-standards
description: Write code that follows Norsktorget project standards. Use when writing Flutter/Dart or TypeScript/NestJS code for the Norsktorget marketplace project. Ensures code adheres to project architecture, naming conventions, file organization, and best practices specific to this Norwegian AI-powered marketplace platform.
license: MIT
---

# Norsktorget Coding Standards

This skill helps write code that follows Norsktorget.no project standards, ensuring consistency and maintainability across the Norwegian AI-powered marketplace platform.

## When to Use This Skill

Use this skill when:
- Writing new Flutter/Dart code for the mobile app
- Writing new TypeScript/NestJS code for the backend (Stage 2+)
- Refactoring existing code to meet project standards
- Creating new features following the project architecture
- Reviewing code for compliance with project conventions
- Setting up new modules or components

## Project Context

**Project**: Norsktorget.no - Norwegian AI-powered marketplace
**Current Stage**: Stage 1 (Mobile app with offline-first AI features)
**Flutter Version**: 3.35.2 (Dart 3.8.0+)
**State Management**: Riverpod 3.0
**Architecture**: Clean Architecture with feature-first organization

## Core Principles (Always Follow)

### 1. File Size Limit
**Maximum 300 lines per file**. If a file exceeds this limit, split it into multiple smaller files with clear responsibilities.

### 2. Import Style
**Use relative imports** for project files, never absolute `package:` imports:
```dart
// ✅ CORRECT
import '../../core/constants/app_constants.dart';
import '../models/ad_model.dart';

// ❌ WRONG
import 'package:norsktorget_mobile/core/constants/app_constants.dart';
import 'package:norsktorget_mobile/features/ads/models/ad_model.dart';
```

### 3. Hive CE Usage
**Use Hive CE (Community Edition)**, not the old Hive package:
```dart
// ✅ CORRECT
import 'package:hive_ce/hive_ce.dart';

// ❌ WRONG
import 'package:hive/hive.dart';
```

### 4. Freezed for Data Classes
**All data classes use Freezed** for immutability and code generation:
```dart
@freezed
class AdDraft with _$AdDraft {
  const factory AdDraft({
    required String id,
    required String title,
    required double price,
  }) = _AdDraft;
  
  factory AdDraft.fromJson(Map<String, dynamic> json) =>
      _$AdDraftFromJson(json);
}
```

### 5. Riverpod 3.0 Style
**Use `@riverpod` annotation style**, not the old provider style:
```dart
// ✅ CORRECT
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
  
  void increment() => state++;
}

// ❌ WRONG
final counterProvider = StateNotifierProvider<Counter, int>((ref) {
  return Counter();
});
```

### 6. Result Pattern for Error Handling
**Use Result<T> pattern** instead of throwing exceptions directly:
```dart
Future<Result<Ad>> createAd(AdDraft draft) async {
  try {
    final ad = await repository.create(draft);
    return Success(ad);
  } catch (e) {
    return Failure(AppException(e.toString()));
  }
}
```

## Quick Reference Decision Tree

### "I need to write Flutter code"
1. Read `references/flutter_standards.md` for naming, imports, and patterns
2. Read `references/project_architecture.md` for where the file should go
3. Ensure max 300 lines per file
4. Use relative imports
5. Follow the Clean Architecture layers (domain → data → presentation)

### "I need to write TypeScript code"
1. Read `references/typescript_standards.md` for NestJS patterns
2. Read `references/project_architecture.md` for module structure
3. Ensure max 300 lines per file
4. Use DTOs for validation
5. Follow dependency injection patterns

### "I need to understand project architecture"
1. Read `references/project_architecture.md` for full structure
2. Understand the 3-layer Clean Architecture (Domain, Data, Presentation)
3. Follow feature-first organization
4. Use the Result pattern for error handling

### "I need to create a new feature"
1. Read `references/project_architecture.md` for feature structure
2. Create folders: `data/`, `domain/`, `presentation/`
3. Start with domain entities and repository interfaces
4. Implement data layer (models, datasources, repository implementations)
5. Create presentation layer (providers, screens, widgets)
6. Follow naming conventions from `references/flutter_standards.md`

## Code Review Checklist

Before completing any code task, verify:

### General
- [ ] No file exceeds 300 lines
- [ ] All imports are properly organized
- [ ] No unused imports or variables
- [ ] Descriptive names (no `temp`, `data`, `var1`)

### Flutter-Specific
- [ ] Using relative imports (not `package:` imports)
- [ ] Using `package:hive_ce/hive_ce.dart` (not old Hive)
- [ ] Freezed classes for all data models
- [ ] Riverpod 3.0 `@riverpod` annotation style
- [ ] Result pattern for error handling
- [ ] Const constructors where possible

### TypeScript-Specific
- [ ] DTOs with validation decorators
- [ ] TypeORM entities for database
- [ ] Dependency injection (no `new`)
- [ ] Custom exceptions (not generic Error)
- [ ] Proper async/await (no `.then()`)

### Architecture
- [ ] Correct layer placement (domain/data/presentation)
- [ ] Feature-first organization
- [ ] Single responsibility per file
- [ ] Proper dependency flow (presentation → domain → data)

### Testing
- [ ] Unit tests for business logic
- [ ] Widget tests for UI components
- [ ] Target >80% coverage

## Common Patterns

### Creating a New Feature (Flutter)

```bash
# Structure
features/new_feature/
├── data/
│   ├── datasources/
│   │   └── new_feature_datasource.dart
│   ├── models/
│   │   └── new_feature_model.dart
│   └── repositories/
│       └── new_feature_repository_impl.dart
├── domain/
│   ├── entities/
│   │   └── new_feature.dart
│   ├── repositories/
│   │   └── new_feature_repository.dart
│   └── usecases/
│       └── do_something.dart
└── presentation/
    ├── providers/
    │   └── new_feature_provider.dart
    ├── screens/
    │   └── new_feature_screen.dart
    └── widgets/
        └── new_feature_widget.dart
```

### Creating a New API Endpoint (NestJS)

```bash
# Structure
modules/new_module/
├── dto/
│   ├── create-item.dto.ts
│   └── update-item.dto.ts
├── entities/
│   └── item.entity.ts
├── mappers/
│   └── item.mapper.ts
├── new-module.controller.ts
├── new-module.service.ts
└── new-module.module.ts
```

## Norwegian Language Considerations

### UI Text
- All user-facing text in Norwegian (Bokmål)
- Use Norwegian characters: æ, ø, å
- Error messages in Norwegian
- API responses in English (internal)

### Code
- Code (variables, functions, classes) in English
- Comments in English
- Documentation in Polish (project docs) or English (code docs)

## Performance Guidelines

### Flutter
1. Use `const` constructors wherever possible
2. Avoid rebuilds with Riverpod's `select`
3. Compress images before processing (max 2MB)
4. Debounce text inputs (2 second delay)
5. Lazy load heavy widgets

### TypeScript
1. Use database indexes for frequent queries
2. Cache expensive operations (Redis)
3. Paginate large result sets
4. Use transactions for multi-step operations
5. Monitor query performance

## Security Checklist

### Flutter
- [ ] No API keys in code (use environment variables)
- [ ] Validate all user inputs
- [ ] Sanitize data before display
- [ ] Use HTTPS for all API calls
- [ ] Secure local storage (Hive encryption if needed)

### TypeScript
- [ ] Input validation with DTOs
- [ ] Authentication with JWT
- [ ] Authorization with Guards
- [ ] Rate limiting on endpoints
- [ ] SQL injection prevention (TypeORM handles this)
- [ ] CORS properly configured

## References

For detailed information on specific topics:

1. **Flutter/Dart Standards**: Read `references/flutter_standards.md`
   - Naming conventions
   - File organization
   - Widget patterns
   - State management
   - Testing requirements

2. **TypeScript/NestJS Standards**: Read `references/typescript_standards.md`
   - Module structure
   - Controller/Service patterns
   - DTO validation
   - Entity definitions
   - Error handling

3. **Project Architecture**: Read `references/project_architecture.md`
   - Clean Architecture layers
   - Feature-first organization
   - Data flow
   - Common integrations
   - Testing strategy

## Examples

### Example 1: Creating a New Screen

```dart
// 1. Start with the screen widget
class NewFeatureScreen extends ConsumerWidget {
  const NewFeatureScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(newFeatureProvider);
    
    return Scaffold(
      appBar: AppBar(title: const Text('Ny Funksjon')),
      body: state.when(
        loading: () => const Center(child: CircularProgressIndicator()),
        data: (data) => _buildContent(context, data),
        error: (error, _) => ErrorWidget(error.toString()),
      ),
    );
  }
  
  Widget _buildContent(BuildContext context, FeatureData data) {
    // Build UI here (keep under 300 lines!)
    return ListView(/* ... */);
  }
}
```

### Example 2: Creating a Repository

```dart
// domain/repositories/feature_repository.dart
abstract class FeatureRepository {
  Future<Result<List<Item>>> getAll();
  Future<Result<Item>> getById(String id);
  Future<Result<void>> save(Item item);
}

// data/repositories/feature_repository_impl.dart
class FeatureRepositoryImpl implements FeatureRepository {
  final FeatureDatasource datasource;
  
  const FeatureRepositoryImpl(this.datasource);
  
  @override
  Future<Result<List<Item>>> getAll() async {
    try {
      final models = await datasource.getAll();
      return Success(models.map((m) => m.toEntity()).toList());
    } catch (e) {
      return Failure(CacheException(e.toString()));
    }
  }
}
```

### Example 3: Creating a Provider

```dart
@riverpod
class NewFeature extends _$NewFeature {
  @override
  Future<FeatureData> build() async {
    final repository = ref.watch(featureRepositoryProvider);
    final result = await repository.getAll();
    
    return result.when(
      success: (data) => FeatureData(items: data),
      failure: (error) => throw error,
    );
  }
  
  Future<void> addItem(Item item) async {
    state = const AsyncValue.loading();
    
    final repository = ref.read(featureRepositoryProvider);
    final result = await repository.save(item);
    
    state = await AsyncValue.guard(() => build());
  }
}
```

## Troubleshooting

### Common Issues

**Issue**: "Cannot use Ref after disposed"
**Solution**: Use `ref.read()` in callbacks, not `ref.watch()`

**Issue**: "Type 'LinkedMap<dynamic, dynamic>' cannot be cast to 'Map<String, dynamic>'"
**Solution**: Use `Map<String, dynamic>.from(data)` when deserializing from Hive

**Issue**: "Image.file is not supported on Flutter Web"
**Solution**: Create platform-specific image widgets using `kIsWeb` checks

**Issue**: File too large (>300 lines)
**Solution**: Extract widgets/functions to separate files with clear names

## Project-Specific Notes

### Stage 1 (Current)
- **Offline-first**: All data stored locally in Hive
- **Mock data**: Vegvesen API uses proxy with fallback mocks
- **No backend**: No authentication or publishing yet
- **AI powered**: Gemini 2.5 Flash for image analysis
- **Cross-platform**: Targets iOS, Android, and Web

### Stage 2 (Future)
- **Backend**: NestJS + PostgreSQL + Redis
- **Authentication**: JWT-based user accounts
- **Publishing**: Sync local drafts to server
- **Admin panel**: Next.js for moderation

### Stage 3 (Future)
- **Full marketplace**: Search, filters, chat
- **Payments**: Stripe + Vipps integration
- **Social features**: Ratings, favorites, following

## Final Checklist

Before marking code as complete:

1. [ ] Read relevant reference documentation
2. [ ] Followed project architecture
3. [ ] Used correct naming conventions
4. [ ] No files exceed 300 lines
5. [ ] Proper imports (relative, organized)
6. [ ] Used Freezed for data classes
7. [ ] Used Riverpod 3.0 patterns
8. [ ] Implemented Result pattern
9. [ ] Added necessary tests
10. [ ] No lint warnings or errors
11. [ ] Verified with `flutter analyze` or `npm run lint`
12. [ ] Committed with semantic commit message

## Getting Help

When uncertain about:
- **Architecture decisions**: Check `references/project_architecture.md`
- **Naming conventions**: Check language-specific standards references
- **Best practices**: Check `references/principles.md` (if available)
- **API patterns**: Check existing code in similar features

Remember: Consistency is key. When in doubt, follow existing patterns in the codebase.
