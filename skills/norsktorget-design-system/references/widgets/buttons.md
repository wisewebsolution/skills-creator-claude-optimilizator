# Norsktorget Button Components

This guide provides the standard implementation for different button types.

## 1. Primary Button (`ElevatedButton`)

Use for the primary, most important action on a screen.

### Code Example

```dart
ElevatedButton(
  onPressed: () {
    // Primary action
  },
  style: ElevatedButton.styleFrom(
    // Style is defined in AppTheme, no need to specify here
  ),
  child: const Text('Primary Action'),
);
```

## 2. Secondary Button (`OutlinedButton`)

Use for secondary actions that are less important than the primary one.

### Code Example

```dart
OutlinedButton(
  onPressed: () {
    // Secondary action
  },
  child: const Text('Secondary Action'),
);
```

## 3. Text Button (`TextButton`)

Use for the least prominent actions, often for "Cancel" or "Dismiss".

### Code Example

```dart
TextButton(
  onPressed: () {
    // Tertiary action
  },
  child: const Text('Cancel'),
);
```

## 4. Destructive Action Button

For actions that delete data or have irreversible consequences, use a primary or text button styled with the error color.

### Code Example

```dart
// Using ElevatedButton
ElevatedButton(
  onPressed: () {
    // Destructive action
  },
  style: ElevatedButton.styleFrom(
    backgroundColor: Theme.of(context).colorScheme.error,
    foregroundColor: Theme.of(context).colorScheme.onError,
  ),
  child: const Text('Delete'),
);

// Using TextButton
TextButton(
  onPressed: () {
    // Destructive action
  },
  style: TextButton.styleFrom(
    foregroundColor: Theme.of(context).colorScheme.error,
  ),
  child: const Text('Delete'),
);
```
