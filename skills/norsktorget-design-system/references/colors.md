# Norsktorget Color System

This guide defines the official color palette and its usage within the `AppTheme`.

## Core Palette

Inspired by the Norwegian flag, our palette is clean, modern, and accessible.

| Name | Hex | Role |
| :--- | :--- | :--- |
| **Norwegian Blue** | `#003087` | Primary actions, branding |
| **Norwegian Red** | `#C8102E` | Destructive actions, accents |
| **Pure White** | `#FFFFFF` | Main surface, text on dark |
| **Light Grey** | `#F5F5F5` | Main background |
| **Dark Grey** | `#212121` | Main text on light |

## Theme-Based Color Usage

All colors **must** be accessed via `Theme.of(context).colorScheme`. Do not use the hex values directly.

### Main Colors

| Theme Role | How to Access | Usage |
| :--- | :--- | :--- |
| `primary` | `theme.colorScheme.primary` | Primary buttons, links, active indicators |
| `onPrimary` | `theme.colorScheme.onPrimary` | Text/icons on primary color backgrounds |
| `secondary` | `theme.colorScheme.secondary` | Floating Action Buttons, accents |
| `onSecondary`| `theme.colorScheme.onSecondary`| Text/icons on secondary color backgrounds |
| `error` | `theme.colorScheme.error` | Error messages, destructive action icons |
| `onError` | `theme.colorScheme.onError` | Text/icons on error color backgrounds |

### Surface & Background Colors

| Theme Role | How to Access | Usage |
| :--- | :--- | :--- |
| `background` | `theme.colorScheme.background` | The main background of most screens |
| `surface` | `theme.colorScheme.surface` | Surface of components like Cards, BottomSheets |
| `onBackground`| `theme.colorScheme.onBackground`| Text/icons on the main background |
| `onSurface` | `theme.colorScheme.onSurface` | Main text color on components |
| `onSurfaceVariant` | `theme.colorScheme.onSurfaceVariant` | Secondary text, subtitles, hints |

### Example Usage

```dart
// Inside a build method
final theme = Theme.of(context);

return Scaffold(
  backgroundColor: theme.colorScheme.background,
  appBar: AppBar(
    backgroundColor: theme.colorScheme.surface,
    title: Text('Title', style: TextStyle(color: theme.colorScheme.onSurface)),
  ),
  body: Card(
    color: theme.colorScheme.surface,
    child: Text(
      'Secondary text',
      style: TextStyle(color: theme.colorScheme.onSurfaceVariant),
    ),
  ),
  floatingActionButton: FloatingActionButton(
    backgroundColor: theme.colorScheme.secondary,
    child: Icon(Icons.add, color: theme.colorScheme.onSecondary),
    onPressed: () {},
  ),
);
```

## Custom Semantic Colors

For specific states, use the `CustomColors` theme extension.

| Name | How to Access | Usage |
| :--- | :--- | :--- |
| `success` | `customColors.success` | Success messages, validation success |
| `warning` | `customColors.warning` | Warnings, medium alerts |
| `info` | `customColors.info` | Informational messages, hints |

### Example

```dart
// Inside a build method
final customColors = Theme.of(context).extension<CustomColors>()!;

return Container(
  padding: const EdgeInsets.all(AppSpacing.md),
  decoration: BoxDecoration(
    color: customColors.success.withOpacity(0.1),
    border: Border.all(color: customColors.success),
    borderRadius: BorderRadius.circular(AppSpacing.sm),
  ),
  child: Text('Success!', style: TextStyle(color: customColors.success)),
);
```
