# AppIntents Updates

## Overview

AppIntents is a framework that enables apps to extend functionality across the system, allowing users to perform app actions from anywhere, even when not in the app. Recent updates have expanded the capabilities and improved the developer experience for implementing AppIntents.

Key areas of improvement include:
- New system integrations with Apple Intelligence and visual intelligence
- User experience refinements with intent modes and foreground/background execution
- Convenience APIs with new property macros and Swift Package support
- Enhanced interactive snippets
- Improved Spotlight integration

## New System Integrations

### Visual Intelligence Integration

AppIntents now supports integration with visual intelligence, allowing users to circle objects in the visual intelligence camera or onscreen and view matching results from your app.

```swift
@UnionValue
enum VisualSearchResult {
    case landmark(LandmarkEntity)
    case collection(CollectionEntity)
}

struct LandmarkIntentValueQuery: IntentValueQuery {
    func values(for input: SemanticContentDescriptor) async throws -> [VisualSearchResult] {
        // Implementation to match visual input to app entities
    }
}

// Implement OpenIntent for each entity type
struct OpenLandmarkIntent: OpenIntent { /* ... */ }
struct OpenCollectionIntent: OpenIntent { /* ... */ }
```

### Onscreen Entities

Associate app entities with onscreen content using NSUserActivities, enabling users to ask Siri or ChatGPT about things currently visible in your app.

```swift
struct LandmarkDetailView: View {
    let landmark: LandmarkEntity

    var body: some View {
        Group { /* View content */ }
        .userActivity("com.landmarks.ViewingLandmark") { activity in
            activity.title = "Viewing \(landmark.name)"
            activity.appEntityIdentifier = EntityIdentifier(for: landmark)
        }
    }
}
```

## User Experience Refinements

### Intent Modes

AppIntents now supports more granular control over how intents execute with the new `supportedModes` property:

```swift
struct GetCrowdStatusIntent: AppIntent {
    static let supportedModes: IntentModes = [.background, .foreground(.dynamic)]

    func perform() async throws -> some ReturnsValue<Int> & ProvidesDialog {
        // Check if the landmark is open
        guard await modelData.isOpen(landmark) else { 
            // Return early if closed
            return .result(value: 0, dialog: "The landmark is currently closed.")
        }

        // Continue in foreground if possible
        if systemContext.currentMode.canContinueInForeground {
            do {
                try await continueInForeground(alwaysConfirm: false)
                await navigator.navigateToCrowdStatus(landmark)
            } catch {
                // Handle case where opening app was denied
            }
        }

        // Retrieve status and return dialog
        let status = await modelData.getCrowdStatus(landmark)
        return .result(value: status, dialog: "Current crowd level: \(status)")
    }
}
```

Available modes include:
- `.background` - Intent performs entirely in the background
- `.foreground(.immediate)` - App is foregrounded immediately before `perform()` runs
- `.foreground(.dynamic)` - App can be foregrounded during execution based on runtime conditions
- `.foreground(.deferred)` - App performs in background initially but will be foregrounded before completion

You can also combine modes:
- `[.background, .foreground]` - Foreground by default, background as fallback
- `[.background, .foreground(.dynamic)]` - Background by default, can request foreground
- `[.background, .foreground(.deferred)]` - Background initially, guaranteed foreground when requested

### Continuing in Foreground

New APIs to request continuation in the foreground:

```swift
// Request to continue in foreground
try await continueInForeground(alwaysConfirm: false)

// Request to continue in foreground after an error
throw needsToContinueInForegroundError(
    IntentDialog("Need to open app to complete this action"),
    alwaysConfirm: true
)
```

### Multiple Choice API

Request user input with the new choice API:

```swift
let options = [
    IntentChoiceOption(title: "Option 1", subtitle: "Description 1"),
    IntentChoiceOption(title: "Option 2", subtitle: "Description 2"),
    IntentChoiceOption.cancel(title: "Not now")
]

let choice = try await requestChoice(
    between: options,
    dialog: IntentDialog("Please select an option")
)

// Handle the user's choice
switch choice.id {
case options[0].id: // Option 1 selected
case options[1].id: // Option 2 selected
default: // Cancelled
}
```

## Convenience APIs

### New Property Macros

#### ComputedProperty

Use the `@ComputedProperty` macro to create computed properties for AppEntities that directly access the source of truth:

```swift
struct SettingsEntity: UniqueAppEntity {
    @ComputedProperty
    var defaultPlace: PlaceDescriptor {
        UserDefaults.standard.defaultPlace
    }

    init() { }
}
```

#### DeferredProperty

Use the `@DeferredProperty` macro for properties that are expensive to calculate and should only be fetched when explicitly requested:

```swift
struct LandmarkEntity: IndexedEntity {
    @DeferredProperty
    var crowdStatus: Int {
        get async throws {
            await modelData.getCrowdStatus(self)
        }
    }
}
```

### Swift Package Support

AppIntents can now be included in Swift Packages and static libraries:

```swift
// Framework or dynamic library
public struct LandmarksKitPackage: AppIntentsPackage { }

// App target
struct LandmarksPackage: AppIntentsPackage {
    static var includedPackages: [any AppIntentsPackage.Type] {
        [LandmarksKitPackage.self]
    }
}
```

## Interactive Snippets

### Static Snippets

Return a static snippet to show the outcome of an app intent:

```swift
func perform() async throws -> some IntentResult {
    // Perform the intent's action
    
    return .result(view: Text("Some example text.").font(.title))
}
```

### Interactive Snippets

Return an interactive snippet that allows users to perform follow-up actions:

```swift
func perform() async throws -> some IntentResult {
    // Find information about a nearby landmark
    let landmark = await findNearestLandmark()
    
    // Return an interactive snippet with buttons for follow-up actions
    return .result(
        value: landmark,
        opensIntent: OpenLandmarkIntent(landmark: landmark),
        snippetIntent: LandmarkSnippetIntent(landmark: landmark)
    )
}

// Define the snippet intent
struct LandmarkSnippetIntent: SnippetIntent {
    @Parameter var landmark: LandmarkEntity
    
    var snippet: some View {
        VStack {
            Text(landmark.name).font(.headline)
            Text(landmark.description).font(.body)
            
            HStack {
                Button("Add to Favorites") {
                    // Add to favorites action
                }
                
                Button("Search Tickets") {
                    // Search tickets action
                }
            }
        }
        .padding()
    }
}
```

## Spotlight Integration

### Making App Entities Available in Spotlight

1. Create an intent that displays your entity in your app:

```swift
struct OpenLandmarkIntent: OpenIntent {
    static let title: LocalizedStringResource = "Open Landmark"

    @Parameter(title: "Landmark", requestValueDialog: "Which landmark?")
    var target: LandmarkEntity

    func perform() async throws -> some IntentResult {
        return .result()
    }
}
```

2. Make your app entity indexable:

```swift
struct LandmarkEntity: AppEntity, IndexedEntity {
    static var typeDisplayRepresentation = TypeDisplayRepresentation(
        name: "Landmark",
        systemImage: "mountain.2"
    )
    
    var id: String
    var name: String
    var description: String
    var coordinate: CLLocationCoordinate2D
    var activities: [String]
    var regionDescription: String
    
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(
            title: "\(name)",
            subtitle: "\(regionDescription)",
            image: .init(systemName: "mountain.2")
        )
    }
}
```

3. Implement the searchable attribute set:

```swift
extension LandmarkEntity {
    var searchableAttributes: CSSearchableItemAttributeSet {
        let attributes = CSSearchableItemAttributeSet()
        
        attributes.title = name
        attributes.namedLocation = regionDescription
        attributes.keywords = activities
        
        attributes.latitude = NSNumber(value: coordinate.latitude)
        attributes.longitude = NSNumber(value: coordinate.longitude)
        attributes.supportsNavigation = true
        
        return attributes
    }
}
```

4. Add entities to the Spotlight index:

```swift
func indexLandmarks() async {
    let landmarks = await fetchLandmarks()
    
    do {
        try await CSSearchableIndex.default().indexAppEntities(
            landmarks,
            priority: .normal
        )
    } catch {
        print("Failed to index landmarks: \(error)")
    }
}
```

5. Update the index when data changes:

```swift
func deleteLandmark(_ landmark: LandmarkEntity) async {
    // Delete from data store
    await dataStore.delete(landmark)
    
    // Remove from Spotlight index
    do {
        try await CSSearchableIndex.default().deleteAppEntities(
            identifiedBy: [landmark.id],
            ofType: LandmarkEntity.self
        )
    } catch {
        print("Failed to remove landmark from index: \(error)")
    }
}
```

## Code Examples

### Basic App Intent

```swift
struct FindNearestLandmarkIntent: AppIntent {
    static var title: LocalizedStringResource = "Find Nearest Landmark"
    
    @Parameter(title: "Category")
    var category: String?
    
    func perform() async throws -> some IntentResult {
        let landmark = await findNearestLandmark(category: category)
        return .result(value: landmark)
    }
}
```

### App Shortcut

```swift
struct AppShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: FindNearestLandmarkIntent(),
            phrases: ["Find the closest landmark with \(.applicationName)"],
            systemImageName: "location"
        )
    }
}
```

### Entity with Indexable Properties

```swift
struct LandmarkEntity: AppEntity, IndexedEntity {
    static var typeDisplayRepresentation = TypeDisplayRepresentation(
        name: "Landmark",
        systemImage: "mountain.2"
    )
    
    var id: String
    
    @Property(title: "Name")
    var name: String
    
    @Property(title: "Description")
    var description: String
    
    @Property(title: "Location", indexingKey: \CSSearchableItemAttributeSet.namedLocation)
    var regionDescription: String
    
    @ComputedProperty(title: "Is Favorite")
    var isFavorite: Bool {
        UserDefaults.standard.favorites.contains(id)
    }
    
    @DeferredProperty(title: "Current Weather")
    var weather: String {
        get async throws {
            try await WeatherService.getWeather(for: coordinate)
        }
    }
    
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(
            title: "\(name)",
            subtitle: "\(regionDescription)",
            image: .init(systemName: "mountain.2")
        )
    }
}
```

## References

- [App Intents updates](https://developer.apple.com/documentation/Updates/AppIntents)
- [Adopting App Intents to support system experiences](https://developer.apple.com/documentation/AppIntents/adopting-app-intents-to-support-system-experiences)
- [Making app entities available in Spotlight](https://developer.apple.com/documentation/AppIntents/making-app-entities-available-in-spotlight)
- [Displaying static and interactive snippets](https://developer.apple.com/documentation/AppIntents/displaying-static-and-interactive-snippets)
- [WWDC 2025: Explore new advances in App Intents](https://developer.apple.com/videos/play/wwdc2025/275)
- [WWDC 2025: Get to know App Intents](https://developer.apple.com/videos/play/wwdc2025/244)