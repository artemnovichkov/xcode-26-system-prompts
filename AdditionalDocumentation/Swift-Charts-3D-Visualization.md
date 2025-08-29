# Using 3D Charts with Swift Charts

## Overview

Swift Charts provides powerful 3D visualization capabilities through the `Chart3D` component, allowing developers to create immersive three-dimensional data visualizations. This guide covers how to create, customize, and interact with 3D charts in SwiftUI applications using the Swift Charts framework.

Key components for 3D charts include:
- `Chart3D`: The main container view for 3D chart content
- `SurfacePlot`: For visualizing 3D surface data
- `Chart3DPose`: For controlling the viewing angle and perspective
- `Chart3DSurfaceStyle`: For styling the appearance of 3D surfaces

## Basic Setup

### Importing Required Frameworks

```swift
import SwiftUI
import Charts
```

### Creating a Simple 3D Chart

The most basic 3D chart can be created using a mathematical function that maps x,y coordinates to z values:

```swift
struct Basic3DChartView: View {
    var body: some View {
        Chart3D {
            SurfacePlot(
                x: "X Axis",
                y: "Y Axis",
                z: "Z Axis",
                function: { x, y in
                    // Simple mathematical function: z = sin(x) * cos(y)
                    sin(x) * cos(y)
                }
            )
        }
    }
}
```

### Creating a 3D Chart from Data

You can also create 3D charts from collections of data:

```swift
struct DataPoint3D: Identifiable {
    var x: Double
    var y: Double
    var z: Double
    var id = UUID()
}

struct Data3DChartView: View {
    let dataPoints: [DataPoint3D] = [
        // Your 3D data points
    ]
    
    var body: some View {
        Chart3D(dataPoints) { point in
            // Create appropriate 3D visualization for each point
        }
    }
}
```

## Customizing 3D Charts

### Setting the Chart Pose (Viewing Angle)

Control the viewing angle of your 3D chart using `Chart3DPose`:

```swift
struct CustomPose3DChartView: View {
    // Create a state variable to store the pose
    @State private var chartPose: Chart3DPose = .default
    
    var body: some View {
        Chart3D {
            SurfacePlot(
                x: "X Axis",
                y: "Y Axis",
                z: "Z Axis",
                function: { x, y in
                    sin(x) * cos(y)
                }
            )
        }
        // Apply the pose to the chart
        .chart3DPose(chartPose)
    }
}
```

You can use predefined poses:
- `.default`: The default viewing angle
- `.front`: View from the front
- `.back`: View from the back
- `.top`: View from the top
- `.bottom`: View from the bottom
- `.right`: View from the right side
- `.left`: View from the left side

Or create a custom pose with specific azimuth and inclination angles:

```swift
Chart3DPose(azimuth: .degrees(45), inclination: .degrees(30))
```

### Interactive Pose Control

Allow users to interact with the chart by binding the pose to a state variable:

```swift
struct Interactive3DChartView: View {
    @State private var chartPose: Chart3DPose = .default
    
    var body: some View {
        Chart3D {
            SurfacePlot(
                x: "X Axis",
                y: "Y Axis",
                z: "Z Axis",
                function: { x, y in
                    sin(x) * cos(y)
                }
            )
        }
        // Bind the pose to enable interactive rotation
        .chart3DPose($chartPose)
    }
}
```

### Setting the Camera Projection

Control the camera projection of the points in a 3D chart using `Chart3DCameraProjection`:

```swift
struct CustomProjection3DChartView: View {
    // Create a state variable to store the pose
    @State private var cameraProjection: Chart3DCameraProjection = .perspective
    
    var body: some View {
        Chart3D {
            SurfacePlot(
                x: "X Axis",
                y: "Y Axis",
                z: "Z Axis",
                function: { x, y in
                    sin(x) * cos(y)
                }
            )
        }
        // Apply the camera projection to the chart
        .chart3DCameraProjection(cameraProjection)
    }
}
```

You can use the following camera projection styles:
- `.automatic`: Automatically determines the camera projection
- `.orthographic`: Objects maintain size regardless of depth
- `.perspective`: Objects appear smaller with distance

## Working with Surface Plots

### Basic Surface Plot

```swift
SurfacePlot(
    x: "X Axis",
    y: "Y Axis",
    z: "Z Axis",
    function: { x, y in
        // Mathematical function defining the surface
        sin(sqrt(x*x + y*y))
    }
)
```

### Styling Surface Plots

Apply different styles to your surface plots:

```swift
SurfacePlot(
    x: "X Axis",
    y: "Y Axis",
    z: "Z Axis",
    function: { x, y in
        sin(x) * cos(y)
    }
)
.foregroundStyle(Color.blue)
```

Available surface styles:
- `.heightBased`: Colors the surface based on the height (y-value)
- `.normalBased`: Colors the surface based on the surface normal direction

### Custom Gradient Surface Style

Create a custom gradient for your surface:

```swift
let customGradient = Gradient(colors: [.blue, .purple, .red])

SurfacePlot(
    x: "X Axis",
    y: "Y Axis",
    z: "Z Axis",
    function: { x, y in
        sin(x) * cos(y)
    }
)
.foregroundStyle(LinearGradient(gradient: customGradient, startPoint: .topLeading, endPoint: .bottomTrailing))
```

### Controlling Surface Roughness

Adjust the roughness of the surface:

```swift
SurfacePlot(
    x: "X Axis",
    y: "Y Axis",
    z: "Z Axis",
    function: { x, y in
        sin(x) * cos(y)
    }
)
.roughness(0.3) // 0 is smooth, 1 is completely rough
```

## Advanced Techniques

### Combining Multiple Surface Plots

```swift
Chart3D {
    // First surface plot
    SurfacePlot(
        x: "X",
        y: "Y",
        z: "Z",
        function: { x, y in
            sin(x) * cos(y)
        }
    )
    
    // Second surface plot
    SurfacePlot(
        x: "X",
        y: "Y",
        z: "Z",
        function: { x, y in
            cos(x) * sin(y) + 2 // Offset to avoid overlap
        }
    )
}
```

### Specifying Y-Range for Height-Based Styling

Control the color mapping by specifying the y-range:

```swift
SurfacePlot(
    x: "X Axis",
    y: "Y Axis",
    z: "Z Axis",
    function: { x, y in
        sin(x) * cos(y)
    }
)
.foregroundStyle(Chart3DSurfaceStyle.heightBased(yRange: -1.0...1.0))
```

### Custom Gradient with Y-Range

```swift
let customGradient = Gradient(colors: [.blue, .green, .yellow, .red])

SurfacePlot(
    x: "X Axis",
    y: "Y Axis",
    z: "Z Axis",
    function: { x, y in
        sin(x) * cos(y)
    }
)
.foregroundStyle(Chart3DSurfaceStyle.heightBased(customGradient, yRange: -1.0...1.0))
```

## Complete Example: Interactive 3D Visualization

Here's a complete example that demonstrates an interactive 3D chart with customized styling:

```swift
import SwiftUI
import Charts

struct Interactive3DSurfaceView: View {
    // State for interactive rotation
    @State private var chartPose: Chart3DPose = .default
    
    // Custom gradient for surface coloring
    let surfaceGradient = Gradient(colors: [
        .blue,
        .cyan,
        .green,
        .yellow,
        .orange,
        .red
    ])
    
    var body: some View {
        VStack {
            Text("Interactive 3D Surface Visualization")
                .font(.headline)
            
            Chart3D {
                SurfacePlot(
                    x: "X Value",
                    y: "Y Value",
                    z: "Result",
                    function: { x, y in
                        // Interesting mathematical function
                        sin(sqrt(x*x + y*y)) / sqrt(x*x + y*y + 0.1)
                    }
                )
                .roughness(0.2)
            }
            .chart3DPose($chartPose)
            .frame(height: 400)
            
            Text("Drag to rotate the visualization")
                .font(.caption)
                .foregroundColor(.secondary)
            
            HStack {
                Button("Front View") {
                    withAnimation {
                        chartPose = .front
                    }
                }
                
                Button("Top View") {
                    withAnimation {
                        chartPose = .top
                    }
                }
                
                Button("Default View") {
                    withAnimation {
                        chartPose = .default
                    }
                }
            }
            .buttonStyle(.bordered)
        }
        .padding()
    }
}
```

## References

- [Apple Developer Documentation: Chart3D](https://developer.apple.com/documentation/Charts/Chart3D)
- [Apple Developer Documentation: SurfacePlot](https://developer.apple.com/documentation/Charts/SurfacePlot)
- [Apple Developer Documentation: Chart3DPose](https://developer.apple.com/documentation/Charts/Chart3DPose)
- [Apple Developer Documentation: Chart3DSurfaceStyle](https://developer.apple.com/documentation/Charts/Chart3DSurfaceStyle)
- [Apple Developer Documentation: Creating a chart using Swift Charts](https://developer.apple.com/documentation/Charts/Creating-a-chart-using-Swift-Charts)