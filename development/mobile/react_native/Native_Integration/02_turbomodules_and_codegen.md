# TurboModules and Codegen

TurboModules are the Native Module system for the New Architecture.

## Key Differences

### 1. Lazy Loading
TurboModules are lazy. If you have a massive "ARModule" but the user is on the "Settings" screen, that module is not loaded into memory. This significantly improves startup time.

### 2. JSI Binding
TurboModules use JSI to expose C++ functions directly to JS. This allows synchronous calling (if designed that way) and removes the JSON serialization bridge cost.

## Codegen
Writing JSI C++ code manually is hard and error-prone. **Codegen** automates this.

### The Workflow
1.  **Define Spec (TypeScript/Flow):** You write a typed interface for your module in a `Native*` file.
    ```typescript
    // NativeCalendar.ts
    export interface Spec extends TurboModule {
      createEvent(name: string, location: string): Promise<string>;
    }
    export default TurboModuleRegistry.get<Spec>('Calendar');
    ```
2.  **Run Build:** The build system (Gradle/CocoaPods) invokes Codegen.
3.  **Generate Glue Code:** Codegen creates C++ and Java/ObjC interfaces.
4.  **Implement Native:** You write the native class that *implements* the generated interface. If you change the TS spec, the native build will fail at compile time (Safety!).

This ensures that your JavaScript types and Native types are always 100% in sync.
