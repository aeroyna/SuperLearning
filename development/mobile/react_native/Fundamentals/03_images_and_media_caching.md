# Images and Media Caching

Handling images on mobile is significantly more resource-intensive than on the web. A 4K image loaded into a standard `<img>` tag might slow a browser; on mobile, decoding it can crash the app (OOM - Out Of Memory).

## The `<Image>` Component
React Native provides an `<Image>` component that communicates with native image libraries (Fresco on Android, ImageIO on iOS).

### Source Types
1.  **Local Assets:** Bundled with the app binary (icons, splash screens).
    *   `source={require('./logo.png')}`
    *   Processed by the packager (Metro) to serve correct densities (@2x, @3x).
2.  **Network Images:** Loaded from a URL.
    *   `source={{ uri: 'https://...' }}`
    *   **Requires Dimensions:** Unlike web, you *must* specify width and height for network images, as RN doesn't know the size until download (and won't reflow layout automatically).

## Caching Mechanisms

### iOS Caching
iOS relies on the `NSURLCache` and internal ImageIO caching. It's generally automatic but less configurable out-of-the-box.

### Android Caching (Fresco)
RN uses **Fresco** under the hood. Fresco is powerful and uses a three-tier cache:
1.  **Bitmap Memory Cache:** Decoded images ready to display. Fast, but uses RAM.
2.  **Encoded Memory Cache:** Compressed images in memory.
3.  **Disk Cache:** Files saved on local storage.

## Optimization Strategies
*   **ResizeMode:** `cover`, `contain`, `stretch`. Understanding how these affect GPU rendering is key.
*   **Prefetching:** `Image.prefetch(url)` downloads the image to disk cache before the view renders.
*   **Third-party Libraries:** For advanced needs (progressive loading, aggressive caching), libraries like `react-native-fast-image` (which wraps SDWebImage on iOS and Glide on Android) are standard industry choices over the built-in component.
