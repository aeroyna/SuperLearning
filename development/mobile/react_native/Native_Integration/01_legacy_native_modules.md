# Legacy Native Modules

Before the New Architecture, writing a Native Module involved registering a class with the React Bridge.

## Anatomy of a Native Module

### Android (Java/Kotlin)
1.  **Module Class:** Extends `ReactContextBaseJavaModule`.
2.  **getName():** Returns the string name used in JS (e.g., "CalendarModule").
3.  **@ReactMethod:** Annotate methods exposed to JS.
    *   *Constraint:* Arguments must be readable by the Bridge (ReadableMap, ReadableArray, String, Boolean, Double).
4.  **Package Class:** Extends `ReactPackage` to register the Module with the Application.

```java
@ReactMethod
public void createCalendarEvent(String name, String location, Promise promise) {
    try {
       // Native Logic
       promise.resolve("Success");
    } catch(Exception e) {
       promise.reject("Error", e);
    }
}
```

### iOS (Objective-C/Swift)
1.  **Macros:** Use `RCT_EXPORT_MODULE()` and `RCT_EXPORT_METHOD()`.
2.  **Bridge Header:** Swift requires exposing the class to Objective-C.

### JavaScript Side
```javascript
import { NativeModules } from 'react-native';
const { CalendarModule } = NativeModules;
CalendarModule.createCalendarEvent('Party', 'My House');
```

## Limitations
*   **Startup:** All Native Modules are initialized at app startup, slowing down launch time, even if the module isn't used immediately.
*   **Typing:** No guarantee that the JS call matches the Native signature until runtime crash.
