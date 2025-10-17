# SwiftyPython User Guide

## Table of Contents
- [Introduction](#introduction)
- [Features](#features)
- [Installation](#installation)
- [Basic Setup](#basic-setup)
- [Getting Started](#getting-started)
- [Type Conversion](#type-conversion)
- [Working with Python Modules](#working-with-python-modules)
- [Error Handling](#error-handling)
- [Advanced Usage](#advanced-usage)
- [SwiftUI Integration](#swiftui-integration)
- [Demo Applications](#demo-applications)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Introduction

SwiftyPython is a powerful Swift Package Manager package that enables seamless integration of Python code within macOS applications. It provides a type-safe, Swift-friendly interface to the Python ecosystem, allowing developers to leverage Python's extensive libraries while building native macOS apps with Swift.

## Features

- **Type-Safe Interface**: Provides type conversion between Swift and Python types
- **Dynamic Member Lookup**: Access Python modules and attributes naturally from Swift
- **Dynamic Callable Support**: Call Python functions with native Swift syntax
- **Error Handling**: Converts Python exceptions to Swift errors
- **Virtual Environment Support**: Manages Python virtual environments within your app bundle
- **Package Management**: Automatically installs Python packages from requirements.txt
- **Bundled Libraries**: Includes common Python libraries (numpy, pillow, boto3, matplotlib, certifi)
- **Collection Protocol Conformance**: Treat Python iterables as Swift collections

## Installation

### Adding to Your Xcode Project

1. Open your Xcode project
2. Select File > Add Packages...
3. Enter the repository URL: `https://github.com/r0ml/SwiftyPython`
4. Choose the desired version or branch
5. Click "Add Package"
6. Select the target you want to add the package to

### Project Configuration

After adding SwiftyPython to your project, you need to make the following configurations:

1. **Create a venv Directory**: Create an empty directory named `venv` in your app bundle (usually in the Resources folder)
2. **Add Requirements File** (Optional): Create a `requirements.txt` file in your project root if you need additional Python packages
3. **Disable Library Validation** (If using Hardened Runtime):
   - Select your target
   - Go to Signing & Capabilities
   - Check "Disable Library Validation" under Hardened Runtime
4. **Disable User Script Sandboxing**:
   - Select your target
   - Go to Build Settings
   - Search for "Sandbox"
   - Set "Enable User Script Sandboxing" to NO

## Basic Setup

### Importing the Module

```swift
import PythonSupport
```

### Initializing Python

Initialize the Python interpreter in your application startup code. For a SwiftUI app, this would typically be in your App struct:

```swift
import SwiftUI
import PythonSupport

@main struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .task {
                    await MainActor.run {
                        // Initialize Python
                        let p = Python
                        Python.start()
                    }
                }
        }
    }
}
```

## Getting Started

### Running Python Code

The simplest way to execute Python code is using the `run` method:

```swift
try Python.run("print('Hello from Python!')")
```

### Returning Values from Python

You can capture return values from Python code using the `returning` parameter:

```swift
let result = try Python.run(
    "x = 42; result = x * 2", 
    returning: "result"
)

if let value = Int(result!) {
    print("Swift received: \(value)") // Prints: Swift received: 84
}
```

### Accessing Python Modules

Access Python modules using dynamic member lookup:

```swift
let math = Python.math
let pi = try Double(math.pi)
print("π = \(pi)")
```

### Calling Python Functions

Call Python functions using dynamic callable syntax:

```swift
let sqrtResult = try Double(math.sqrt(16))
print("Square root of 16 = \(sqrtResult)")
```

## Type Conversion

SwiftyPython provides seamless type conversion between Swift and Python types through two protocols:

- `ConvertibleFromPython`: For converting Python objects to Swift types
- `ConvertibleToPython`: For converting Swift types to Python objects

### Supported Type Conversions

| Swift Type | Python Type |
|------------|-------------|
| Bool | bool |
| Int | int |
| UInt | int |
| Double | float |
| String | str |
| Array | list |
| Dictionary | dict |
| Optional | None or wrapped type |
| Data | bytes |
| Range | slice |

### Explicit Type Conversion

Convert Python objects to Swift types explicitly:

```swift
let pythonValue = try Python.run("value = '42'", returning: "value")!

// Explicit conversion
if let stringValue = String(pythonValue) {
    print(stringValue)
}

// Then convert to other Swift types
if let intValue = Int(stringValue) {
    print(intValue)
}
```

## Working with Python Modules

### Importing and Using Modules

```swift
// Import a module
let numpy = Python.numpy

// Use the module
let array = try numpy.array([1, 2, 3, 4])
let sumResult = try Double(numpy.sum(array))
print("Sum: \(sumResult)")
```

### Importing with From Syntax

Use the `imports` method for Python's `from module import name` syntax:

```swift
let sqrt = Python.imports("sqrt", from: "math")
let result = try Double(sqrt(25))
```

## Error Handling

### Python Exceptions

Python exceptions are converted to Swift's `PythonError` enum:

```swift
do {
    try Python.run("result = 1 / 0")
} catch let error as PythonError {
    print("Python error: \(error)")
}
```

### Common Error Types

- `PythonError.exception(PythonObject, traceback: PythonObject?)`: Python exception
- `PythonError.invalidCall(PythonObject)`: Invalid function call
- `PythonError.invalidModule(String)`: Module not found
- `PythonError.indexError(PythonObject)`: Index out of range
- `PythonError.runError(PythonObject)`: Error during code execution

## Advanced Usage

### Working with Collections

Python objects that support iteration can be treated as Swift sequences:

```swift
// Create a Python list
let pythonList = PythonObject(tupleOf: 1, 2, 3, 4, 5)

// Iterate over it in Swift
for item in pythonList {
    if let number = Int(item) {
        print(number * 2)
    }
}
```

### Dictionary Access

Access Python dictionaries using subscript notation:

```swift
try Python.run(
    "person = {'name': 'John', 'age': 30, 'city': 'New York'}",
    returning: "person"
)
let person = try Python.run("person", returning: "person")!

if let name = String(person["name"]) {
    print("Name: \(name)")
}

if let age = Int(person["age"]) {
    print("Age: \(age)")
}
```

### Keyword Arguments

Pass keyword arguments to Python functions:

```swift
let result = try Python.sorted([3, 1, 2], key: { x in x * -1 }, reverse: true)
```

### Numpy Integration

SwiftyPython provides special support for numpy arrays:

```swift
// Create a numpy array from Swift
let np = Python.numpy
let arr = try np.array([1.0, 2.0, 3.0])

// Perform operations
let doubled = try np.multiply(arr, 2)

// Convert back to Swift
if let swiftArray = [Double](numpyArray: doubled) {
    print(swiftArray) // [2.0, 4.0, 6.0]
}
```

## SwiftUI Integration

### Displaying Python Output

Use `stdout` to capture and display Python output in your SwiftUI views:

```swift
import SwiftUI
import PythonSupport

struct OutputView: View {
    @State private var output: String = ""
    
    var body: some View {
        Text(output)
            .font(.monospacedSystemFont(ofSize: 14, weight: .regular))
            .padding()
            .onAppear {
                // Capture stdout
                output = stdout.capturedOutput
            }
    }
}
```

### Handling Python Results in Views

```swift
struct PythonResultView: View {
    @State private var result: String = ""
    
    var body: some View {
        VStack {
            Button("Run Python Code") { runPythonCode() }
            Text("Result: \(result)")
        }
    }
    
    func runPythonCode() {
        Task {
            await MainActor.run {
                do {
                    let pythonResult = try Python.run(
                        "import random\nresult = random.randint(1, 100)",
                        returning: "result"
                    )
                    if let value = Int(pythonResult!) {
                        result = "\(value)"
                    }
                } catch {
                    result = "Error: \(error.localizedDescription)"
                }
            }
        }
    }
}
```

## Demo Applications

SwiftyPython includes several demo applications that showcase different use cases:

### 1. Image Processing with PIL

Converts images to ASCII art using Python's PIL library:

```swift
// Example from AsciifyDemo
let k = try Python.run("""
import PIL.Image

img = PIL.Image.open('\(imagePath)')
img = img.resize((120, 60))
img = img.convert('L')

chars = ['@', '#', 'S', '%', '?', '*', '+', ';', ':', ',', '.']
pixels = img.getdata()
ascii_str = ''.join([chars[pixel//25] for pixel in pixels])
""", returning: "ascii_str")

if let asciiArt = String(k!) {
    // Display ASCII art
}
```

### 2. Data Visualization with Matplotlib

Creates charts and plots using matplotlib and numpy:

```swift
let np = Python.numpy
let plt = Python.matplotlib.pyplot

// Create data
let x = np.linspace(0, 10, 100)
let y = np.sin(x)

// Create plot
plt.figure(figsize: (10, 6))
plt.plot(x, y)
plt.title("Sine Wave")
plt.savefig("plot.png")
plt.close()
```

### 3. Web Scraping with Bing Image Downloader

Downloads images from Bing search:

```swift
try Python.run("""
from bing_image_downloader import downloader

downloader.download('Swift programming language', limit=5, output_dir='images')
""")
```

### 4. HTML Generation with Dominate

Creates HTML documents programmatically:

```swift
let html = try Python.run("""
from dominate import document
from dominate.tags import h1, p, a

doc = document()
with doc:
    h1('Hello, World!')
    p('This HTML was generated using Python's dominate library.')
    a('Visit GitHub', href='https://github.com')
""", returning: "str(doc)")

if let htmlString = String(html!) {
    // Display or save HTML
}
```

## Best Practices

### 1. Memory Management

- Always handle Python objects with care to avoid memory leaks
- Use explicit type conversion when you know the expected type
- Be mindful of reference counting when passing objects between Swift and Python

### 2. Error Handling

- Always use try/catch blocks when executing Python code
- Handle Python exceptions gracefully in your UI
- Log Python tracebacks for debugging

### 3. Performance Considerations

- Minimize the number of Swift-Python boundary crossings
- Batch Python operations when possible
- Use numpy arrays for efficient numerical computations

### 4. Code Organization

- Separate Python code into well-defined functions or modules
- Keep Python code simple and focused on specific tasks
- Document the expected behavior and data formats

## Troubleshooting

### Common Issues and Solutions

#### Python Module Not Found

**Issue**: Python raises ModuleNotFoundError

**Solution**: 
- Add the module to requirements.txt
- Ensure your venv directory is properly set up
- Check that PYTHONPATH includes the correct directories

#### ImportError or AttributeError

**Issue**: Cannot import module or access attribute

**Solution**:
- Verify the module name and attribute exist
- Check version compatibility
- Ensure correct Python environment is initialized

#### Memory Leaks

**Issue**: Application memory usage grows over time

**Solution**:
- Release Python objects when no longer needed
- Avoid circular references between Swift and Python objects
- Use autoreleasepools for large operations

#### Hardened Runtime Errors

**Issue**: Errors when running with hardened runtime enabled

**Solution**:
- Disable library validation in your app entitlements
- Ensure all required libraries are properly signed

#### Sandbox Violations

**Issue**: Sandbox violations when installing Python packages

**Solution**:
- Disable user script sandboxing in build settings
- Ensure your app has the necessary entitlements

### Debugging Tips

1. **Capture Python Output**: Use `stdout.capturedOutput` to see Python's print statements
2. **Print Tracebacks**: Catch PythonError and print the traceback information
3. **Test in Isolation**: Test Python code in a standalone script before integrating with Swift
4. **Check Python Version**: Verify compatibility between your code and the bundled Python version
5. **Monitor Memory Usage**: Use Instruments to identify memory issues

## Conclusion

SwiftyPython provides a powerful bridge between the Swift and Python ecosystems, allowing you to leverage the strengths of both languages in your macOS applications. By following the patterns and best practices outlined in this guide, you can create robust applications that seamlessly integrate Python functionality while maintaining the performance and user experience benefits of native Swift development.