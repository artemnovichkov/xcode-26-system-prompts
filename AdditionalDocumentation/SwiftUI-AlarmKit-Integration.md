# Using AlarmKit in a SwiftUI App

## Overview

AlarmKit is a framework introduced in iOS 18 that allows developers to create custom alarms and timers within their apps. It provides a comprehensive system for managing alarms with customizable schedules and UI. AlarmKit supports one-time and repeating alarms, with options for countdown durations and snooze functionality. It handles alarm authorization and provides UI for both templated and widget presentations.

Key features:
- Schedule one-time or repeating alarms
- Create countdown timers
- Customize alarm UI and presentation
- Integrate with Live Activities for Dynamic Island and Lock Screen
- Override device focus and silent mode

## Key Components

### AlarmManager

The central class for interacting with the alarm system:

```swift
// Access the shared instance
let alarmManager = AlarmManager.shared
```

### Alarm

A structure that describes an alarm that can alert once or on a repeating schedule:

```swift
struct Alarm {
    var id: UUID
    var schedule: Schedule?
    var countdownDuration: CountdownDuration?
    // Other properties
}
```

### AlarmPresentation

Defines the content required for the alarm UI in different states:

```swift
struct AlarmPresentation {
    var alert: Alert
    var countdown: Countdown?
    var paused: Paused?
}
```

### AlarmAttributes

Contains all information necessary for the alarm UI:

```swift
struct AlarmAttributes<Metadata: AlarmMetadata> {
    var presentation: AlarmPresentation
    var metadata: Metadata
    var tintColor: Color
}
```

## Authorization

Before scheduling alarms, your app needs to request authorization:

```swift
// Add to Info.plist
// NSAlarmKitUsageDescription: "We'll schedule alerts for alarms you create within our app."

func requestAlarmAuthorization() async -> Bool {
    do {
        let state = try await AlarmManager.shared.requestAuthorization()
        return state == .authorized
    } catch {
        print("Error occurred while requesting authorization: \(error)")
        return false
    }
}
```

You can also check the current authorization status:

```swift
func checkAuthorizationStatus() async {
    let status = await AlarmManager.shared.authorizationState
    switch status {
    case .authorized:
        print("Authorized to schedule alarms")
    case .denied:
        print("Permission denied")
    case .notDetermined:
        print("Permission not yet requested")
    @unknown default:
        print("Unknown status")
    }
}
```

> NOTE: Use `authorizationState` — never use `authorizationStatus` when checking the authorization status.

## Creating Alarms

### One-time Alarm

```swift
func createOneTimeAlarm(hour: Int, minute: Int) async throws -> Alarm {
    // Create a unique ID
    let id = UUID()
    
    // Create the schedule for a specific time
    let time = Alarm.Schedule.Relative.Time(hour: hour, minute: minute)
    let schedule = Alarm.Schedule.relative(.init(
        time: time,
        repeats: .never
    ))
    
    // Create the alarm presentation
    let alertContent = AlarmPresentation.Alert(
        title: "Wake Up",
        stopButton: .stopButton,
        secondaryButton: .openAppButton,
        secondaryButtonBehavior: .custom
    )
    
    let presentation = AlarmPresentation(alert: alertContent)
    
    // Create attributes with presentation and metadata
    struct EmptyMetadata: AlarmMetadata {}
    let attributes = AlarmAttributes(
        presentation: presentation,
        metadata: EmptyMetadata(),
        tintColor: .blue
    )
    
    // Create the configuration
    let configuration = AlarmManager.AlarmConfiguration(
        countdownDuration: nil,
        schedule: schedule,
        attributes: attributes,
        sound: .default
    )
    
    // Schedule the alarm
    return try await AlarmManager.shared.schedule(id: id, configuration: configuration)
}
```

### Repeating Alarm

```swift
func createWeeklyAlarm(hour: Int, minute: Int, weekdays: Set<Locale.Weekday>) async throws -> Alarm {
    let id = UUID()
    
    // Create the schedule for repeating on specific days
    let time = Alarm.Schedule.Relative.Time(hour: hour, minute: minute)
    let schedule = Alarm.Schedule.relative(.init(
        time: time,
        repeats: .weekly(Array(weekdays))
    ))
    
    // Create the alarm presentation
    let alertContent = AlarmPresentation.Alert(
        title: "Weekly Reminder",
        stopButton: .stopButton,
        secondaryButton: .snoozeButton,
        secondaryButtonBehavior: .countdown
    )
    
    // Create countdown duration for snooze (9 minutes)
    let countdownDuration = Alarm.CountdownDuration(
        preAlert: nil,
        postAlert: 9 * 60
    )
    
    let presentation = AlarmPresentation(alert: alertContent)
    
    // Create attributes with presentation and metadata
    struct EmptyMetadata: AlarmMetadata {}
    let attributes = AlarmAttributes(
        presentation: presentation,
        metadata: EmptyMetadata(),
        tintColor: .green
    )
    
    // Create the configuration
    let configuration = AlarmManager.AlarmConfiguration(
        countdownDuration: countdownDuration,
        schedule: schedule,
        attributes: attributes,
        sound: .default
    )
    
    // Schedule the alarm
    return try await AlarmManager.shared.schedule(id: id, configuration: configuration)
}
```

### Timer (Countdown)

```swift
func createCountdownTimer(seconds: TimeInterval) async throws -> Alarm {
    let id = UUID()
    
    // Create countdown duration
    let countdownDuration = Alarm.CountdownDuration(
        preAlert: seconds,
        postAlert: 10 // Optional post-alert countdown for snooze
    )
    
    // Create the alarm presentation with all states
    let alertContent = AlarmPresentation.Alert(
        title: "Timer Complete",
        stopButton: .stopButton,
        secondaryButton: .repeatButton,
        secondaryButtonBehavior: .countdown
    )
    
    let countdownContent = AlarmPresentation.Countdown(
        title: "Timer Running",
        pauseButton: .pauseButton
    )
    
    let pausedContent = AlarmPresentation.Paused(
        title: "Timer Paused",
        resumeButton: .resumeButton
    )
    
    let presentation = AlarmPresentation(
        alert: alertContent,
        countdown: countdownContent,
        paused: pausedContent
    )
    
    // Create attributes with presentation and metadata
    struct TimerMetadata: AlarmMetadata {
        let purpose: String
    }
    
    let attributes = AlarmAttributes(
        presentation: presentation,
        metadata: TimerMetadata(purpose: "Cooking Timer"),
        tintColor: .orange
    )
    
    // Create the configuration
    let configuration = AlarmManager.AlarmConfiguration(
        countdownDuration: countdownDuration,
        schedule: nil, // No schedule for a timer
        attributes: attributes,
        sound: .default
    )
    
    // Schedule the alarm
    return try await AlarmManager.shared.schedule(id: id, configuration: configuration)
}
```

## Customizing Alarm UI

### Alert Presentation

```swift
// Basic alert with stop button only
let basicAlert = AlarmPresentation.Alert(
    title: "Alarm",
    stopButton: .stopButton
)

// Alert with stop and snooze buttons
let alertWithSnooze = AlarmPresentation.Alert(
    title: "Wake Up",
    stopButton: .stopButton,
    secondaryButton: .snoozeButton,
    secondaryButtonBehavior: .countdown
)

// Alert with custom button labels
let customAlert = AlarmPresentation.Alert(
    title: "Medication Reminder",
    stopButton: AlarmButton(label: "Taken"),
    secondaryButton: AlarmButton(label: "Remind Later"),
    secondaryButtonBehavior: .countdown
)
```

### Countdown Presentation

```swift
// Basic countdown
let countdown = AlarmPresentation.Countdown(
    title: "Timer",
    pauseButton: .pauseButton
)

// Countdown with custom button label
let customCountdown = AlarmPresentation.Countdown(
    title: "Cooking Timer",
    pauseButton: .pauseButton
)
```

### Paused Presentation

```swift
// Basic paused state
let paused = AlarmPresentation.Paused(
    title: "Paused",
    resumeButton: .resumeButton
)

// Paused with custom button label
let customPaused = AlarmPresentation.Paused(
    title: "Timer Paused",
    resumeButton: .resumeButton
)
```

### Custom Metadata

```swift
// Define custom metadata for your alarm
struct RecipeMetadata: AlarmMetadata {
    let recipeName: String
    let cookingStep: String
}

// Use the metadata in your alarm attributes
let recipeAttributes = AlarmAttributes(
    presentation: presentation,
    metadata: RecipeMetadata(
        recipeName: "Chocolate Cake",
        cookingStep: "Remove from oven"
    ),
    tintColor: .brown
)
```

## Managing Alarms

### Scheduling an Alarm

```swift
func scheduleAlarm(id: UUID, configuration: AlarmManager.AlarmConfiguration) async throws -> Alarm {
    return try await AlarmManager.shared.schedule(id: id, configuration: configuration)
}
```

### Retrieving All Alarms

```swift
func getAllAlarms() throws -> [Alarm] {
    return try AlarmManager.shared.alarms
}
```

### Pausing an Alarm

```swift
func pauseAlarm(id: UUID) async throws {
    try await AlarmManager.shared.pause(id: id)
}
```

### Resuming an Alarm

```swift
func resumeAlarm(id: UUID) async throws {
    try await AlarmManager.shared.resume(id: id)
}
```

### Canceling an Alarm

```swift
func cancelAlarm(id: UUID) async throws {
    try await AlarmManager.shared.cancel(id: id)
}
```

## Observing Alarm Changes

Use the `alarmUpdates` property to observe changes to alarms:

```swift
func observeAlarmChanges() {
    Task {
        for await alarms in AlarmManager.shared.alarmUpdates {
            // Update your UI with the latest alarms
            updateAlarmState(with: alarms)
        }
    }
}

func updateAlarmState(with alarms: [Alarm]) {
    // Update your app's state with the latest alarms
    // An alarm not included in this array is no longer scheduled
}
```

You can also observe authorization state changes:

```swift
func observeAuthorizationChanges() {
    Task {
        for await authState in AlarmManager.shared.authorizationUpdates {
            switch authState {
            case .authorized:
                // User granted permission
            case .denied:
                // User denied permission
            case .notDetermined:
                // Permission not yet requested
            @unknown default:
                break
            }
        }
    }
}
```

## Live Activities Integration

AlarmKit integrates with Live Activities to show countdown timers in the Dynamic Island and Lock Screen. You need to create a widget extension for this:

1. Add a Widget Extension target to your project
2. Implement the widget to display the alarm state:

```swift
struct AlarmWidgetView: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: AlarmAttributes<YourMetadata>.self) { context in
            // Dynamic Island and Lock Screen UI
            VStack {
                Text(context.state.mode == .countdown ? "Counting down" : "Alarm set")
                if context.state.mode == .countdown {
                    Text(timerInterval: context.state.countdownEndDate.timeIntervalSinceNow, countsDown: true)
                        .bold()
                }
            }
            .padding()
        } dynamicIsland: { context in
            // Dynamic Island UI
            DynamicIsland {
                // Expanded UI
                DynamicIslandExpandedRegion(.leading) {
                    Text(context.attributes.presentation.alert.title)
                }
                DynamicIslandExpandedRegion(.trailing) {
                    if context.state.mode == .countdown {
                        Text(timerInterval: context.state.countdownEndDate.timeIntervalSinceNow, countsDown: true)
                    }
                }
            } compactLeading: {
                // Compact leading UI
                Image(systemName: "alarm")
            } compactTrailing: {
                // Compact trailing UI
                if context.state.mode == .countdown {
                    Text(timerInterval: context.state.countdownEndDate.timeIntervalSinceNow, countsDown: true)
                }
            } minimal: {
                // Minimal UI
                Image(systemName: "alarm")
            }
        }
    }
}
```

## SwiftUI Integration Example

Here's a complete example of a simple alarm app using AlarmKit:

```swift
import SwiftUI
import AlarmKit

struct AlarmApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

struct ContentView: View {
    @StateObject private var viewModel = AlarmViewModel()
    @State private var showingAddAlarm = false
    
    var body: some View {
        NavigationView {
            List {
                ForEach(viewModel.alarms, id: \.id) { alarm in
                    AlarmRow(alarm: alarm, viewModel: viewModel)
                }
                .onDelete { indexSet in
                    viewModel.deleteAlarms(at: indexSet)
                }
            }
            .navigationTitle("Alarms")
            .toolbar {
                Button(action: {
                    showingAddAlarm = true
                }) {
                    Image(systemName: "plus")
                }
            }
            .sheet(isPresented: $showingAddAlarm) {
                AddAlarmView(viewModel: viewModel)
            }
            .onAppear {
                viewModel.requestAuthorization()
                viewModel.loadAlarms()
            }
        }
    }
}

@Observable
class AlarmViewModel {
    var alarms: [Alarm] = []
    private let alarmManager = AlarmManager.shared
    
    func requestAuthorization() {
        Task {
            do {
                let state = try await alarmManager.requestAuthorization()
                print("Authorization state: \(state)")
            } catch {
                print("Error requesting authorization: \(error)")
            }
        }
    }
    
    func loadAlarms() {
        Task {
            do {
                let fetchedAlarms = try alarmManager.alarms
                DispatchQueue.main.async {
                    self.alarms = fetchedAlarms
                }
                
                // Start observing alarm updates
                for await updatedAlarms in alarmManager.alarmUpdates {
                    DispatchQueue.main.async {
                        self.alarms = updatedAlarms
                    }
                }
            } catch {
                print("Error loading alarms: \(error)")
            }
        }
    }
    
    func addAlarm(hour: Int, minute: Int, weekdays: Set<Locale.Weekday>) {
        Task {
            do {
                let id = UUID()
                
                // Create schedule
                let time = Alarm.Schedule.Relative.Time(hour: hour, minute: minute)
                let schedule = Alarm.Schedule.relative(.init(
                    time: time,
                    repeats: weekdays.isEmpty ? .never : .weekly(Array(weekdays))
                ))
                
                // Create presentation
                let alertContent = AlarmPresentation.Alert(
                    title: "Alarm",
                    stopButton: .stopButton,
                    secondaryButton: .snoozeButton,
                    secondaryButtonBehavior: .countdown
                )
                
                let presentation = AlarmPresentation(alert: alertContent)
                
                // Create attributes
                struct EmptyMetadata: AlarmMetadata {}
                let attributes = AlarmAttributes(
                    presentation: presentation,
                    metadata: EmptyMetadata(),
                    tintColor: .blue
                )
                
                // Create configuration with 9-minute snooze
                let configuration = AlarmManager.AlarmConfiguration(
                    countdownDuration: Alarm.CountdownDuration(preAlert: nil, postAlert: 9 * 60),
                    schedule: schedule,
                    attributes: attributes,
                    sound: .default
                )
                
                // Schedule the alarm
                let _ = try await alarmManager.schedule(id: id, configuration: configuration)
            } catch {
                print("Error adding alarm: \(error)")
            }
        }
    }
    
    func toggleAlarm(id: UUID, isPaused: Bool) {
        Task {
            do {
                if isPaused {
                    try await alarmManager.resume(id: id)
                } else {
                    try await alarmManager.pause(id: id)
                }
            } catch {
                print("Error toggling alarm: \(error)")
            }
        }
    }
    
    func deleteAlarms(at indexSet: IndexSet) {
        Task {
            for index in indexSet {
                let alarm = alarms[index]
                do {
                    try await alarmManager.cancel(id: alarm.id)
                } catch {
                    print("Error deleting alarm: \(error)")
                }
            }
        }
    }
}

struct AlarmRow: View {
    let alarm: Alarm
    let viewModel: AlarmViewModel
    
    var body: some View {
        HStack {
            VStack(alignment: .leading) {
                Text(formatTime())
                    .font(.title)
                
                if let schedule = alarm.schedule {
                    Text(formatSchedule(schedule))
                        .font(.caption)
                }
            }
            
            Spacer()
            
            if alarm.state == .paused {
                Button(action: {
                    viewModel.toggleAlarm(id: alarm.id, isPaused: true)
                }) {
                    Text("Resume")
                }
            } else if alarm.countdownDuration != nil {
                Button(action: {
                    viewModel.toggleAlarm(id: alarm.id, isPaused: false)
                }) {
                    Text("Pause")
                }
            }
        }
        .padding(.vertical, 8)
    }
    
    private func formatTime() -> String {
        if let schedule = alarm.schedule, case .relative(let relative) = schedule {
            let hour = relative.time.hour
            let minute = relative.time.minute
            return String(format: "%02d:%02d", hour, minute)
        }
        return "Timer"
    }
    
    private func formatSchedule(_ schedule: Alarm.Schedule) -> String {
        if case .relative(let relative) = schedule, case .weekly(let weekdays) = relative.repeats {
            if weekdays.isEmpty {
                return "One time"
            } else {
                return weekdays.map { $0.description }.joined(separator: ", ")
            }
        }
        return "One time"
    }
}

struct AddAlarmView: View {
    let viewModel: AlarmViewModel
    @Environment(\.presentationMode) var presentationMode
    
    @State private var hour = 8
    @State private var minute = 0
    @State private var selectedDays: Set<Locale.Weekday> = []
    
    var body: some View {
        NavigationView {
            Form {
                Section(header: Text("Time")) {
                    DatePicker("", selection: Binding(
                        get: {
                            var components = DateComponents()
                            components.hour = hour
                            components.minute = minute
                            return Calendar.current.date(from: components) ?? Date()
                        },
                        set: { date in
                            let components = Calendar.current.dateComponents([.hour, .minute], from: date)
                            hour = components.hour ?? 8
                            minute = components.minute ?? 0
                        }
                    ), displayedComponents: .hourAndMinute)
                    .datePickerStyle(WheelDatePickerStyle())
                    .labelsHidden()
                }
                
                Section(header: Text("Repeat")) {
                    ForEach([
                        Locale.Weekday.sunday,
                        .monday, .tuesday, .wednesday,
                        .thursday, .friday, .saturday
                    ], id: \.self) { day in
                        Button(action: {
                            if selectedDays.contains(day) {
                                selectedDays.remove(day)
                            } else {
                                selectedDays.insert(day)
                            }
                        }) {
                            HStack {
                                Text(day.description)
                                Spacer()
                                if selectedDays.contains(day) {
                                    Image(systemName: "checkmark")
                                }
                            }
                        }
                        .foregroundColor(.primary)
                    }
                }
            }
            .navigationTitle("Add Alarm")
            .navigationBarItems(
                leading: Button("Cancel") {
                    presentationMode.wrappedValue.dismiss()
                },
                trailing: Button("Save") {
                    viewModel.addAlarm(hour: hour, minute: minute, weekdays: selectedDays)
                    presentationMode.wrappedValue.dismiss()
                }
            )
        }
    }
}
```

## Best Practices

1. **Request Authorization Early**: Request AlarmKit authorization when your app first launches or when the user first attempts to create an alarm.

2. **Handle Authorization Denial**: Provide a clear path for users to enable permissions if they initially deny them.

3. **Provide Clear Usage Description**: Include a clear `NSAlarmKitUsageDescription` in your Info.plist.

4. **Implement Widget Extension**: Always implement a widget extension if your app uses countdown presentations to ensure proper functionality.

5. **Observe Alarm Updates**: Use the `alarmUpdates` async sequence to keep your app's state in sync with the system.

6. **Persist Alarm IDs**: Store the UUIDs of alarms you create to manage them later.

7. **Handle Errors Gracefully**: AlarmKit operations can throw errors, so implement proper error handling.

8. **Limit Number of Alarms**: Be mindful that there's a system limit on the number of alarms an app can schedule.

9. **Customize UI Appropriately**: Provide clear and meaningful titles and button labels for your alarms.

10. **Test on Real Devices**: AlarmKit behavior, especially notifications and Live Activities, should be tested on physical devices.

## References

- [Scheduling an alarm with AlarmKit](https://developer.apple.com/documentation/AlarmKit/scheduling-an-alarm-with-alarmkit)
- [AlarmKit Framework](https://developer.apple.com/documentation/AlarmKit)
- [AlarmManager Class](https://developer.apple.com/documentation/AlarmKit/AlarmManager)
- [Alarm Structure](https://developer.apple.com/documentation/AlarmKit/Alarm)
- [AlarmPresentation Structure](https://developer.apple.com/documentation/AlarmKit/AlarmPresentation)
- [AlarmAttributes Structure](https://developer.apple.com/documentation/AlarmKit/AlarmAttributes)
- [WWDC25 Session 230: Wake up to the AlarmKit API](https://developer.apple.com/wwdc25/230)