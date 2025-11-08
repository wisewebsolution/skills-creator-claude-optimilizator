# Norsktorget Typography System

This guide defines the text styles used throughout the application. All text styling **must** use these predefined styles from the theme.

## Font Scale

We use the standard Material 3 type scale. Do not define `TextStyle` with custom `fontSize` or `fontWeight`.

| Style Name | How to Access | Example Usage |
| :--- | :--- | :--- |
| `displayLarge` | `textTheme.displayLarge` | Hero numbers, very large headlines |
| `displayMedium`| `textTheme.displayMedium`| Large headlines |
| `displaySmall` | `textTheme.displaySmall` | Headlines |
| `headlineLarge`| `textTheme.headlineLarge`| Sub-headlines |
| `headlineMedium`|`textTheme.headlineMedium`| Section titles |
| `headlineSmall`| `textTheme.headlineSmall`| Card titles, dialog titles |
| `titleLarge` | `textTheme.titleLarge` | App bar titles |
| `titleMedium` | `textTheme.titleMedium` | List item titles, primary button text |
| `titleSmall` | `textTheme.titleSmall` | |
| `bodyLarge` | `textTheme.bodyLarge` | Main body text, paragraphs |
| `bodyMedium` | `textTheme.bodyMedium` | Secondary body text, descriptions |
| `bodySmall` | `textTheme.bodySmall` | Captions, helper text, legal text |
| `labelLarge` | `textTheme.labelLarge` | Buttons |
| `labelMedium` | `textTheme.labelMedium`| |
| `labelSmall` | `textTheme.labelSmall` | Overline text |

## Example Usage

```dart
// Inside a build method
final textTheme = Theme.of(context).textTheme;
final colorScheme = Theme.of(context).colorScheme;

return Padding(
  padding: const EdgeInsets.all(AppSpacing.md),
  child: Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: [
      Text('Main Headline', style: textTheme.headlineMedium),
      const SizedBox(height: AppSpacing.sm),
      Text(
        'This is the main paragraph text for the application. It should be easy to read and have good contrast.',
        style: textTheme.bodyLarge,
      ),
      const SizedBox(height: AppSpacing.xs),
      Text(
        'This is a caption or some less important helper text.',
        style: textTheme.bodySmall?.copyWith(
          color: colorScheme.onSurfaceVariant, // Use copyWith to change color
        ),
      ),
    ],
  ),
);
```

## Custom Text Styles

For special cases, use the `CustomTextStyles` theme extension.

| Name | How to Access | Usage |
| :--- | :--- | :--- |
| `price` | `customTextStyles.price` | For displaying prices in a prominent way |
| `link` | `customTextStyles.link` | For hyperlinks |

### Example

```dart
// Inside a build method
final customTextStyles = Theme.of(context).extension<CustomTextStyles>()!;

return Column(
  children: [
    Text('10 000 kr', style: customTextStyles.price),
    Text('Read more', style: customTextStyles.link),
  ],
);
```
