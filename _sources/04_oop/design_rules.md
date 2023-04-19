# Design rules summary

## Keep methods small

- Keep methods short and simple: not too deep nesting of flow control blocks (maximum four) and not too many method parameters. Prefer a small data class (record) as return value over complex data structures.

## Classes

- When a class defines an instance variable that needs to be initialized in order to have an object that makes sense, and you can not give it a reasonable default value, you should make it a constructor parameter 
- Chain constructors using `this(fieldValue, fieldValue)`
- Consider the use of factory methods instead of constructors
- Make classes abstract that should not be instantiated
- Preferably use records for data classes instead of normal classes; they have immutability in their nature already.

## Access modifiers

- Keep the number of public methods as low as possible
- Minimize the accessibility of class members(fields, methods, member classes).
- Use getters and setters to intercept and control access to fields.

Changing public API methods is a nuisance since it may create backward incompatibility.

## `equals()` and `hashCode()`

- Always implement equals() and hashCode() when objects are going to live in collections (which is quite often!)
- Always implement both equals and hashCode; never only one!
- And have your IDE generate them for you...

## Code against interfaces, not implementations

Use interfaces and abstract classes as the declared type of your variables. That way, implementation details may change without serious repercussions.

## Exceptions

- Never let exceptions pass silently
- Deal with exceptions at an appropriate location
- Nowadays, the trend is to prefer unchecked exceptions over checked exceptions (see discussion at http://www.yegor256.com/2015/07/28/checked-vs-unchecked-exceptions.html)
- Never ever try to catch an Error

## Single Responsibility Principle

- A module, class or method should have one, and only one, reason to change.  

Alternative description: every module, class, or function should have responsibility over a single part of the functionality, and that responsibility should be entirely encapsulated by the class

## Various

- Prefer enums to define a static set of possible values for a given property above `public static final` constants
- Always put Java classes in well-defined packages with a correct name structure
