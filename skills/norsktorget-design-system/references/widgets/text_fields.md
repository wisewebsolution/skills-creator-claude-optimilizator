# Norsktorget Text Field Component

This guide provides the standard implementation for text input fields.

## Standard Text Field (`TextFormField`)

Use `TextFormField` within a `Form` widget to get validation and state management.

### Code Example

```dart
TextFormField(
  controller: _myController,
  decoration: const InputDecoration(
    labelText: 'Label Text',
    hintText: 'Hint text for the user',
    // Border, padding, etc. are defined in AppTheme
  ),
  validator: (value) {
    if (value == null || value.isEmpty) {
      return 'This field cannot be empty';
    }
    return null;
  },
  onSaved: (value) {
    // Save the value
  },
)
```

## Text Field with Icon

```dart
TextFormField(
  decoration: const InputDecoration(
    labelText: 'Email Address',
    prefixIcon: Icon(Icons.email_outlined),
  ),
)
```

## Password Field

```dart
TextFormField(
  obscureText: _isPasswordVisible,
  decoration: InputDecoration(
    labelText: 'Password',
    prefixIcon: const Icon(Icons.lock_outline),
    suffixIcon: IconButton(
      icon: Icon(
        _isPasswordVisible ? Icons.visibility_off : Icons.visibility,
      ),
      onPressed: () {
        setState(() {
          _isPasswordVisible = !_isPasswordVisible;
        });
      },
    ),
  ),
)
```
