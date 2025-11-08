---
name: norsktorget-design-system
description: Use when creating or modifying Flutter UI for the Norsktorget project. This skill provides the official design system, including color palettes, typography, spacing, and reusable widget patterns based on Material 3, ensuring visual consistency across the application.
---

# Norsktorget Design System

## Overview

This skill is the single source of truth for all UI/UX design and implementation in the Norsktorget Flutter application. It ensures that every component is visually consistent, on-brand, and follows modern Material 3 best practices.

**Core Principle:** All UI code MUST adhere to the standards and patterns defined in this skill. Do not use hardcoded colors, font sizes, or spacing values.

## When to Use

- When creating any new Flutter widget or screen.
- When modifying the appearance of an existing UI component.
- When choosing colors, fonts, or spacing for a new design.
- When you need a code example for a standard Norsktorget UI component.

## 1. Core Principles

1.  **Material 3 First**: All components must be based on the Material 3 library (`Theme.of(context)`).
2.  **Theme-Driven**: All styling values (colors, text styles) **must** be sourced from the `AppTheme` and its extensions.
3.  **Consistency**: Use the defined spacing, colors, and typography to maintain a consistent look and feel.
4.  **Responsiveness**: Widgets should adapt gracefully to different screen sizes.

## 2. Reference Guides (The "How-To")

Detailed specifications are located in the `references/` directory. **Always consult these files for specific values.**

-   **Colors (`references/colors.md`):** Defines the complete Norwegian-inspired color palette, including primary, secondary, semantic (success, warning, error), and neutral colors with their theme roles.
-   **Typography (`references/typography.md`):** Specifies the font scales (`displayLarge`, `headlineMedium`, `bodySmall`, etc.) and when to use them.
-   **Spacing (`references/spacing.md`):** Defines the standard spacing scale (`AppSpacing.xs`, `sm`, `md`, `lg`, `xl`) to be used for all padding and margins.

## 3. Widget Library (The "Building Blocks")

The `references/widgets/` directory contains code examples and usage guidelines for standard Norsktorget components.

-   **Buttons (`references/widgets/buttons.md`):** How to create primary, secondary, and text buttons.
-   **Cards (`references/widgets/cards.md`):** Standard card implementation with correct elevation, shape, and padding.
-   **Text Fields (`references/widgets/text_fields.md`):** Standard implementation for text inputs, including decoration and validation styles.

## 4. Quick Start: Building a New Widget

Follow this workflow when creating a new widget to ensure it aligns with the design system.

### Step 1: Use `ConsumerWidget`

All widgets that may need to react to state or theme changes should be `ConsumerWidget`.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class MyNewWidget extends ConsumerWidget {
  const MyNewWidget({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ...
  }
}
```

### Step 2: Access Theme and Colors

Never use `Colors.blue` or hardcoded hex values. Always get colors from the theme.

```dart
// Inside the build method:
final theme = Theme.of(context);
final customColors = theme.extension<CustomColors>()!; // For custom semantic colors

return Container(
  color: theme.colorScheme.primaryContainer,
  child: Text(
    'Hello',
    style: TextStyle(color: customColors.success),
  ),
);
```

**Reference:** See `references/colors.md` for a full list of available theme colors.

### Step 3: Use Standard Typography

Do not define `TextStyle` manually. Use the predefined text styles from the theme.

```dart
// Inside the build method:
final textTheme = Theme.of(context).textTheme;

return Column(
  children: [
    Text('This is a title', style: textTheme.headlineSmall),
    Text('This is body text', style: textTheme.bodyMedium),
  ],
);
```

**Reference:** See `references/typography.md` for all available text styles.

### Step 4: Apply Standard Spacing

Use the `AppSpacing` constants for all padding, margins, and `SizedBox` dimensions.

```dart
// Inside the build method:
return Padding(
  padding: const EdgeInsets.all(AppSpacing.md), // 16.0
  child: Column(
    children: [
      const Text('Some text'),
      const SizedBox(height: AppSpacing.sm), // 8.0
      ElevatedButton(onPressed: () {}, child: const Text('Submit')),
    ],
  ),
);
```

**Reference:** See `references/spacing.md` for the full spacing scale.

### Step 5: Use Standard Components

Before creating a new component from scratch, check if a standard implementation exists.

```dart
// Use a standard Norsktorget card
return NorsktorgetCard(
  child: Text('This card is styled correctly by default.'),
);
```

**Reference:** See `references/widgets/` for available components.

## Common Mistakes / Anti-Patterns

| ❌ Anti-Pattern | ✅ Required Norsktorget Pattern |
|:---|:---|
| `color: Colors.red` | `color: Theme.of(context).colorScheme.error` |
| `fontSize: 16, fontWeight: FontWeight.bold` | `style: Theme.of(context).textTheme.titleMedium` |
| `padding: const EdgeInsets.all(10.0)` | `padding: const EdgeInsets.all(AppSpacing.sm)` (lub md) |
| Tworzenie własnego widgetu przycisku od zera. | Użycie `ElevatedButton.styleFrom` lub dedykowanego `NorsktorgetPrimaryButton`. |
