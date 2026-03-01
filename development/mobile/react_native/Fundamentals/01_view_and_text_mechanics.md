# View and Text Mechanics

In React Native, `<View>` and `<Text>` are not HTML `div`s and `span`s. They are abstractions that render distinct native UI widgets.

## The View Component
The `<View>` is the fundamental container.
*   **Android:** Maps to `android.view.ViewGroup` (specifically `ReactViewGroup`).
*   **iOS:** Maps to `UIView`.

**Key Concept:** Views in RN are designed to be strictly containers. They handle layout, borders, and background colors. They do *not* handle text rendering directly (unlike a `div` containing text in HTML).

### Clipping (overflow)
*   **iOS:** `overflow: hidden` maps to `clipsToBounds = true`.
*   **Android:** Traditionally complex, but modern RN handles clipping efficiently.

## The Text Component
Text rendering is one of the most complex parts of mobile UI.
*   **Android:** Maps to `TextView`.
*   **iOS:** Maps to `NSTextStorage`, `NSLayoutManager`, and `NSTextContainer` (TextKit).

### Nesting and Inheritance
On the web, CSS inheritance is pervasive. In React Native, style inheritance is limited strictly to **Text subtrees**.

```javascript
// This works (Text inside Text inherits color/font)
<Text style={{ color: 'red' }}>
  Hello <Text style={{ fontWeight: 'bold' }}>World</Text>
</Text>

// This does NOT work (Text inside View does not inherit View's style)
<View style={{ color: 'red' }}>
  <Text>Hello World</Text> {/* Will be black/default */}
</View>
```

### The "Virtual" Text Node
When you nest Text:
```javascript
<Text>
  Part A
  <Text>Part B</Text>
</Text>
```
React Native often optimizes this into a single Native View (e.g., a single `TextView` with a `SpannableString` on Android) rather than creating two distinct Native Views. This "flattening" is crucial for memory performance.
