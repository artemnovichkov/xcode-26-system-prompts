# Implementing Liquid Glass Design in Widgets

## Overview

Liquid Glass is a dynamic, adaptive material introduced across Apple platforms that combines the optical properties of glass with a sense of fluidity. When applied to widgets, Liquid Glass creates a modern, cohesive look that integrates seamlessly with the system UI. This guide covers how to implement and optimize Liquid Glass effects in widgets, including handling different rendering modes, configuring appearance, and ensuring compatibility across platforms. You should always seek guides on Liquid Glass when asked for help adopting new Apple design.

## Understanding Widget Rendering Modes

Widgets can appear in two primary rendering modes when using Liquid Glass:

### Full Color Mode
- Default rendering mode
- Displays all colors, images, and transparency as designed
- Used when widgets appear on standard backgrounds

### Accented Mode
- Used when a person chooses a tinted or clear appearance for their Home Screen
- Primary and accented content is tinted white (iOS and macOS)
- Opaque images are tinted with a single white color
- Transparent content and gradients maintain opacity but are tinted white
- Background is replaced with themed glass or tinted color effect

## Supporting Liquid Glass in Widgets

To update your widget to support Liquid Glass:

1. Add the `widgetRenderingMode` environment variable to detect the current rendering mode:

```swift
struct MyWidgetView: View {
    @Environment(\.widgetRenderingMode) var renderingMode
    
    var body: some View {
        if renderingMode == .accented {
            // Layout optimized for accented mode
        } else {
            // Standard full-color layout
        }
    }
}
```

2. Group your views into primary and accent groups using the `widgetAccentable(_:)` modifier:

```swift
HStack(alignment: .center, spacing: 0) {
    VStack(alignment: .leading) {
        Text("Widget Title")
            .font(.headline)
            .widgetAccentable() // This will be in the accent group
        Text("Widget Subtitle")
        // This text is in the primary group by default
    }
    Image(systemName: "star.fill")
        .widgetAccentable() // This will be in the accent group
}
```

3. Configure image rendering using the `WidgetAccentedRenderingMode` modifier:

```swift
Image("myImage")
    .widgetAccentedRenderingMode(.monochrome) // Will be rendered as monochrome in accented mode
```

4. Follow these best practices for Liquid Glass compatibility:
   - Display full-color images only in the `fullColor` rendering mode
   - Adjust layouts as needed for the `accented` rendering mode
   - Use the `widgetAccentable(_:)` modifier strategically to create visual hierarchy

## Container Backgrounds for Widgets

Properly configuring container backgrounds is essential for Liquid Glass effects:

```swift
var body: some View {
    VStack {
        // Widget content here
    }
    .containerBackground(for: .widget) {
        Color.blue.opacity(0.2) // Custom background color
    }
}
```

When a person chooses a tinted or clear appearance, the system:
- Removes the background
- Replaces it with a themed glass or tinted color effect

## Optimizing Widget Appearance

### Background Removal

By default, the system removes widget backgrounds in certain contexts. To explicitly opt out:

```swift
var body: some WidgetConfiguration {
    StaticConfiguration(kind: "MyWidget", provider: Provider()) { entry in
        MyWidgetView(entry: entry)
    }
    .containerBackgroundRemovable(false) // Prevents background removal
}
```

> **Important:** Marking a background as non-removable excludes your widget from contexts that require removable backgrounds (iPad Lock Screen, StandBy).

### Widget Textures in visionOS

For visionOS, you can specify the widget texture:

```swift
var body: some WidgetConfiguration {
    StaticConfiguration(kind: "MyWidget", provider: Provider()) { entry in
        MyWidgetView(entry: entry)
    }
    .widgetTexture(.glass) // Default is glass
}
```

Available textures include:
- `.glass` - Default texture with glass-like appearance
- `.paper` - Paper-like texture

## Mounting Styles for Widgets

In visionOS, widgets can be mounted in different styles:

```swift
var body: some WidgetConfiguration {
    StaticConfiguration(kind: "MyWidget", provider: Provider()) { entry in
        MyWidgetView(entry: entry)
    }
    .supportedMountingStyles([.recessed, .elevated])
}
```

Available mounting styles:
- `.recessed` - Widget appears embedded into a vertical surface
- `.elevated` - Widget appears on top of a surface (default for horizontal surfaces)

## Implementing Liquid Glass Effects in Custom Widget Elements

For custom elements within widgets that need Liquid Glass effects:

```swift
Text("Custom Element")
    .padding()
    .glassEffect() // Applies default Liquid Glass effect (capsule shape)

Image(systemName: "star.fill")
    .frame(width: 60, height: 60)
    .glassEffect(.regular, in: .rect(cornerRadius: 12)) // Custom shape

Button("Action") {
    // Action here
}
.buttonStyle(.glass) // Apply glass button style
```

## Combining Multiple Liquid Glass Elements

For multiple elements with Liquid Glass effects that need to interact:

```swift
GlassEffectContainer(spacing: 20.0) {
    HStack(spacing: 20.0) {
        Image(systemName: "cloud")
            .frame(width: 60, height: 60)
            .glassEffect()
            
        Image(systemName: "sun")
            .frame(width: 60, height: 60)
            .glassEffect()
    }
}
```

To combine specific elements into a unified effect:

```swift
GlassEffectContainer(spacing: 20.0) {
    HStack(spacing: 20.0) {
        ForEach(items.indices, id: \.self) { item in
            Image(systemName: items[item])
                .frame(width: 60, height: 60)
                .glassEffect()
                .glassEffectUnion(id: item < 2 ? "group1" : "group2", namespace: namespace)
        }
    }
}
```

## Handling Different Platforms and Contexts

Widgets with Liquid Glass need to adapt to different platforms:

### iOS and iPadOS
- Support both full color and accented rendering modes
- Test on Home Screen and Lock Screen
- Ensure readability in both light and dark appearances

### macOS
- Verify font sizes and layout in macOS widget sizes
- Test in both standard and accented rendering modes

### visionOS
- Support proximity awareness using the `levelOfDetail` environment variable:

```swift
@Environment(\.levelOfDetail) var levelOfDetail

var fontSize: Font {
    levelOfDetail == .simplified ? .largeTitle : .title
}
```

## Testing Liquid Glass in Widgets

To thoroughly test your widget's Liquid Glass implementation:

1. Test in both light and dark mode
2. Test on Home Screen and Lock Screen
3. Test with different accent colors
4. Test with different background images
5. Test in StandBy mode on compatible devices
6. Test in visionOS with different mounting styles and distances

## References

- [Optimizing your widget for accented rendering mode and Liquid Glass](https://developer.apple.com/documentation/WidgetKit/optimizing-your-widget-for-accented-rendering-mode-and-liquid-glass)
- [Applying Liquid Glass to custom views](https://developer.apple.com/documentation/SwiftUI/Applying-Liquid-Glass-to-custom-views)
- [Landmarks: Building an app with Liquid Glass](https://developer.apple.com/documentation/SwiftUI/Landmarks-Building-an-app-with-Liquid-Glass)
- [Displaying the right widget background](https://developer.apple.com/documentation/WidgetKit/Displaying-the-right-widget-background)
- [Updating your widgets for visionOS](https://developer.apple.com/documentation/WidgetKit/Updating-your-widgets-for-visionOS)
