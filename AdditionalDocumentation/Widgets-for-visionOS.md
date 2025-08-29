# Widgets for visionOS

## Overview

Widgets in visionOS provide a powerful way to display glanceable information from your app in a spatial computing environment. Unlike traditional 2D widgets on other platforms, visionOS widgets are three-dimensional objects that can be placed in a user's physical space, either mounted on surfaces (walls, tables) or floating in the environment. They support unique features like proximity awareness, different mounting styles, and specialized textures that help them blend naturally into the spatial environment.

Key concepts for visionOS widgets include mounting styles (elevated or recessed), textures (glass or paper), proximity awareness, and support for various widget families including extra-large sizes. This guide covers the essential APIs and implementation details for creating effective widgets in visionOS.

## Widget Mounting Styles

In visionOS, widgets can be mounted in two different styles:

- **Elevated**: Widgets sit on top of horizontal or vertical surfaces (default style)
- **Recessed**: Widgets appear embedded into vertical surfaces like walls

```swift
struct WeatherWidget: Widget {
    var body: some WidgetConfiguration {
        StaticConfiguration(
            // Configuration details
        ) { entry in
            WeatherWidgetView(entry: entry)
        }
        .configurationDisplayName("Weather Widget")
        // Specify supported mounting styles
        .supportedMountingStyles([.elevated, .recessed]) // Default is both
        // Or limit to just one style
        // .supportedMountingStyles([.recessed])
    }
}
```

If your widget only supports the recessed mounting style, users won't be able to place it on horizontal surfaces.

## Widget Textures

visionOS offers two texture options for widgets:

- **Glass**: The default texture that gives widgets a transparent glass-like appearance
- **Paper**: An alternative texture that provides a poster-like look

```swift
struct CaffeineTrackerWidget: Widget {
    var body: some WidgetConfiguration {
        StaticConfiguration(
            // Configuration details
        ) { entry in
            CaffeineTrackerWidgetView(entry: entry)
        }
        .configurationDisplayName("Caffeine Tracker")
        // Specify the widget texture
        .widgetTexture(.glass) // Default
        // Or use paper texture
        // .widgetTexture(.paper)
    }
}
```

## Proximity Awareness

A key feature of widgets in visionOS is their ability to respond to a user's proximity. Widgets can display different levels of detail based on how close or far away the user is viewing them from.

```swift
struct TotalCaffeineView: View {
    // Access the level of detail environment variable
    @Environment(\.levelOfDetail) var levelOfDetail
    
    // Other properties
    
    var body: some View {
        VStack {
            Text("Total Caffeine")
                .font(.caption)
            Text(totalCaffeine.formatted())
                .font(caffeineFont)
        }
    }
    
    // Adjust font size based on proximity
    var caffeineFont: Font {
        if levelOfDetail == .simplified {
            return .largeTitle // Larger text when viewed from a distance
        } else {
            return .title // Normal size when viewed up close
        }
    }
}
```

The `levelOfDetail` environment variable can have two values:
- `.default`: Used when the user is close to the widget
- `.simplified`: Used when the user is viewing from a distance

When a user's distance to a widget changes, the system automatically animates the layout changes between these two states.

## Supporting Widget Families

visionOS supports all system family widget sizes, from small to extra large:

```swift
struct MyVisionWidget: Widget {
    var body: some WidgetConfiguration {
        StaticConfiguration(
            // Configuration details
        ) { entry in
            MyWidgetView(entry: entry)
        }
        .configurationDisplayName("My Widget")
        .supportedFamilies([
            .systemSmall,
            .systemMedium,
            .systemLarge,
            .systemExtraLarge,
            .systemExtraLargePortrait // visionOS-specific
        ])
    }
}
```

The extra large widget families (`.systemExtraLarge` and `.systemExtraLargePortrait`) are particularly effective when using the paper texture for a poster-like appearance.

## Rendering Modes

Widgets in visionOS support both full color and accented rendering modes:

- **Full Color**: The default mode that displays the widget with its complete design
- **Accented**: A simplified mode where the background is removed and replaced with a solid color that complements the user's selected color theme

```swift
// No special code is needed to support accented mode
// Just ensure your widget looks good with or without its background

// Use containerBackground to mark removable backgrounds
var body: some View {
    VStack {
        // Widget content
    }
    .containerBackground(for: .widget) {
        Color.gameBackground
    }
}
```

To detect whether a widget appears with or without a background, use the `showsWidgetContainerBackground` environment variable.

## Complete Widget Example

Here's a complete example of a widget configured for visionOS:

```swift
struct WeatherWidget: Widget {
    var body: some WidgetConfiguration {
        StaticConfiguration(
            kind: "com.example.weather",
            provider: WeatherProvider()
        ) { entry in
            WeatherWidgetView(entry: entry)
                .containerBackground(for: .widget) {
                    Color.skyBlue.opacity(0.8)
                }
        }
        .configurationDisplayName("Weather")
        .description("Current weather conditions")
        .supportedFamilies([.systemSmall, .systemMedium, .systemExtraLarge])
        .supportedMountingStyles([.elevated, .recessed])
        .widgetTexture(.glass)
    }
}

struct WeatherWidgetView: View {
    var entry: WeatherProvider.Entry
    @Environment(\.levelOfDetail) var levelOfDetail
    
    var body: some View {
        if levelOfDetail == .simplified {
            // Simplified view for distance viewing
            VStack {
                Image(systemName: entry.weatherIcon)
                    .font(.system(size: 40))
                Text("\(entry.temperature)°")
                    .font(.system(size: 36, weight: .bold))
            }
        } else {
            // Detailed view for close viewing
            VStack(alignment: .leading) {
                HStack {
                    Image(systemName: entry.weatherIcon)
                        .font(.title)
                    Spacer()
                    Text("\(entry.temperature)°")
                        .font(.title)
                }
                
                Spacer()
                
                Text(entry.location)
                    .font(.caption)
                Text(entry.condition)
                    .font(.caption2)
            }
            .padding()
        }
    }
}
```

## Widget Preview

Use the `Preview` macro to preview your widget in different configurations:

```swift
#Preview("Weather Widget", as: .systemSmall) {
    WeatherWidget()
} timelineProvider: {
    WeatherProvider()
}
```

## Background Removal

To ensure your widget appears correctly in different contexts, mark your background views as removable:

```swift
var body: some View {
    VStack {
        // Widget content
    }
    .containerBackground(for: .widget) {
        // This background will be automatically removed when needed
        Color.widgetBackground
    }
}
```

If you need to detect whether a widget appears with or without a background, use:

```swift
@Environment(\.showsWidgetContainerBackground) var showsBackground
```

## References

- [Updating your widgets for visionOS](https://developer.apple.com/documentation/WidgetKit/Updating-your-widgets-for-visionOS)
- [Support recessed and elevated mounting styles](https://developer.apple.com/documentation/WidgetKit/Updating-your-widgets-for-visionOS#Support-recessed-and-elevated-mounting-styles)
- [Support visionOS rendering styles and extra large widgets](https://developer.apple.com/documentation/WidgetKit/Updating-your-widgets-for-visionOS#Support-visionOS-rendering-styles-and-extra-large-widgets)
- [Add proximity awareness to your widget](https://developer.apple.com/documentation/WidgetKit/Updating-your-widgets-for-visionOS#Add-proximity-awareness-to-your-widget)
- [Developing a WidgetKit strategy](https://developer.apple.com/documentation/WidgetKit/Developing-a-WidgetKit-strategy)
- [Displaying the right widget background](https://developer.apple.com/documentation/WidgetKit/Displaying-the-right-widget-background)