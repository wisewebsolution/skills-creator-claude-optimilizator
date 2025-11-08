# Norsktorget Spacing System

This guide defines the standard spacing scale for all padding, margins, and gaps to ensure a consistent layout.

## Spacing Scale

All spacing values **must** be accessed via the `AppSpacing` class. Do not use hardcoded numeric values.

| Name | Constant | Value | Usage |
| :--- | :--- | :--- | :--- |
| X-Small | `AppSpacing.xs` | 4.0 | Small gaps between icons and text |
| Small | `AppSpacing.sm` | 8.0 | Gaps between related elements, icon padding |
| Medium | `AppSpacing.md` | 16.0 | Default padding for containers, cards, list items |
| Large | `AppSpacing.lg` | 24.0 | Padding for screen edges, gaps between sections |
| X-Large | `AppSpacing.xl` | 32.0 | Large gaps between major UI regions |
| XX-Large | `AppSpacing.xxl`| 48.0 | Very large vertical spacing |

## Example Usage

### Padding and Margin

```dart
// Inside a build method
return Container(
  // Use EdgeInsets with AppSpacing constants
  padding: const EdgeInsets.symmetric(
    vertical: AppSpacing.sm,
    horizontal: AppSpacing.md,
  ),
  margin: const EdgeInsets.only(bottom: AppSpacing.lg),
  child: const Text('Hello'),
);
```

### Sized Box for Gaps

Using `SizedBox` is the preferred way to create space between widgets in a `Column` or `Row`.

```dart
// Inside a Column
Column(
  children: [
    const Text('Title'),
    const SizedBox(height: AppSpacing.sm), // Consistent gap
    const Text('Subtitle'),
    const SizedBox(height: AppSpacing.md),
    ElevatedButton(onPressed: () {}, child: const Text('Action')),
  ],
)
```

### Radius

The spacing scale should also be used for corner radii to maintain visual harmony.

```dart
// Inside a BoxDecoration
decoration: BoxDecoration(
  borderRadius: BorderRadius.circular(AppSpacing.sm), // 8.0 radius
),
```
