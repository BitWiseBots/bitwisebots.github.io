---
title: "Quick-Start Guide"
permalink: /fluent-builders/quick-start
---

### Simple DTO type
* The example type to be built:
    ```csharp
    public class User
    {
        public string FirstName { get; set;}
        public string LastName { get; set; }
        public int Age { get; set; }
    }
    ```
* Building an instance:
    ```csharp
    BuilderFactory.Create<User>()
        .With(u => u.FirstName, "John")
        .With(u => u.LastName, "Doe")
        .Build();
    ```

### Immutable types
* The example type to be built:
    ```csharp
    public class User
    {
        public User(string firstName, string lastName, int age)
        {
            FirstName = firstName;
            LastName = lastName;
            Age = age;
        }

        public string FirstName { get; }
        public string LastName { get; }
        public int Age { get; }
    }
    ```
* Building an instance:
    ```csharp
    BuilderFactory.Create<User>(b => 
        new User(
            b.From(u => u.FirstName), 
            b.From(u => u.LastName), 
            b.From(u => u.Age))))
        .With(u => u.FirstName, "John")
        .With(u => u.LastName, "Doe")
        .Build();
    ```

## The Config Store
The config store allows you to to setup some default configurations that will be used whenever a builder for a matching type is used.

Three different types of configurations are available:

1. Constructor
    * Defines a function to be used to construct the specified type by default; this allows for the `Create` overload shown in the Immutable type section above to be omitted.
2. Post Build
    * Defines an action that is called after running through the `Build` but before returning the built object; this allows for things like wiring up Id properties on something like an ORM object graph from the navigational properties.
3. Type Default
    * Defines an action that is called when setting a value for the specific type and the `With` calls does not provide an explicit value.

### Baseline Builder Config
* Creating an instance of the builder config:
    ```csharp
    public class SampleBuilderConfig : BuilderConfig
    {
        public SampleBuilderConfig()
        {
            /* All Constructor, Post Build, and Type default configs are defined here */
        }
    }
    ```
* Initializing the configurations for all tests:
    ```csharp
    [SetUpFixture]
    public class AllTestsFixture
    {
        [OneTimeSetUp]
        public void OneTimeSetUp()
        {
            BuilderFactory.AddConfig<SampleBuilderConfig>();
        }
    }
    ```

### Constructor Configuration
* The example type to build:
    ```csharp
    public class User
    {
        public User(string firstName, string lastName, int age)
        {
            FirstName = firstName;
            LastName = lastName;
            Age = age;
        }

        public string FirstName { get; }
        public string LastName { get; }
        public int Age { get; }
    }
    ```
* Creating the configuration:
    ```csharp
    public class SampleBuilderConfig : BuilderConfig
    {
        public SampleBuilderConfig()
        {
            AddConstructor<User>(b => 
                new User(
                    b.From(u => u.FirstName),
                    b.From(u => u.lastName),
                    b.From(u => u.Age)));
        }
    }
    ```
*  With this configuration everytime you ask the `Builders` for a `User` it will automatically call the constructor and will use the values provided in matching `With` calls; if no value was provided it will use `default(T)` instead.

### Running Addtional Setup on Object Graphs
* The example type to build:
    ```csharp
    public class User
    {
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public int Age { get; set; }

        public Guid Id { get;set; }
        public User Supervisor { get; set; }
        public Guid SupervisorId { get; set; }
    }
    ```
* Creating a post build config:
    ```csharp
    public class SampleBuilderConfig : BuilderConfig
    {
        public SampleBuilderConfig()
        {
            AddPostBuild<User>(u => {
                if(u.Supervisor != null)
                {
                    u.SupervisorId = u.Supervisor.Id
                };
            });
        }
    }
    ```
* This kind of configuration is great if your application uses an ORM and has both logic that only looks at a relationship's Id property or that is more involved and requires the full navigational property.

### Specifying Default Values for Specific Types
* The example type to build:
    ```csharp
    public class User
    {
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public int Age { get; set; }

        public Guid Id { get;set; }
    }
    ```
* Creating a type default config:
    ```csharp
    public class SampleBuilderConfig : BuilderConfig
    {
        public SampleBuilderConfig()
        {
            AddTypeDefault<Guid>(() => Guid.NewGuid());
        }
    }
    ```
* With this configuration if a `With` statement provides a value of `default(Guid)` or if the type was immutable, had a stored constructor, and did not have a corresponding `With` call, then the function above would automatically be used to generate a value.