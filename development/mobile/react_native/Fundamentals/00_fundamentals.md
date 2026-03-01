# Fundamentals of React Native

This chapter covers the essential building blocks of the framework. We look at how basic components map to their native counterparts and how the layout system functions internally.

## In this chapter, you will learn:

*   **[2.1 View and Text Mechanics](01_view_and_text_mechanics.md)**
    *   The mapping of `<View>` to `ViewGroup`/`UIView`.
    *   Why `<Text>` behaves differently than on the web (nesting, inheritance).

*   **[2.2 Flexbox Layout Engine](02_flexbox_layout_engine.md)**
    *   How 'Yoga' implements Flexbox in C++.
    *   Differences between Web Flexbox and Native Flexbox (defaults, missing properties).

*   **[2.3 Images and Media Caching](03_images_and_media_caching.md)**
    *   Image processing, resizing, and caching strategies.
    *   Handling remote vs local assets.

*   **[2.4 Platform Specific Code](04_platform_specific_code.md)**
    *   File extensions (`.ios.js`, `.android.js`).
    *   The `Platform` module.
    *   Conditional styling.
