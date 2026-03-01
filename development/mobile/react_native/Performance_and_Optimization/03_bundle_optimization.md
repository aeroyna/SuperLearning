# Bundle Optimization

The JavaScript bundle is the file downloaded and executed by the JS engine. A large bundle means:
1.  Slower download (for OTA updates).
2.  Slower parsing/startup time.

## Metro Bundler
Metro is the bundler for React Native (like Webpack/Vite for web).

## Optimization Techniques

### 1. Inline Requires
Instead of:
```javascript
import MyHugeModule from './MyHugeModule'; // Loaded at startup
```
Use:
```javascript
const MyScreen = () => {
  const handlePress = () => {
     const MyHugeModule = require('./MyHugeModule').default; // Loaded ON DEMAND
     MyHugeModule.doSomething();
  };
};
```
Metro can automate this via configuration (`inlineRequires: true`), deferring the parsing of modules until they are actually executed.

### 2. Hermes Bytecode
As mentioned in the Architecture section, Hermes pre-compiles the JS bundle into Bytecode. This doesn't reduce the *file size* on disk necessarily (bytecode can be larger than minified source), but it drastically reduces the *memory size* and *execution time* because the engine doesn't need to parse text.

### 3. Tree Shaking / Dead Code Elimination
Ensure you are not importing massive libraries (like `lodash`) when you only need one function.
*   *Bad:* `import _ from 'lodash';`
*   *Good:* `import map from 'lodash/map';`
