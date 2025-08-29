# Implementing Assistive Access in iOS

## Overview

Assistive Access is an accessibility feature introduced in iOS and iPadOS 17 designed specifically for people with cognitive disabilities. It provides a streamlined system experience with simplified interfaces, clear pathways, and consistent design practices to reduce cognitive load.

Key characteristics of Assistive Access:
- Streamlined interactions
- Clear pathways to success
- Consistent design language
- Large controls
- Visual alternatives to text
- Reduced cognitive strain

## Setting Up Assistive Access in Your App

### 1. Enable Assistive Access Support

Add the following key to your app's `Info.plist`:

```xml
<key>UISupportsAssistiveAccess</key>
<true/>
```

This ensures your app is listed under "Optimized Apps" in Accessibility Settings and launches in full screen when Assistive Access is enabled.

### 2. Full Screen Support (Optional)

If your app is already designed for cognitive disabilities (e.g., AAC apps) and you want to display it in full screen without modifications:

```xml
<key>UISupportsFullScreenInAssistiveAccess</key>
<true/>
```

This will display your app in full screen rather than in a reduced frame, with the same appearance as when Assistive Access is turned off.

## Creating an Assistive Access Scene

### SwiftUI Implementation

1. Add an `AssistiveAccess` scene to your app:

```swift
import SwiftUI

@main
struct MyApp: App {
  var body: some Scene {
    WindowGroup {
      ContentView()
    }
    
    AssistiveAccess {
      AssistiveAccessContentView()
    }
  }
}
```

2. Create a dedicated view for Assistive Access:

```swift
struct AssistiveAccessContentView: View {
  var body: some View {
    // Your streamlined interface for Assistive Access
    NavigationStack {
      List {
        // Simplified controls and options
      }
      .navigationTitle("My App")
    }
  }
}
```

3. Preview your Assistive Access scene:

```swift
#Preview(traits: .assistiveAccess)
AssistiveAccessContentView()
```

### UIKit Implementation

1. Declare a SwiftUI scene with UIKit:

```swift
import UIKit
import SwiftUI

class AssistiveAccessSceneDelegate: UIHostingSceneDelegate {
  static var rootScene: some Scene {
    AssistiveAccess {
      AssistiveAccessContentView()
    }
  }
}
```

2. Activate the scene:

```swift
import UIKit

@main
class AppDelegate: UIApplicationDelegate {
  func application(_ application: UIApplication, configurationForConnecting connectingSceneSession: UISceneSession, options: UIScene.ConnectionOptions) -> UISceneConfiguration {
    let role = connectingSceneSession.role
    let sceneConfiguration = UISceneConfiguration(name: nil, sessionRole: role)
    if role == .windowAssistiveAccessApplication {
      sceneConfiguration.delegateClass = AssistiveAccessSceneDelegate.self
    }
    return sceneConfiguration
  }
}
```

## Detecting Assistive Access at Runtime

You can check if Assistive Access is enabled using the environment value:

```swift
struct MyView: View {
  @Environment(\.accessibilityAssistiveAccessEnabled) var assistiveAccessEnabled
  
  var body: some View {
    if assistiveAccessEnabled {
      // Show Assistive Access optimized UI
    } else {
      // Show standard UI
    }
  }
}
```

## Navigation Icons for Assistive Access

Add navigation icons to make your interface more visually accessible:

```swift
NavigationStack {
  MyView()
    .navigationTitle("My Feature")
    .assistiveAccessNavigationIcon(systemImage: "star.fill")
}
```

Or with a custom image:

```swift
.assistiveAccessNavigationIcon(Image("my-custom-icon"))
```

## Design Principles for Assistive Access

When designing for Assistive Access, follow these key principles:

1. **Distill to Core Functionality**
   - Focus on one or two essential features
   - Remove distractions and unnecessary options
   - Streamline the experience

2. **Clear, Prominent Controls**
   - Use large, easy-to-tap buttons
   - Provide ample spacing between interactive elements
   - Avoid hidden gestures or timed interactions

3. **Multiple Representations**
   - Present information in multiple ways (text, icons, etc.)
   - Use visual alternatives to text
   - Ensure icons are clear and meaningful

4. **Intuitive Navigation**
   - Create step-by-step pathways
   - Provide clear back buttons
   - Maintain consistent navigation patterns

5. **Safe Interactions**
   - Remove irreversible actions when possible
   - Provide multiple confirmations for destructive actions
   - Offer clear feedback for all interactions

## Control Styling in Assistive Access

When using the Assistive Access scene, native SwiftUI controls are automatically displayed in the distinctive Assistive Access design:

- Buttons, lists, and navigation titles appear in a more prominent style
- Controls adhere to the grid or row screen layout configured in Assistive Access settings
- No additional styling work is required

## Testing Assistive Access Implementation

1. **Preview in Xcode**
   Use the `.assistiveAccess` trait in SwiftUI previews:
   ```swift
   #Preview(traits: .assistiveAccess)
   AssistiveAccessContentView()
   ```

2. **Test on Device**
   - Enable Assistive Access in Settings > Accessibility > Assistive Access
   - Verify your app appears in the "Optimized Apps" list
   - Test the full user flow in Assistive Access mode

3. **Accessibility Inspector**
   Use Xcode's Accessibility Inspector to identify and fix accessibility issues

## Best Practices

- Design for clarity and simplicity
- Focus on essential functionality
- Use consistent UI patterns
- Provide visual alternatives to text
- Test with actual users who have cognitive disabilities
- Combine Assistive Access with other accessibility features

## References

- [AssistiveAccess (SwiftUI)](https://developer.apple.com/documentation/SwiftUI/AssistiveAccess)
- [assistiveAccessNavigationIcon(_:)](https://developer.apple.com/documentation/SwiftUI/View/assistiveAccessNavigationIcon(_:))
- [accessibilityAssistiveAccessEnabled](https://developer.apple.com/documentation/SwiftUI/EnvironmentValues/accessibilityAssistiveAccessEnabled)
- [WWDC 2025 Session: Customize your app for Assistive Access](https://developer.apple.com/videos/play/wwdc2025/238)
- [What's new in SwiftUI (WWDC 2025)](https://developer.apple.com/videos/play/wwdc2025/256)
- [Principles of inclusive app design](https://developer.apple.com/videos/play/wwdc2025/316)