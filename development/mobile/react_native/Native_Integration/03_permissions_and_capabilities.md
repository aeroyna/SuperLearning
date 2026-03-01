# Permissions and Capabilities

Accessing sensitive hardware (Camera, Mic, GPS) requires explicit user permission.

## Configuration (Static)

### iOS (`Info.plist`)
You must add a usage description string. If you access the camera but don't have `NSCameraUsageDescription` in your plist, the app will crash immediately upon access.
*   *Key:* `NSCameraUsageDescription`
*   *Value:* "We need access to your camera to scan QR codes."

### Android (`AndroidManifest.xml`)
You must declare the permission.
*   `<uses-permission android:name="android.permission.CAMERA" />`

## Runtime Request (Dynamic)
Since Android 6.0 (API 23), declaring it in the manifest isn't enough. You must ask the user at runtime.

### React Native Permissions Library
While RN has `PermissionsAndroid`, the community standard is `react-native-permissions` because it handles both platforms with a unified API and supports "Blocked" states (where the user has permanently denied permission and must be directed to settings).

```javascript
import { check, request, PERMISSIONS, RESULTS } from 'react-native-permissions';

check(PERMISSIONS.IOS.CAMERA).then((result) => {
  switch (result) {
    case RESULTS.UNAVAILABLE: // Hardware missing
      break;
    case RESULTS.DENIED: // Requestable
      request(PERMISSIONS.IOS.CAMERA).then(...)
      break;
    case RESULTS.GRANTED:
      break;
    case RESULTS.BLOCKED: // Open Settings
      break;
  }
});
```
