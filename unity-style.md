# Unity C# Style Guide

## File Organization

### File Naming
- File name must match the primary class name in the file (one class per file)
- Example: `LoggerFilter.cs` contains the `LoggerFilter` class

### File Encoding
- UTF-8 encoding is required
- Non-ASCII characters should be used directly rather than using Unicode escape sequences
  ```csharp
  // Good
  string unitAbbrev = "μs";
  // Bad
  string unitAbbrev = "\u03bcs";
  ```

### File Structure Example
```csharp
using System;
using System.Xml;
using Adept.Logger;
// ... other using statements

namespace Avataria3d.Game.Core.Filter
{
    public class LoggerFilter : IAppFilter
    {
        public async UniTask RunAsync()
        {
            // Implementation
        }
    }
}
```

## Variable Declaration

### Using var vs Explicit Types
- Use explicit type declarations when there is any ambiguity about the variable's type
- Avoid using `var` if the type is not immediately obvious from the right side of the assignment
- When in doubt, use explicit type declaration

Examples:
```csharp
// Bad - type not immediately clear
var result = GetData();
var handler = CreateHandler();

// Good - explicit types when purpose/type might be ambiguous
IDataResult result = GetData();
EventHandler handler = CreateHandler();

// Good - type is obvious from initialization
XmlDocument xml = XmlUtils.CreateDocument(LogConfig.XML);
DictionaryConfiguration config = new(xml);
string? loggerName = config.GetString("activeLogger");

// Bad - using var when type might not be obvious
var xml = XmlUtils.CreateDocument(LogConfig.XML);
var config = new DictionaryConfiguration(xml);
var loggerName = config.GetString("activeLogger");
```

## Formatting

### Braces
- Always use braces, even for single-line blocks
- Use K&R style (same line) for:
  - Conditional statements
  - Switch statements
  - Loops
  - Anonymous functions
  - Object and array initializers
  
- Use BSD style (new line) for:
  - Namespace declarations
  - Class/interface declarations
  - Method declarations

Example:
```csharp
namespace Avataria3d.Game.Core.Filter
{
    public class LoggerFilter : IAppFilter
    {
        public async UniTask RunAsync()
        {
            if (string.IsNullOrEmpty(loggerName)) {
                Debug.LogWarning("active logger not set");
                return;
            }
        }
    }
}
```

### Indentation
- Use 4 spaces for indentation
- Do not use tabs
- Each new block increases indentation by 4 spaces
- Maximum line length is 150 characters

### Line Breaks
- One statement per line
- Line continuation should break:
  - Before operators (except assignment operators)
  - Before dots (.) and pipes (|)
  - After assignment operators
  - Keep commas attached to the preceding token

### Empty Lines
- Maximum two consecutive empty lines
- Empty lines can be used to improve readability between logical sections
- No empty lines before first or after last member/initializer in a class

### Spacing
Spaces are required:
- Before opening parenthesis in control structures (if, for, catch)
- After closing brace before else or catch
- After opening brace (except in attributes)
- Around all operators except dot operator
- After commas, colons, and semicolons
- After type casts
- After comment markers
- Between type declaration and variable name

## Dependency Injection

### Injection Methods Priority
1. Constructor Injection (Preferred)
   - Use constructor injection as the primary method for required dependencies
   - Makes dependencies explicit and ensures proper initialization
   - Helps maintain immutability
   - Easier to test

2. Field Injection (Secondary)
   - Use `[Inject]` attribute for field injection only when constructor injection is not possible
   - Common in MonoBehaviour classes where constructor injection isn't available
   - Mark injected fields as `private`

Examples:

```csharp
// Good - Constructor injection
public class ServiceClass
{
    private readonly IDataService _dataService;
    private readonly ILoggerService _loggerService;

    public ServiceClass(IDataService dataService, ILoggerService loggerService)
    {
        _dataService = dataService;
        _loggerService = loggerService;
    }
}

// Acceptable - Field injection for MonoBehaviour
public class GameComponent : MonoBehaviour
{
    [Inject]
    private IDataService _dataService = null!;
    
    [Inject]
    private ILoggerService _loggerService = null!;
}

// Bad - Field injection when constructor injection is possible
public class ServiceClass
{
    [Inject]
    private IDataService _dataService = null!;
    
    [Inject]
    private ILoggerService _loggerService = null!;
}
```

### Injection Guidelines
1. Always prefer constructor injection for non-MonoBehaviour classes
2. Use field injection with `[Inject]` attribute only when:
   - The class inherits from MonoBehaviour
   - Constructor injection is not possible due to framework limitations
3. Do not mark injected fields as `readonly` as this prevents the injector from setting the field
4. Initialize injected fields with `null!` to satisfy null safety requirements
5. Keep injected dependencies private unless there's a specific reason for other access levels

### Benefits of Constructor Injection
- Makes dependencies explicit and visible
- Ensures proper initialization order
- Facilitates unit testing
- Helps identify dependency cycles early
- Makes it clear which dependencies are required for the class to function

## Code Organization

### Class Member Ordering
1. Constants and static fields
2. Delegates
3. Transfers
4. Bindings
5. Injections
6. Auto-properties and public fields
7. Private fields
8. Constructor
9. MonoBehaviour methods (Awake, Init, OnDestroy, Update)
10. Public methods
11. Event handlers with binding
12. Event handlers
13. Private methods
14. Properties

Example from LoggerFilter.cs:
```csharp
public class LoggerFilter : IAppFilter
{
    // Public method
    public async UniTask RunAsync()
    {
        XmlDocument xml = XmlUtils.CreateDocument(LogConfig.XML);
        DictionaryConfiguration config = new(xml);
        string? loggerName = config.GetString("activeLogger");
        
        if (string.IsNullOrEmpty(loggerName)) {
            Debug.LogWarning("active logger not set");
            return;
        }
        
        if (!LoggerConfigurator.Configure(xml, (LoggerType) Enum.Parse(typeof(LoggerType), loggerName!, true))) {
            Debug.LogWarning("logger config empty");
        }
        
        LoggerConfigurator.AddParam("unityAnalyticsUserId", AnalyticsSessionInfo.userId);
        LoggerConfigurator.AddParam("deviceName", SystemInfo.deviceModel);
        LoggerConfigurator.AddParam("operatingSystem", SystemInfo.operatingSystem);
    }
}
```

## Naming Conventions

### Namespaces
- PascalCase
- Follow directory structure
- Format: `<Product>.<Feature>[.<Subnamespace>]`
- Example: `Avataria3d.Game.Core.Filter`

### Classes and Interfaces
- PascalCase
- Classes: noun or noun phrase
- Interfaces: prefix with 'I'
- Example: `LoggerFilter`, `IAppFilter`

### Methods
- PascalCase
- Should contain a verb
- Example: `RunAsync()`, `Configure()`

### Fields
- camelCase
- Non-public fields start with underscore (_)
- Example:
```csharp
public int publicField;
private int _privateField;
protected int _protectedField;
```

### Constants
- All uppercase with underscores
- Example: `public const int THE_ANSWER = 42;`

### Parameters
- camelCase
- Example: `public void DoSomething(Vector3 location)`

### Delegates
- PascalCase
- Use EventHandler suffix for event delegates
- Use Callback suffix for other delegates
- Example:
```csharp
public delegate void ClickEventHandler()
public delegate void RenderCallback()
```

### Events
- Start with "On"
- Example: `public static event CloseEventHandler OnClose;`

### Variables
- camelCase
- Example: `string myVariable`

### Abbreviations
- Treat as words (PascalCase or camelCase as appropriate)
- Example:
```csharp
XmlHttpRequest
string url
findPostById
```
```

This is the complete, updated style guide incorporating all the previous sections plus the corrections regarding dependency injection and variable declarations. The document provides a comprehensive guide for C# coding in Unity, with clear examples and explanations for each section.