# Platform Specific Code

Cross-platform development often requires divergence. You want 90% shared code, but the 10% that differs (navigation patterns, specific native UI components) is critical for a "native feel".

## 1. File Extension (`.ios.js` / `.android.js`)
The Metro bundler automatically resolves files based on the platform.
*   If you import `import Header from './Header';`
*   And you have `Header.ios.js` and `Header.android.js`.
*   Metro will bundle the correct file for the build target.

**Use Case:** Completely different implementations of a component.
*   *Example:* A DatePicker where iOS uses a scroll wheel and Android uses a calendar modal.

## 2. The `Platform` Module
For minor logic branches within the same file.

```javascript
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  header: {
    height: 60,
    marginTop: Platform.OS === 'ios' ? 20 : 0, // Handle status bar
    ...Platform.select({
      ios: { backgroundColor: 'red' },
      android: { backgroundColor: 'blue' },
      default: { backgroundColor: 'white' } // Web or others
    }),
  },
});
```

### Version Detection
`Platform.Version` returns the OS version.
*   **Android:** Returns the API Level (e.g., 30 for Android 11).
*   **iOS:** Returns a string (e.g., "14.2").

## 3. Environment Specifics
*   **__DEV__:** A global boolean variable. True in development, false in production builds. Useful for logging or dev-tools.
