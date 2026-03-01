# Native Integration

Sometimes JavaScript isn't enough. You might need to access a specific platform API (HealthKit, Bluetooth, ARCore) or reuse an existing native SDK.

## In this chapter, you will learn:

*   **[5.1 Legacy Native Modules](01_legacy_native_modules.md)**
    *   How to write Java/Objective-C modules that talk to JS via the Bridge.
    *   The `ReactContextBaseJavaModule` and `RCTBridgeModule` patterns.

*   **[5.2 TurboModules and Codegen](02_turbomodules_and_codegen.md)**
    *   The typed, synchronous future.
    *   Defining specs in TypeScript and generating C++ bindings automatically.

*   **[5.3 Permissions and Capabilities](03_permissions_and_capabilities.md)**
    *   Handling runtime permissions (Camera, Location) on iOS vs Android.
    *   Modifying `Info.plist` and `AndroidManifest.xml`.
