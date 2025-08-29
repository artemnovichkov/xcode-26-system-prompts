# Swift Standard Library: InlineArray and Span

## Overview

InlineArray and Span are two new types introduced in Swift 6.2 to enhance performance of critical code. These types are designed to provide more efficient memory usage and better performance in specific scenarios where standard Swift collections might introduce overhead.

Key concepts:
- **InlineArray**: A fixed-size array with inline storage that eliminates heap allocation
- **Span**: A safe abstraction for accessing contiguous memory without the dangers of unsafe pointers

## InlineArray

### What is InlineArray?

`InlineArray` is a specialized array type that stores its elements contiguously inline, rather than allocating an out-of-line region of memory with copy-on-write optimization. This means the elements are stored directly within the array's memory layout, not in a separate heap allocation.

### Declaration

```swift
@frozen struct InlineArray<let count: Int, Element> where Element: ~Copyable
```

The `count` parameter uses Swift's value generics feature to make the size part of the type.

### Key Characteristics

- **Fixed size**: Size is specified at compile time and cannot be changed
- **Inline storage**: Elements are stored directly, not via a reference to a buffer
- **No heap allocation**: Can be stored on the stack or directly within other types
- **No dynamic resizing**: Cannot append or remove elements
- **No copy-on-write**: Copies are made eagerly when assigned to a new variable
- **No reference counting**: Eliminates the overhead of retain/release operations
- **No exclusivity checks**: Improves performance in hot paths

### Initialization

Array literals can be used to initialize an InlineArray:

```swift
// Explicit type specification
let a: InlineArray<4, Int> = [1, 2, 4, 8]

// Type inference for count
let b: InlineArray<_, Int> = [1, 2, 4, 8]  // count inferred as 4

// Type inference for element type
let c: InlineArray<4, _> = [1, 2, 4, 8]    // Element type inferred as Int

// Type inference for both
let d: InlineArray = [1, 2, 4, 8]          // InlineArray<4, Int>
```

### Memory Layout

```swift
// Empty array
MemoryLayout<InlineArray<0, UInt16>>.size       // 0
MemoryLayout<InlineArray<0, UInt16>>.stride     // 1
MemoryLayout<InlineArray<0, UInt16>>.alignment  // 1

// Non-empty array
MemoryLayout<InlineArray<3, UInt16>>.size       // 6 (2 bytes × 3 elements)
MemoryLayout<InlineArray<3, UInt16>>.stride     // 6
MemoryLayout<InlineArray<3, UInt16>>.alignment  // 2 (same as UInt16)
```

### Basic Usage

```swift
// Create an inline array
var array: InlineArray<3, Int> = [1, 2, 3]

// Modify elements
array[0] = 4
// array == [4, 2, 3]

// Cannot append or remove elements
// array.append(4)  // Error: Value of type 'InlineArray<3, Int>' has no member 'append'

// Cannot assign to a different sized inline array
// let bigger: InlineArray<6, Int> = array  // Error: Cannot assign value of type 'InlineArray<3, Int>' to type 'InlineArray<6, Int>'

// Copying behavior
var copy = array    // copy happens on assignment
for i in copy.indices {
    copy[i] += 10
}
// array == [4, 2, 3]
// copy == [14, 12, 13]
```

### Common Properties and Methods

```swift
var array: InlineArray<3, Int> = [1, 2, 3]

// Properties
array.count        // 3
array.isEmpty      // false
array.indices      // 0..<3

// Accessing elements
let firstElement = array[0]  // 1

// Iteration
for element in array {
    print(element)
}

// Using indices
for i in array.indices {
    print(array[i])
}
```

### When to Use InlineArray

InlineArray is ideal for:
- Performance-critical code paths
- Fixed-size collections that never change size
- Avoiding heap allocations and reference counting overhead
- Collections that are modified in place but rarely copied
- Embedded systems or low-level programming

Not suitable for:
- Collections that need to grow or shrink
- Collections that benefit from copy-on-write semantics
- Collections that are frequently copied or shared between variables

## Span

### What is Span?

`Span` is an abstraction that provides fast, direct access to contiguous memory without compromising memory safety. It's designed as a safe alternative to unsafe pointers, providing efficient access to the underlying storage of containers like Array and InlineArray.

### Declarations

```swift
@frozen struct Span<Element> where Element : ~Copyable
```

Span<Element> represents a contiguous region of memory which contains initialized instances of Element.

```swift
var span: Span<Element> { get }
var mutableSpan: MutableSpan<Element> { mutating get }
```

Instance Properties of Array.

### Key Characteristics

- **Memory safety**: Ensures memory validity through compile-time checks
- **Lifetime dependency**: Cannot outlive the original container
- **No runtime overhead**: Safety checks are performed at compile time
- **Direct access**: Provides efficient access to contiguous memory
- **Non-escapable**: Cannot be returned from functions or captured in closures
- **Prevents common memory issues**: Use-after-free, overlapping modification, etc.

### Types of Spans

The Span family includes:
- **Span<Element>**: Read-only access to contiguous elements
- **MutableSpan<Element>**: Mutable access to contiguous elements
- **RawSpan**: Read-only access to raw bytes
- **MutableRawSpan**: Mutable access to raw bytes
- **OutputSpan**: For initializing a new collection
- **UTF8Span**: Specialized for safe and efficient Unicode processing

### Accessing Spans

Containers with contiguous storage provide a `span` property:

```swift
let array = [1, 2, 3, 4]
let span = array.span  // Get a Span<Int> over the array's elements

// Data also provides span access
let data = Data([0, 1, 2, 3])
let dataSpan = data.span  // Get a Span<UInt8>
```

### Safe Usage

```swift
// Safe usage of a span
func processUsingSpan(_ array: [Int]) -> Int {
    let intSpan = array.span
    var result = 0
    for i in 0..<intSpan.count {
        result += intSpan[i]
    }
    return result
}
```

### Safety Constraints

Spans have compile-time safety constraints:

1. **Cannot escape their scope**:
```swift
// This won't compile
func getSpan() -> Span<UInt8> {
    let array: [UInt8] = Array(repeating: 0, count: 128)
    return array.span  // Error: Cannot return span that depends on local variable
}
```

2. **Cannot be captured in closures**:
```swift
// This won't compile
func getClosure() -> () -> Int {
    let array: [UInt8] = Array(repeating: 0, count: 128)
    let span = array.span
    return { span.count }  // Error: Cannot capture span in closure
}
```

3. **Cannot access span after modifying the original container**:
```swift
var array = [1, 2, 3]
let span = array.span
array.append(4)
// let x = span[0]  // Error: Cannot access span after modifying the original container
```

### Example: Implementing a Method Using RawSpan

```swift
extension RawSpan {
    mutating func readByte() -> UInt8? {
        guard !isEmpty else { return nil }
        
        let value = unsafeLoadUnaligned(as: UInt8.self)
        self = self._extracting(droppingFirst: 1)
        return value
    }
}
```

### When to Use Span

Span is ideal for:
- Efficiently operating on contiguous memory without unsafe pointers
- Processing large amounts of data with minimal overhead
- Implementing high-performance algorithms that need direct memory access
- Safely working with the underlying storage of collections
- Binary parsing and other low-level operations

## Performance Considerations

### InlineArray vs. Array

Standard Array:
- Stores elements in a separate heap allocation
- Uses reference counting to track copies
- Performs uniqueness checks on mutation
- Enforces exclusivity at runtime in some cases
- Supports dynamic resizing

InlineArray:
- Stores elements directly inline
- No reference counting overhead
- No uniqueness checks
- No runtime exclusivity checks
- Fixed size known at compile time

### Span vs. Unsafe Pointers

Unsafe Pointers:
- Require manual memory management
- Prone to memory safety issues
- No compile-time safety guarantees
- Can lead to crashes or undefined behavior

Span:
- Provides memory safety through compile-time checks
- Prevents use-after-free and similar issues
- No runtime overhead compared to pointers
- Cannot outlive the original container

## References

- [Swift Documentation: InlineArray](https://developer.apple.com/documentation/Swift/InlineArray)
- [Swift Documentation: Span](https://developer.apple.com/documentation/Swift/Span)
- [Swift Documentation: Array.span](https://developer.apple.com/documentation/swift/array/span)
- [Swift Documentation: Array.mutableSpan](https://developer.apple.com/documentation/swift/array/mutablespan)
- [WWDC 2025 Session: What's new in Swift](https://developer.apple.com/videos/play/wwdc2025/245/)
- [WWDC 2025 Session: Improve memory usage and performance with Swift](https://developer.apple.com/videos/play/wwdc2025/312/)