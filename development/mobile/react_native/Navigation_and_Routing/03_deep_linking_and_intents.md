# Deep Linking and Intents

Deep linking allows users to open your app from a URL (e.g., from an email or a website) directly to a specific screen.

## The Mechanisms

### iOS: Universal Links & URL Schemes
*   **URL Schemes (`myapp://`):** Easy to set up, but not unique. Anyone can register `myapp://`.
*   **Universal Links (`https://myapp.com/post/1`):** Secure. Requires verifying domain ownership via an Apple App Site Association (AASA) file on your server. When the user clicks the HTTP link, iOS opens the app instead of Safari.

### Android: Intents & App Links
*   **Deep Links:** Custom schemes (`myapp://`).
*   **App Links:** The Android equivalent of Universal Links. Requires `assetlinks.json` on the server to verify ownership.

## Handling in React Native
1.  **Native Setup:** You must configure `Info.plist` (iOS) and `AndroidManifest.xml` (Android) to listen for the schemes/domains.
2.  **JS Listener:**
    ```javascript
    import { Linking } from 'react-native';

    // Cold Launch (App was dead)
    Linking.getInitialURL().then(url => { ... });

    // Warm Launch (App was backgrounded)
    Linking.addEventListener('url', ({ url }) => { ... });
    ```
3.  **Integration with Navigation:** React Navigation has a `linking` prop config that automatically parses the URL path (e.g., `/user/:id`) and dispatches the correct navigation action (Navigate to 'User', params: { id }).
