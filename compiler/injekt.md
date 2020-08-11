# Injekt: Next gen DI framework powered by a Kotlin compiler plugin

Minimal example:
```kotlin
@InitializeInjekt interface InjektInitializer
@Given fun foo(): Foo = Foo()

fun main() {
    runReader { 
        given<Foo>()
    }
}
```

What happens there:
- `@InitializeInject` indicates that this module is supported and records the file where initialization happens.
  - **NOTE:** Why is it required? Maybe transform can be stateless and rely on the information discovered on current declaration / file?
    - I see that you use it to find an `initFile` which later propagated to `lookupTracker` while generating IR. I am not sure why it is needed, as that part of compiler is not familiar to me.
    - What will happen if there are multiple initializers?
- Most of the magic happens with the functions marked with `@Reader` annotation. They have an additional context generated for each call of of `given` inside.
- `@Given` indicates a declaration, result of which will be used in the reader graph.
  - `@Given` functions are automatically `@Reader`, i.e. their parameters participate in the graph resolution.
  - Nullable `given` can be provided by non-nullable binding.
  - Same types can be distinguished with a typealias. (`ApplicationContext` - `ActivityContext`) in Android.
  - `given` calls inside this and other `@Reader` functions will be replaced with a method calls on an interface.
  - **QUESTION:** + index file with inputs?
- `runReader` is a way to transition between `@Reader` and non-`@Reader` worlds. It will take the scope it is applied on and generate the context to run the function. 

**NOTE:** on discoverability of `@Given` elements. It is quite hard to figure out currently where the element is coming from. It could be a global function, could be function in a class and so on. Maybe limit global scale or unite them in something like module?

**NOTE:** on private elements. Here most of the dependency graph is available for the user, but frequently you'd want to hide some elements of the chain (e.g. exposing view model and network layers but hide repository or such). Maybe add tools for this?

## Scoping
Lifetime scopes are defined as follows:
```kotlin
@Scoping
object MyScoping {
    private val storage = Storage()

    @Reader
    fun <T> scope(key: Any, init: () -> T) = storage.scope(key, init)
}
```

- `object` provides identifier of that scope.
  - Indicates that here we need to generate keys.
  - **NOTE**: Do you need this structure to support it? Seems a bit too magical to get right, maybe could be simplified?
- `fun scope` in conjuction with `Storage` helper make the scoping happen. Key is generated uniquely for each type / unique instance. 
  - **NOTE**: Scoping has to go through a hashmap / synchronised block every time. Maybe a bit harsh on perfomance.
  - On the other side, you don't have to generate a lot of methods as dagger does, so maybe it is more positive.

Multi instance scoping can be handled using global storage based on unique elements:
```kotlin
@Scoping
object ActivityScoping {
    private val storages = hashMapOf<Activity, Storage>()

    @Reader
    fun <T> scope(key: Any, init: () -> T) = storage.getOrPut(given<Activity>(), Storage()).scope(key, init)
}
```

**NOTE:** sсoping functionality seems to be cool, but the implementation with storage in a global hashmap is slightly confusing, although functional. I am personally a bit afraid of good old memory leaks in dynamic scopes, which is also a bit annoying to create right.

## Effects 

`@Effect` provides additional definitions for types:
```kotlin
typealias FooFactory = () -> Foo
        
@Effect
annotation class BindFooFactory {
    companion object {
        @Given
        operator fun <T : FooFactory> invoke(): FooFactory = given<T>()
    }
}

@BindFooFactory
@Reader
fun fooFactory(): Foo {
    return Foo()
}

fun invoke(): Foo { 
    return runReader { given<FooFactory>()() }
}
```
The example above contains
  - definition, `annotation class BindFooFactory`, annotated with `Effect`
  - its companion object, which defines types which it provide using bound on type parameter `T`

Annotating function with `@BindFooFactory` will now return reference to this function as in current scope.
So `runReader { given<FooFactory>()() }` will be essentially converted to `runReader { fooFactory() }`.

**NOTE:** why annotating with `@Reader` every time?

One more example of using it is to bind different types:
```kotlin
@Effect
annotation class ToString {
    companion object {
        @Given
        operator fun <T : Any> invoke(): String = given<T>().toString()
    }
}

@ToString
@Given
fun foo(): Foo = Foo()

fun invoke(): Foo { 
    return runReader { given<String>() /* Foo().toString() */ }
}
```

**NOTE:** Idea seems to bee neat, but type binding is a bit confusing. First example binds function type, whereas second only the result. The difference here is in a single annotation, maybe it could be approached in a simpler way?

## Implementation

Don't have much time to research here, but seems like it generates interfaces with `given()` types used inside the function. Obvious drawback is a number of classes generated (dexcount). It provides quite a huge in a big project.

**NOTE**: I still like it :)

Other things:
- Inputs
- Type parameters?
- Everything else I missed?