---
title: "Builder Usage Guide"
permalink: /fluent-builders/builder-guide
---
{% raw %}
## **General Builder Tips, Tricks and Tidbits**

* Calling `Build()` is optional, the builder has a defined `implicit` conversion to the type its building. So as long as you store it to a variable of that type or pass it to a method expecting that type, then `Build()` will be called for you.

* The builder cannot currently make method calls, but supports nearly every variation of a property that C# allows for. This means a property setter can be any access level (public, internal, private) and the builder will be able to use it, the only exception are `get` only properties since there is no backing `set` method generated in the IL at all. This also includes custom indexed properties like `public string this[int s] { ... }` where the index value can either be provided as an inline value, or by a variable; see below for more details on this.

* The builder **does not** call the expressions passed to it in calls to `With`, instead it decomposes them into a data structure that allows it to set the properties via reflection. So it is possible to give the builder an expression that it cannot support, like a method call. The builder will throw an exception in these cases.

## **Simple With Statement Usages**
### **Setting a Property to a Specific Value**
The simplest way to use the builder is to just call `With` on a property and provide the value you'd like to set it to.

**Example:**
```csharp
BuilderFactory.Create<MyObject>()
    .With(u => u.SomeStringProperty, "someString")
    .Build();
```
**Result as Json:**
``` javascript
{
    "SomeStringProperty": "someString"
}
```

### **Setting a Property to `default`**
If you would like to set a property to simply be `default` it is as simple as just not providing a value to the `With` call. 

This is something that is not hugely useful as it could also be achieved by just not calling with. One use case could be on a shared builder thats being used to produce multiple built objects, or also if the class being built has a defined default value and you need to roll it back to `default`.

**Example:**
```csharp
BuilderFactory.Create<MyObject>()
    .With(u => u.SomeStringProperty)
    .Build();
```
**Result as Json:**
``` javascript
{
    "SomeStringProperty": null
}
```

### **Setting an `IEnumerable` Property**
If you're setting the value(s) for an `IEnumerable<T>` property (or any subclasses thereof) rather than creating a new `List<T>` to pass as the value you can just provide zero or more values directly to `With`.

**Example for zero items:**
```csharp
    BuilderFactory.Create<MyObject>()
        .With(u => u.SomeStringList)
        .Build();
```
**Result as Json:**
```javascript
{
    "SomeStringList": []
}
```

**Example for multiple items:**
```csharp
BuilderFactory.Create<MyObject>()
    .With(u => u.SomeStringList, "string1", "string2")
    .Build();
```
**Result as Json:**
```javascript
{
    "SomeStringList": ["string1","string2"]
}
```

### **Setting a Nested Property**
If you need to create an object that has complex object properties where you only really care about values in the child objects or lower you can just chain the property accesses to set the specific descendant properties that you care about. This does work with immutable objects as well so long as a Constructor has been configured; the builder will essentially create a new builder behind the scenes to manage these.

**Example for immediate child:**
```csharp
    BuilderFactory.Create<MyObject>()
        .With(u => u.SomeObject.SomeStringProperty, "stringValue")
        .Build();
```
**Result as Json:**
```javascript
{
    "SomeObject": {
        "SomeStringProperty": "stringValue"
    }
}
```

**Example for multi-level descendant:**
```csharp
    BuilderFactory.Create<MyObject>()
        .With(u => u.SomeObjectOne.SomeObjectTwo.SomeObjectThree.SomeStringProperty, "stringValue")
        .Build();
```
**Result as Json:**
```javascript
{
    "SomeObjectOne": {
        "SomeObjectTwo": {
            "SomeObjectThree":{
                "SomeStringProperty": "stringValue"
            }
        }
    }
}
```

### **Setting an Indexed Property**
If you're build a class that has a custom indexer, or maybe is just wrapping another type that has one and provides a passthrough then you can have the builder call the backing `set` for a specific index. The index can be provided either inline or as a variable; when using a variable, standard lambda closure issues can arise. This can also be used for implementations with multiple index parameters.

_**Note:** These examples aren't legit json, they are more like pseudo code to give a rough idea of the object's structure_

**Example inline index:**
```csharp
    BuilderFactory.Create<MyObject>()
        .With(u => u[0], "stringValue")
        .Build();
```
**Result as Json:**
```javascript
{
    [{0:"stringValue"}]
}
```

**Example variable index:**
```csharp
    var index = 1;

    BuilderFactory.Create<MyObject>()
        .With(u => u[index], "stringValue")
        .Build();
```
**Result as Json:**
```javascript
{
    [{1:"stringValue"}]
}
```

**Example with multiple indexes:**
```csharp
    BuilderFactory.Create<MyObject>()
        .With(u => u[0,"someKey"], "stringValue")
        .Build();
```
**Result as Json:**
```javascript
{
    [{{0,"someKey"}: "stringValue"}]
}
```

**Example with multiple with calls:**
```csharp
    BuilderFactory.Create<MyObject>()
        .With(u => u[0], "stringValue1")
        .With(u => u[1], "stringValue2")
        .Build();
```
**Result as Json:**
```javascript
{
    [
        {0: "stringValue1"},
        {1: "stringValue2"}
    ]
}
```

**Example using variable closure:**
```csharp
    var key = "someKey1";
    var builder = BuilderFactory.Create<MyObject>()
        .With(u => u[key], "stringValue");

    var obj1 = builder.Build();

    key = "someKey2";

    var obj2 = builder.Build();
```
**Result as Json:**
```javascript
obj1 = {[{"someKey1": "stringValue"}]}

obj2 = {[{"someKey2": "stringValue"}]}
```
*Note: This does produce the Resharper 'Captured variable is modified in the outer scope' warning. So long as you call build before changing the value, the warning can be safely ignored, otherwise both results would get the same key.

## **Advanced With Statement Usages**
### **Generating a Property Value on Build**

### **Generating an `IEnumerable` Property Value on Build**

### **Setting a Property using a Different Property**
{% endraw %}