---
name: norsktorget-testing
description: Use when writing tests for the Norsktorget Flutter app. This skill provides advanced, tutorial-style patterns for testing complex scenarios involving Riverpod 3.0 (AsyncNotifier), Hive CE, and mocked AI (Gemini) services, complementing the main testing best practices.
---

# Advanced Testing Guide for Norsktorget

## Overview

This skill is a practical, tutorial-style guide for writing advanced tests in the Norsktorget project. It complements the foundational knowledge in `docs/testing/best-practices.md` by providing step-by-step solutions for complex, project-specific scenarios.

**Core Principle:** Tests should be realistic and cover the entire flow of a feature, from user interaction in the UI to data layer operations.

## When to Use

- When testing widgets that depend on `AsyncNotifier` providers.
- When writing tests that involve reading from or writing to the local database (Hive CE).
- When testing UI interactions that trigger AI (Gemini) service calls.
- When writing a full integration test for a user feature flow within the test environment.

## 1. Tutorial: Testing Widgets with `AsyncNotifier`

This pattern is **required** for testing any UI that depends on asynchronous data.

**Scenario:** Test a widget that displays a list of ads fetched by an `AsyncNotifier`.

```dart
// test/features/ads/presentation/ad_list_widget_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:mocktail/mocktail.dart';

// 1. Mock the repository dependency
class MockAdRepository extends Mock implements AdRepository {}

void main() {
  late MockAdRepository mockAdRepository;

  setUp(() {
    mockAdRepository = MockAdRepository();
  });

  testWidgets('Displays loading indicator, then data', (tester) async {
    // Arrange
    final ads = [Ad(id: '1', title: 'Test Ad')];
    when(() => mockAdRepository.fetchAds()).thenAnswer((_) async {
      await Future.delayed(const Duration(milliseconds: 100)); // Simulate network latency
      return ads;
    });

    // Act
    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          adRepositoryProvider.overrideWithValue(mockAdRepository),
        ],
        child: const MaterialApp(home: AdListView()),
      ),
    );

    // Assert: Initial state is loading
    expect(find.byType(CircularProgressIndicator), findsOneWidget);

    // Act: Wait for the future to complete
    await tester.pumpAndSettle();

    // Assert: Final state shows data
    expect(find.text('Test Ad'), findsOneWidget);
    expect(find.byType(CircularProgressIndicator), findsNothing);
  });

  testWidgets('Displays error message on failure', (tester) async {
    // Arrange
    when(() => mockAdRepository.fetchAds()).thenThrow(Exception('Network Error'));

    // Act
    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          adRepositoryProvider.overrideWithValue(mockAdRepository),
        ],
        child: const MaterialApp(home: AdListView()),
      ),
    );

    // Wait for future to complete
    await tester.pumpAndSettle();

    // Assert: Error state is displayed
    expect(find.text('Error: Exception: Network Error'), findsOneWidget);
    expect(find.byType(CircularProgressIndicator), findsNothing);
  });
}
```

## 2. Tutorial: Testing with a Real Hive CE Database

This pattern is required for testing repositories that interact with local storage. Do not mock the Hive Box.

**Scenario:** Test the LocalAdRepository to ensure it saves and retrieves drafts correctly.

```dart
// test/features/ads/data/repositories/local_ad_repository_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:hive_ce/hive_ce.dart';
import 'dart:io';

void main() {
  group('LocalAdRepository with Hive', () {
    const testPath = 'test/hive_test_data';

    // 1. Initialize Hive in a temporary directory for tests
    setUpAll(() async {
      Hive.init(testPath);
      // Register all necessary adapters
      Hive.registerAdapter(AdDraftAdapter());
    });

    // 2. Clean up the database after each test
    tearDown(() async {
      await Hive.deleteFromDisk();
    });

    // 3. Close Hive after all tests are done
    tearDownAll(() async {
      await Hive.close();
      // Clean up the directory
      final dir = Directory(testPath);
      if (await dir.exists()) {
        await dir.delete(recursive: true);
      }
    });

    test('saveDraft writes to the box and getDraft retrieves it', () async {
      // Arrange
      final repository = LocalAdRepository();
      final draft = AdDraft(id: '123', title: 'My Draft');

      // Act
      await repository.saveDraft(draft);
      final retrievedDraft = await repository.getDraft('123');

      // Assert
      expect(retrievedDraft, isNotNull);
      expect(retrievedDraft!.title, 'My Draft');
    });
  });
}
```

## 3. Tutorial: Testing AI Integration in Widgets

This pattern is required for testing UI that interacts with AI services.

**Scenario:** Test a widget that calls the GeminiService to get suggestions when a button is clicked.

```dart
// test/features/ai/presentation/ai_suggester_widget_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:mocktail/mocktail.dart';

// 1. Mock the AI service
class MockGeminiService extends Mock implements GeminiService {}

void main() {
  late MockGeminiService mockGeminiService;

  setUp(() {
    mockGeminiService = MockGeminiService();
  });

  testWidgets('shows suggestions after successful AI call', (tester) async {
    // Arrange
    final suggestions = AiAnalysis(suggestion: 'A great title');
    when(() => mockGeminiService.getSuggestions(any())).thenAnswer((_) async => suggestions);

    // Act
    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          geminiServiceProvider.overrideWithValue(mockGeminiService),
        ],
        child: const MaterialApp(home: AiSuggesterWidget()),
      ),
    );

    // Assert: Initially, no suggestions are visible
    expect(find.text('A great title'), findsNothing);

    // Act: Simulate user tapping the "Get Suggestions" button
    await tester.tap(find.byType(ElevatedButton));
    await tester.pump(); // Start the loading state

    // Assert: Loading indicator is shown
    expect(find.byType(CircularProgressIndicator), findsOneWidget);

    await tester.pumpAndSettle(); // Wait for the future to complete

    // Assert: Suggestions are now visible
    expect(find.text('A great title'), findsOneWidget);
    expect(find.byType(CircularProgressIndicator), findsNothing);

    // Verify the service was called
    verify(() => mockGeminiService.getSuggestions('some input')).called(1);
  });
}
```

## 4. Tutorial: Full Integration Test for "Create Ad" Flow

This pattern combines all previous techniques to test a complete feature flow.

```dart
// test/integration/create_ad_flow_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:mocktail/mocktail.dart';

// Mock all external dependencies
class MockAdRepository extends Mock implements AdRepository {}
class MockGeminiService extends Mock implements GeminiService {}

void main() {
  testWidgets('Full ad creation flow', (tester) async {
    // 1. Arrange: Setup mocks and overrides
    final mockAdRepository = MockAdRepository();
    final mockGeminiService = MockGeminiService();

    when(() => mockAdRepository.saveDraft(any())).thenAnswer((_) async {});
    when(() => mockGeminiService.getSuggestions(any())).thenAnswer((_) async => AiAnalysis(suggestion: 'AI Title'));

    // 2. Act: Pump the root widget with all providers overridden
    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          adRepositoryProvider.overrideWithValue(mockAdRepository),
          geminiServiceProvider.overrideWithValue(mockGeminiService),
        ],
        child: const MyApp(), // Your app's root widget
      ),
    );

    // 3. Act: Simulate user interactions
    await tester.tap(find.byIcon(Icons.add));
    await tester.pumpAndSettle();

    await tester.enterText(find.byKey(const Key('title_field')), 'My Ad');
    await tester.pump();

    await tester.tap(find.byKey(const Key('get_ai_suggestions_button')));
    await tester.pumpAndSettle();

    // 4. Assert: Verify UI updates and service calls
    expect(find.text('AI Title'), findsOneWidget); // AI suggestion is displayed

    await tester.tap(find.byKey(const Key('save_draft_button')));
    await tester.pump();

    // 5. Assert: Verify side-effects
    verify(() => mockAdRepository.saveDraft(any(that: isA<AdDraft>().having((d) => d.title, 'title', 'My Ad')))).called(1);
  });
}
```

## 5. Golden Tests for Visual Regression

Use golden tests to catch unintended UI changes.

```dart
// test/goldens/ad_card_golden_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:golden_toolkit/golden_toolkit.dart';

void main() {
  testGoldens('AdCard should render correctly', (tester) async {
    // Arrange
    final ad = Ad(title: 'Beautiful Vintage Chair', price: 1500);

    // Act
    await tester.pumpWidgetBuilder(
      AdCard(ad: ad),
      surfaceSize: const Size(400, 200),
    );

    // Assert
    await screenMatchesGolden(tester, 'ad_card_default');
  });
}
```

## Common Mistakes / Anti-Patterns

| ❌ Anti-Pattern | ✅ Required Norsktorget Pattern |
|:---|:---|
| Testing UI in isolation without state. | ALWAYS wrap widgets in ProviderScope with necessary overrides to test them with realistic state. |
| Mocking Hive.box() directly. | ALWAYS use a real, temporary Hive instance for repository tests to ensure true-to-life behavior. See Tutorial 2. |
| Not testing loading/error states of AsyncNotifier. | ALWAYS write separate tests for data, loading, and error states to ensure a robust UI. See Tutorial 1. |
| Testing only the final state of a flow. | ALWAYS test the entire flow, including intermediate states (like loading spinners) and verify that dependencies were called correctly. See Tutorial 4. |
