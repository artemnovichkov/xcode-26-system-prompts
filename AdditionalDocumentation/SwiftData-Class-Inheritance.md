# Adopting Class Inheritance in Swift Data

## Overview

Swift Data supports class inheritance, allowing you to create hierarchical relationships between your model classes. This powerful feature enables you to build more flexible and specialized data models by creating subclasses that inherit properties and capabilities from a base class. Class inheritance in Swift Data follows the same principles as standard Swift inheritance but with additional considerations for persistence and querying.

Key concepts include:
- Base classes that define common properties and behaviors
- Specialized subclasses that extend functionality for specific use cases
- Type-based querying that can filter by specific model types
- Polymorphic relationships that work with different model types

## When to Use Inheritance in Swift Data

### Good Use Cases

- When you have a clear "IS-A" relationship between models (e.g., a `BusinessTrip` IS-A `Trip`)
- When models share fundamental properties but diverge as use cases become more specialized
- When your app needs to perform both deep searches (across all properties) and shallow searches (specific to subclass properties)
- When your data model naturally forms a hierarchical structure

```swift
// Example of a good inheritance relationship
@Model class Vehicle {
    var manufacturer: String
    var model: String
    var year: Int
}

@Model class Car: Vehicle {
    var numberOfDoors: Int
    var fuelType: FuelType
}

@Model class Motorcycle: Vehicle {
    var engineDisplacement: Int
    var hasABS: Bool
}
```

### When to Avoid Inheritance

- When specialized subclasses would only share a few common properties
- When your query strategy only focuses on specialized properties (shallow queries)
- When a Boolean flag or enumeration could represent the type distinction more efficiently
- When protocol conformance would be more appropriate for shared behavior

```swift
// Alternative to inheritance using an enum approach
@Model class Vehicle {
    var manufacturer: String
    var model: String
    var year: Int
    
    enum VehicleType: String, Codable {
        case car(numberOfDoors: Int, fuelType: FuelType)
        case motorcycle(engineDisplacement: Int, hasABS: Bool)
    }
    
    var type: VehicleType
}
```

## Designing Class Hierarchies

### Base Class Design

1. Identify common properties that all subclasses will share
2. Define relationships that apply to all subclasses
3. Use the `@Model` macro on the base class
4. Ensure the base class is declared as a `class` (not a struct)

```swift
import SwiftData

@Model class Trip {
    @Attribute(.preserveValueOnDeletion)
    var name: String
    var destination: String
    
    @Attribute(.preserveValueOnDeletion)
    var startDate: Date
    
    @Attribute(.preserveValueOnDeletion)
    var endDate: Date

    @Relationship(deleteRule: .cascade, inverse: \Accommodation.trip)
    var accommodation: Accommodation?
    
    init(name: String, destination: String, startDate: Date, endDate: Date) {
        self.name = name
        self.destination = destination
        self.startDate = startDate
        self.endDate = endDate
    }
}
```

### Subclass Design

1. Inherit from the base class using standard Swift inheritance syntax
2. Add the `@Model` macro to the subclass
3. Add specialized properties and relationships specific to the subclass
4. Override methods as needed
5. Consider availability annotations if needed

```swift
@Model class BusinessTrip: Trip {
    var purpose: String
    var expenseCode: String
    var perDiemRate: Double
    
    @Relationship(deleteRule: .cascade, inverse: \BusinessMeal.trip)
    var businessMeals: [BusinessMeal] = []
    
    @Relationship(deleteRule: .cascade, inverse: \MileageRecord.trip)
    var mileageRecords: [MileageRecord] = []
    
    init(name: String, destination: String, startDate: Date, endDate: Date,
         purpose: String, expenseCode: String, perDiemRate: Double) {
        self.purpose = purpose
        self.expenseCode = expenseCode
        self.perDiemRate = perDiemRate
        super.init(name: name, destination: destination, startDate: startDate, endDate: endDate)
    }
}
```

```swift
@Model class PersonalTrip: Trip {
    enum Reason: String, CaseIterable, Codable, Identifiable {
        case family, vacation, wellness, other
        var id: Self { self }
    }
    
    var reason: Reason
    var notes: String?
    
    @Relationship(deleteRule: .cascade, inverse: \Attraction.trip)
    var attractions: [Attraction] = []
    
    init(name: String, destination: String, startDate: Date, endDate: Date,
         reason: Reason, notes: String? = nil) {
        self.reason = reason
        self.notes = notes
        super.init(name: name, destination: destination, startDate: startDate, endDate: endDate)
    }
}
```

## Querying with Inheritance

### Basic Queries

You can query for all instances of a base class, which will include all subclass instances:

```swift
// Query for all trips (including BusinessTrip and PersonalTrip)
@Query(sort: \Trip.startDate)
var allTrips: [Trip]
```

### Type-Based Filtering

Filter by specific subclass types using the `is` operator in predicates:

```swift
// Query for only BusinessTrip instances
let businessTripPredicate = #Predicate<Trip> { $0 is BusinessTrip }
@Query(filter: businessTripPredicate)
var businessTrips: [Trip]
```

### Combined Filtering

Combine type filtering with property filtering:

```swift
// Query for PersonalTrip instances with a specific reason
let personalVacationPredicate = #Predicate<Trip> {
    if let personalTrip = $0 as? PersonalTrip {
        return personalTrip.reason == .vacation
    }
    return false
}

@Query(filter: personalVacationPredicate)
var vacationTrips: [Trip]
```

### Using Enums for Filtering

Create an enum to simplify filtering by type:

```swift
enum TripKind: String, CaseIterable {
    case all = "All"
    case personal = "Personal"
    case business = "Business"
}

struct TripListView: View {
    @Query var trips: [Trip]
    
    init(tripKind: TripKind, searchText: String = "") {
        // Create type predicate based on selected kind
        let typePredicate: Predicate<Trip>? = {
            switch tripKind {
            case .all:
                return nil
            case .personal:
                return #Predicate { $0 is PersonalTrip }
            case .business:
                return #Predicate { $0 is BusinessTrip }
            }
        }()
        
        // Create search predicate if needed
        let searchPredicate = searchText.isEmpty ? nil : #Predicate<Trip> {
            $0.name.localizedStandardContains(searchText) || 
            $0.destination.localizedStandardContains(searchText)
        }
        
        // Combine predicates if both exist
        let finalPredicate: Predicate<Trip>?
        if let typePredicate, let searchPredicate {
            finalPredicate = #Predicate { typePredicate.evaluate($0) && searchPredicate.evaluate($0) }
        } else {
            finalPredicate = typePredicate ?? searchPredicate
        }
        
        _trips = Query(filter: finalPredicate, sort: \.startDate)
    }
}
```

## Working with Subclass Properties

### Type Casting

When working with a collection of base class instances, you'll need to cast to access subclass-specific properties:

```swift
func calculateTotalExpenses(for trips: [Trip]) -> Double {
    var total = 0.0
    
    for trip in trips {
        if let businessTrip = trip as? BusinessTrip {
            // Access BusinessTrip-specific properties
            let perDiemTotal = businessTrip.perDiemRate * Double(Calendar.current.dateComponents([.day], from: businessTrip.startDate, to: businessTrip.endDate).day ?? 0)
            
            // Add meal expenses
            let mealExpenses = businessTrip.businessMeals.reduce(0.0) { $0 + $1.cost }
            
            total += perDiemTotal + mealExpenses
        }
    }
    
    return total
}
```

### Polymorphic Relationships

You can create relationships that work with the base class but contain instances of different subclasses:

```swift
@Model class TravelPlanner {
    var name: String
    
    @Relationship(deleteRule: .cascade)
    var upcomingTrips: [Trip] = []  // Can contain both BusinessTrip and PersonalTrip instances
    
    func addTrip(_ trip: Trip) {
        upcomingTrips.append(trip)
    }
}
```

## Best Practices

1. **Keep inheritance hierarchies shallow**: Avoid deep inheritance chains that can become difficult to maintain.

2. **Use meaningful IS-A relationships**: Only use inheritance when there's a true "is-a" relationship between models.

3. **Consider alternatives**: For simpler cases, enums or Boolean flags might be more appropriate than inheritance.

4. **Design for query patterns**: Consider how you'll query your data when designing your class hierarchy.

5. **Be mindful of schema migrations**: Changes to your inheritance hierarchy may require more complex migrations.

6. **Document the inheritance structure**: Make sure other developers understand the relationships between your models.

7. **Test with real data**: Verify that your inheritance structure works well with realistic data and query patterns.

## References

- [Adopting inheritance in SwiftData](https://developer.apple.com/documentation/SwiftData/Adopting-inheritance-in-SwiftData)
- [Design for specialization](https://developer.apple.com/documentation/SwiftData/Adopting-inheritance-in-SwiftData#Design-for-specialization)
- [Determine whether inheritance is right for your use case](https://developer.apple.com/documentation/SwiftData/Adopting-inheritance-in-SwiftData#Determine-whether-inheritance-is-right-for-your-use-case)
- [Fetch and Query Data](https://developer.apple.com/documentation/SwiftData/Adopting-inheritance-in-SwiftData#Fetch-and-Query-Data)