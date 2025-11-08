# Norsktorget Card Component

This guide provides the standard implementation for cards.

## Standard Card (`Card`)

Use `Card` for grouping related information in a visually distinct container.

### Code Example

```dart
Card(
  // Style (elevation, shape, margin) is defined in AppTheme
  child: Padding(
    padding: const EdgeInsets.all(AppSpacing.md),
    child: Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text(
          'Card Title',
          style: Theme.of(context).textTheme.headlineSmall,
        ),
        const SizedBox(height: AppSpacing.sm),
        Text(
          'This is the content of the card. It provides more details about the topic mentioned in the title.',
          style: Theme.of(context).textTheme.bodyMedium,
        ),
      ],
    ),
  ),
);
```

## Clickable Card (`InkWell` + `Card`)

To make a card tappable with a ripple effect, wrap it with `InkWell`.

```dart
Card(
  clipBehavior: Clip.antiAlias, // Important for ripple effect to respect shape
  child: InkWell(
    onTap: () {
      // Action on tap
    },
    child: Padding(
      // ... same content as above
    ),
  ),
);
```
