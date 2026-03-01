# List Virtualization

Displaying long lists (tweets, contacts, products) is the most common performance bottleneck.

## The Problem with `ScrollView`
If you put 1,000 items in a `ScrollView` or just `.map()` them in a View, React Native renders 1,000 Native Views immediately. This consumes massive memory and CPU, taking seconds to load.

## VirtualizedList (FlatList / SectionList)
`FlatList` uses virtualization. It only renders the items currently visible on the screen (plus a small buffer).
*   **Windowing:** As you scroll down, items leaving the top are unmounted (memory freed), and new items at the bottom are mounted.
*   **Recycling (FlashList):** The Shopify team created `FlashList`, which is faster than `FlatList`. Instead of destroying and recreating views (which is expensive), it *recycles* existing views, changing the text/image content. This is how native Android `RecyclerView` and iOS `UICollectionView` work.

## Optimization Keys
1.  **getItemLayout:** If all your items are the same height (e.g., 50px), tell the list. This lets it calculate the scroll bar position without rendering the items.
2.  **keyExtractor:** Unique keys prevent unnecessary re-renders.
3.  **initialNumToRender:** Don't render 20 items if only 5 fit on the screen.
4.  **memo:** Wrap list items in `React.memo` to prevent re-rendering unchanged rows when the parent updates.
