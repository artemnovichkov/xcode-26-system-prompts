# Implementing Visual Intelligence in iOS

## Overview

Visual Intelligence is a framework that enables iOS apps to integrate with the system's visual search capabilities. It allows users to find app content that matches their surroundings or objects onscreen by using the visual intelligence camera or screenshots. When a user performs a visual search, your app can provide relevant content that matches what they're looking at.

Key concepts:
- Visual Intelligence framework provides information about objects detected in the camera or screenshots
- App Intents framework facilitates the exchange of information between the system and your app
- Your app searches its content for matches and returns them as app entities
- Results appear directly in the visual search interface, allowing users to view and interact with your content

## Setting Up Visual Intelligence

### Required Frameworks

```swift
import VisualIntelligence
import AppIntents
```

### Implementation Steps

1. Create an `IntentValueQuery` to receive visual search requests
2. Implement the `values(for:)` method to process the `SemanticContentDescriptor`
3. Search your app's content using the provided information
4. Return matching content as app entities

## Working with SemanticContentDescriptor

The `SemanticContentDescriptor` is the core object that provides information about what the user is looking at.

### Key Properties

```swift
// A list of labels that Visual Intelligence uses to classify items
let labels: [String]

// The pixel buffer containing the visual data
var pixelBuffer: CVReadOnlyPixelBuffer?
```

### Accessing Visual Data

You can use either the labels or the pixel buffer (or both) to search for matching content:

```swift
// Using labels
func searchByLabels(_ labels: [String]) -> [AppEntity] {
    // Search your app's content using the provided labels
    return matchingEntities
}

// Using pixel buffer
func searchByImage(_ pixelBuffer: CVReadOnlyPixelBuffer) -> [AppEntity] {
    // Convert pixel buffer to an image and search your content
    return matchingEntities
}
```

## Creating an IntentValueQuery

The `IntentValueQuery` protocol is the entry point for Visual Intelligence to communicate with your app.

### Basic Implementation

```swift
struct LandmarkIntentValueQuery: IntentValueQuery {
    @Dependency var modelData: ModelData
    
    func values(for input: SemanticContentDescriptor) async throws -> [VisualSearchResult] {
        // Check if pixel buffer is available
        guard let pixelBuffer = input.pixelBuffer else {
            return []
        }
        
        // Search for matching landmarks using the pixel buffer
        let landmarks = try await modelData.search(matching: pixelBuffer)
        
        return landmarks
    }
}
```

### Using Union Values for Different Result Types

If your app needs to return different types of results, use a union value:

```swift
@UnionValue
enum VisualSearchResult {
    case landmark(LandmarkEntity)
    case collection(CollectionEntity)
}
```

## Providing Display Representations

Visual Intelligence uses the `DisplayRepresentation` of your `AppEntity` to present your content in the search results.

### Creating a Display Representation

```swift
struct LandmarkEntity: AppEntity {
    static var typeDisplayRepresentation: TypeDisplayRepresentation {
        return TypeDisplayRepresentation(
            name: LocalizedStringResource("Landmark", table: "AppIntents"),
            numericFormat: "\(placeholder: .int) landmarks"
        )
    }
    
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(
            title: "\(name)",
            subtitle: "\(location)",
            image: .init(named: landmark.thumbnailImageName)
        )
    }
    
    // Other required AppEntity properties and methods
}
```

## Opening Items in Your App

When a user taps on a search result, your app should open to display detailed information about that item.

### Implementing AppEntity for Deep Linking

```swift
struct LandmarkEntity: AppEntity {
    var id: String
    var name: String
    var location: String
    var thumbnailImageName: String
    
    // Required for deep linking
    static var typeDisplayRepresentation: TypeDisplayRepresentation {
        // As shown above
    }
    
    var displayRepresentation: DisplayRepresentation {
        // As shown above
    }
    
    // Define how to open this entity in your app
    var appLinkURL: URL? {
        URL(string: "yourapp://landmark/\(id)")
    }
}
```

## Linking to Additional Results

If your app finds many matches, you can provide a "More results" button that opens your app to show the full list.

### Creating a Semantic Content Search Intent

```swift
struct ViewMoreLandmarksIntent: AppIntent, VisualIntelligenceSearchIntent {
    static var title: LocalizedStringResource = "View More Landmarks"
    
    @Parameter(title: "Semantic Content")
    var semanticContent: SemanticContentDescriptor
    
    func perform() async throws -> some IntentResult {
        // Open your app's search view with the semantic content
        return .result()
    }
}
```

## Complete Example

Here's a complete example of implementing Visual Intelligence in a landmarks app:

```swift
import SwiftUI
import AppIntents
import VisualIntelligence

// Define the search result types
@UnionValue
enum VisualSearchResult {
    case landmark(LandmarkEntity)
    case collection(CollectionEntity)
}

// Define the landmark entity
struct LandmarkEntity: AppEntity {
    var id: String
    var name: String
    var location: String
    var thumbnailImageName: String
    
    static var typeDisplayRepresentation: TypeDisplayRepresentation {
        return TypeDisplayRepresentation(
            name: LocalizedStringResource("Landmark", table: "AppIntents"),
            numericFormat: "\(placeholder: .int) landmarks"
        )
    }
    
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(
            title: "\(name)",
            subtitle: "\(location)",
            image: .init(named: thumbnailImageName)
        )
    }
    
    var appLinkURL: URL? {
        URL(string: "yourapp://landmark/\(id)")
    }
}

// Define the collection entity
struct CollectionEntity: AppEntity {
    var id: String
    var name: String
    var landmarks: [LandmarkEntity]
    
    static var typeDisplayRepresentation: TypeDisplayRepresentation {
        return TypeDisplayRepresentation(
            name: LocalizedStringResource("Collection", table: "AppIntents"),
            numericFormat: "\(placeholder: .int) collections"
        )
    }
    
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(
            title: "\(name)",
            subtitle: "\(landmarks.count) landmarks",
            image: .init(systemName: "square.stack.fill")
        )
    }
    
    var appLinkURL: URL? {
        URL(string: "yourapp://collection/\(id)")
    }
}

// Define the intent value query
struct LandmarkIntentValueQuery: IntentValueQuery {
    @Dependency var modelData: ModelData
    
    func values(for input: SemanticContentDescriptor) async throws -> [VisualSearchResult] {
        // Try to use labels first
        if !input.labels.isEmpty {
            let landmarks = try await modelData.search(matching: input.labels)
            return landmarks
        }
        
        // Fall back to pixel buffer if available
        guard let pixelBuffer = input.pixelBuffer else {
            return []
        }
        
        let landmarks = try await modelData.search(matching: pixelBuffer)
        return landmarks
    }
}

// Define the "more results" intent
struct ViewMoreLandmarksIntent: AppIntent, VisualIntelligenceSearchIntent {
    static var title: LocalizedStringResource = "View More Landmarks"
    
    @Parameter(title: "Semantic Content")
    var semanticContent: SemanticContentDescriptor
    
    func perform() async throws -> some IntentResult {
        // Open your app's search view with the semantic content
        return .result()
    }
}

// Example model data service
class ModelData {
    func search(matching labels: [String]) async throws -> [VisualSearchResult] {
        // Search your database for landmarks matching the labels
        // Return matching landmarks as VisualSearchResult objects
        return []
    }
    
    func search(matching pixelBuffer: CVReadOnlyPixelBuffer) async throws -> [VisualSearchResult] {
        // Use image recognition to find landmarks in the pixel buffer
        // Return matching landmarks as VisualSearchResult objects
        return []
    }
}
```

## Best Practices

1. **Performance**: Return results quickly for a good search experience
   - Limit the number of returned items (consider showing 10-20 most relevant results)
   - Use the "More results" button for additional items
   - Optimize your search algorithms for speed

2. **Quality**: Provide high-quality display representations
   - Use clear, concise titles and subtitles
   - Include relevant images that help identify the content
   - Ensure all text is properly localized

3. **Relevance**: Focus on returning the most relevant results
   - Prioritize exact matches over partial matches
   - Consider the context of the search (location, time, etc.)
   - Filter out irrelevant or low-confidence matches

4. **User Experience**: Make it easy to navigate from search results to your app
   - Implement deep linking to open specific content
   - Maintain context when transitioning to your app
   - Provide a consistent experience between search results and your app

## Testing

To test your Visual Intelligence integration:
1. Build and run your app on a device
2. Use the visual intelligence camera or take a screenshot
3. Perform a visual search on content relevant to your app
4. Verify that your app's results appear in the search results
5. Test tapping on results to ensure they open correctly in your app

## References

- [Integrating your app with visual intelligence](https://developer.apple.com/documentation/VisualIntelligence/integrating-your-app-with-visual-intelligence)
- [SemanticContentDescriptor](https://developer.apple.com/documentation/VisualIntelligence/SemanticContentDescriptor)
- [IntentValueQuery](https://developer.apple.com/documentation/AppIntents/IntentValueQuery)
- [DisplayRepresentation](https://developer.apple.com/documentation/AppIntents/DisplayRepresentation)
- [TypeDisplayRepresentation](https://developer.apple.com/documentation/appintents/TypeDisplayRepresentation)
- [App Intents framework](https://developer.apple.com/documentation/AppIntents)
- [Making actions and content discoverable and widely available](https://developer.apple.com/documentation/AppIntents/Making-actions-and-content-discoverable-and-widely-available)