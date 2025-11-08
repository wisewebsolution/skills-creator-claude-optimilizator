# Norsktorget Project Architecture

## Overview

Norsktorget is a Norwegian AI-powered marketplace built in 3 stages:
- **Stage 1**: Mobile app with offline-first AI features (CURRENT)
- **Stage 2**: Backend + Admin Panel + Publishing
- **Stage 3**: Full marketplace with chat and payments

## Current Stage: Stage 1 (Mobile Only)

### Technology Stack
- **Flutter**: 3.35.2
- **Dart**: 3.8.0+
- **State Management**: Riverpod 3.0
- **Local Storage**: Hive CE
- **Navigation**: Go Router 16.0
- **AI**: Google Gemini 2.5 Flash
- **HTTP**: Dio

### Mobile App Structure

```
mobile/lib/
├── core/
│   ├── constants/
│   │   └── app_constants.dart
│   ├── errors/
│   │   ├── exceptions.dart
│   │   └── failures.dart
│   ├── l10n/
│   │   ├── app_no.arb
│   │   └── app_localizations.dart
│   ├── router/
│   │   └── app_router.dart
│   ├── theme/
│   │   ├── app_theme.dart
│   │   └── color_schemes.dart
│   └── utils/
│       ├── validators.dart
│       └── formatters.dart
│
├── features/
│   ├── ad_creation/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   │   ├── local_ad_datasource.dart
│   │   │   │   └── image_datasource.dart
│   │   │   ├── models/
│   │   │   │   ├── ad_draft_model.dart
│   │   │   │   └── ai_analysis_model.dart
│   │   │   └── repositories/
│   │   │       └── local_ad_repository_impl.dart
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   ├── ad_draft.dart
│   │   │   │   └── ai_analysis.dart
│   │   │   ├── repositories/
│   │   │   │   └── local_ad_repository.dart
│   │   │   └── usecases/
│   │   │       ├── create_ad_from_input.dart
│   │   │       └── save_draft.dart
│   │   └── presentation/
│   │       ├── providers/
│   │       │   ├── ad_creation_provider.dart
│   │       │   └── draft_list_provider.dart
│   │       ├── screens/
│   │       │   ├── ad_form_screen.dart
│   │       │   ├── ai_analysis_screen.dart
│   │       │   └── camera_screen.dart
│   │       └── widgets/
│   │           ├── ai_suggestion_card.dart
│   │           ├── category_selector.dart
│   │           └── image_picker_widget.dart
│   │
│   ├── ai/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   │   └── gemini_client.dart
│   │   │   └── repositories/
│   │   │       └── ai_repository_impl.dart
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   └── ai_analysis.dart
│   │   │   ├── repositories/
│   │   │   │   └── ai_repository.dart
│   │   │   └── usecases/
│   │   │       ├── analyze_image.dart
│   │   │       └── recognize_vehicle.dart
│   │   └── presentation/
│   │       └── providers/
│   │           └── ai_provider.dart
│   │
│   ├── categories/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   │   └── category_datasource.dart
│   │   │   └── repositories/
│   │   │       └── category_repository_impl.dart
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   ├── category.dart
│   │   │   │   └── category_field.dart
│   │   │   └── repositories/
│   │   │       └── category_repository.dart
│   │   └── presentation/
│   │       ├── providers/
│   │       │   └── category_provider.dart
│   │       └── widgets/
│   │           └── category_tree_widget.dart
│   │
│   └── vehicle/
│       ├── data/
│       │   ├── datasources/
│       │   │   └── vegvesen_datasource.dart
│       │   └── repositories/
│       │       └── vehicle_repository_impl.dart
│       ├── domain/
│       │   ├── entities/
│       │   │   └── vehicle_data.dart
│       │   └── repositories/
│       │       └── vehicle_repository.dart
│       └── presentation/
│           ├── providers/
│           │   └── vehicle_provider.dart
│           └── screens/
│               └── vehicle_details_screen.dart
│
└── main.dart
```

## Clean Architecture Layers

### 1. Domain Layer
**Purpose**: Business logic and entities (pure Dart, no Flutter/external dependencies)

**Contains**:
- **Entities**: Core business objects (Freezed classes)
- **Repositories**: Abstract interfaces
- **Use Cases**: Single-purpose business operations

**Example**:
```dart
// domain/entities/ad_draft.dart
@freezed
class AdDraft with _$AdDraft {
  const factory AdDraft({
    required String id,
    required String title,
    required double price,
    required String categoryId,
    Map<String, dynamic>? customFields,
  }) = _AdDraft;
}

// domain/repositories/local_ad_repository.dart
abstract class LocalAdRepository {
  Future<Result<List<AdDraft>>> getAll();
  Future<Result<AdDraft>> getById(String id);
  Future<Result<void>> save(AdDraft draft);
  Future<Result<void>> delete(String id);
}

// domain/usecases/save_draft.dart
@riverpod
class SaveDraft {
  Future<Result<void>> call(AdDraft draft) async {
    return repository.save(draft);
  }
}
```

### 2. Data Layer
**Purpose**: Implementation of repositories and data sources

**Contains**:
- **Models**: Serializable versions of entities (with `fromJson`, `toJson`)
- **Datasources**: Direct communication with APIs/databases
- **Repository Implementations**: Concrete implementations of domain interfaces

**Example**:
```dart
// data/models/ad_draft_model.dart
@freezed
class AdDraftModel with _$AdDraftModel {
  @HiveType(typeId: 0)
  const factory AdDraftModel({
    @HiveField(0) required String id,
    @HiveField(1) required String title,
    @HiveField(2) required double price,
    // ...
  }) = _AdDraftModel;
  
  factory AdDraftModel.fromJson(Map<String, dynamic> json) =>
      _$AdDraftModelFromJson(json);
}

// data/datasources/local_ad_datasource.dart
class LocalAdDatasource {
  final Box<AdDraftModel> box;
  
  Future<List<AdDraftModel>> getAll() async {
    return box.values.toList();
  }
  
  Future<void> save(AdDraftModel draft) async {
    await box.put(draft.id, draft);
  }
}

// data/repositories/local_ad_repository_impl.dart
class LocalAdRepositoryImpl implements LocalAdRepository {
  final LocalAdDatasource datasource;
  
  @override
  Future<Result<List<AdDraft>>> getAll() async {
    try {
      final models = await datasource.getAll();
      return Success(models.map((m) => m.toEntity()).toList());
    } catch (e) {
      return Failure(CacheException(e.toString()));
    }
  }
}
```

### 3. Presentation Layer
**Purpose**: UI and state management

**Contains**:
- **Providers**: Riverpod state management
- **Screens**: Full-page widgets
- **Widgets**: Reusable UI components

**Example**:
```dart
// presentation/providers/ad_creation_provider.dart
@riverpod
class AdCreation extends _$AdCreation {
  @override
  AdCreationState build() => const AdCreationState.initial();
  
  Future<void> analyzeImage(XFile image) async {
    state = const AdCreationState.loading();
    
    final result = await ref.read(analyzeImageUsecaseProvider).call(image);
    
    state = result.when(
      success: (analysis) => AdCreationState.analyzed(analysis),
      failure: (error) => AdCreationState.error(error.message),
    );
  }
}

// presentation/screens/ad_form_screen.dart
class AdFormScreen extends ConsumerWidget {
  const AdFormScreen({super.key});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(adCreationProvider);
    
    return Scaffold(
      appBar: AppBar(title: const Text('Nytt Annonse')),
      body: state.when(
        initial: () => const InitialView(),
        loading: () => const LoadingView(),
        analyzed: (analysis) => AdForm(analysis: analysis),
        error: (message) => ErrorView(message: message),
      ),
    );
  }
}
```

## Key Patterns

### Result Pattern
```dart
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
```

### Dependency Injection (Riverpod)
```dart
// Provider for repository
@riverpod
LocalAdRepository localAdRepository(LocalAdRepositoryRef ref) {
  return LocalAdRepositoryImpl(
    datasource: ref.watch(localAdDatasourceProvider),
  );
}

// Provider for use case
@riverpod
SaveDraft saveDraft(SaveDraftRef ref) {
  return SaveDraft(
    repository: ref.watch(localAdRepositoryProvider),
  );
}

// Usage in UI
final result = await ref.read(saveDraftProvider).call(draft);
```

### State Management
```dart
@freezed
class AdCreationState with _$AdCreationState {
  const factory AdCreationState.initial() = _Initial;
  const factory AdCreationState.loading() = _Loading;
  const factory AdCreationState.analyzed(AIAnalysis analysis) = _Analyzed;
  const factory AdCreationState.error(String message) = _Error;
}
```

## Data Flow

```
User Action (UI)
    ↓
Provider/Notifier
    ↓
Use Case (Domain)
    ↓
Repository Interface (Domain)
    ↓
Repository Implementation (Data)
    ↓
Datasource (Data)
    ↓
External Source (Hive/API/File)
    ↓
Return through layers
    ↓
Update State
    ↓
UI Rebuilds
```

## Testing Strategy

1. **Unit Tests**: Domain layer (use cases, entities)
2. **Widget Tests**: Presentation layer (screens, widgets)
3. **Integration Tests**: Full flows (camera → AI → form → save)
4. **Target**: >80% coverage

## Common Integrations

### Hive CE (Local Storage)
```dart
// Initialize
await Hive.initFlutter();
Hive.registerAdapter(AdDraftModelAdapter());
await Hive.openBox<AdDraftModel>('ads');

// CRITICAL: Use Hive CE imports
import 'package:hive_ce/hive_ce.dart'; // ✅ Correct
// NOT: import 'package:hive/hive.dart'; // ❌ Wrong
```

### Gemini AI
```dart
final model = GenerativeModel(
  model: 'gemini-2.0-flash-exp',
  apiKey: apiKey,
);

final response = await model.generateContent([
  Content.multi([
    TextPart(prompt),
    DataPart('image/jpeg', imageBytes),
  ])
]);
```

### Go Router Navigation
```dart
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

## Performance Considerations

1. **Image Compression**: Max 2MB before AI processing
2. **Lazy Loading**: Load categories on-demand
3. **Debouncing**: Text inputs (2s delay)
4. **Caching**: Vegvesen API responses (24h)
5. **Const Widgets**: Use wherever possible
