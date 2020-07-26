# Kotlin compiler plugin DI proposal

Following example of Dagger of compile-time safe DI.
Goals:
- Better syntax
- More possibilities
- Less complexity

## Definition
DI container can be expressed in terms of types it provides.
Considering container that can provide `Int`, `String` and `(Int) -> String`, we can express it as `Container<Int, String, (Int) -> String>`.
Order of type parameters is not important, e.g. `Container<A, B>` == `Container<B, A>`.

Containers are created dynamically from modules, which define parts of a graph. User can control which types and values are accessible + scoping of these values.
`Container<A, B>` can be created from `ModuleA exposing A` and `ModuleB exposing B`. Essentially, `Container<A, B>` is `sum(ModuleA, ModuleB)`.

To retrieve dependency from container, one can use `Container.get<T>()` function. This allows to use different containers which can provide this type.
```kotlin
fun foo(container: Container) {
  container.get<Bar>()
}

val container1 // Container<Int, Bar>
val container2 // Container<String, Bar>

foo(container1) // works, container1 can provide Bar
foo(container2) // works, container2 can provide Bar as well
```

It can also be extended further to allow assigning different containers to the same variable, where resulting type will be an intersection of others:
```kotlin
val container1 // Container<Int, Bar>
val container2 // Container<String, Bar>
val container3 = if (something) container1 else container2 // Container<Bar>
```
**TODO:** Support unsafe cast
**TODO:** Support referencing container type

## Syntax example

```kotlin
/**
 * Module contains parts of the graph
 */
val myModule = module {
  provide { MyDependency() }
  single { SomeOtherDependency() }
}

// or
class MyModule(val value: String) : Module {
  val myDependency: MyDependency = MyDependency()
  val someOtherDependency: SomeOtherDependency by lazy { SomeOtherDependency(value) }
  val recreatedDep: RecreatedDependency get() = RecreatedDependency(myDependency)
  private val invisibleDep: ...
}

/**
 * Component provides complete graph
 * TODO: If module is complete it probably can be a component as well?
 */
val myComponent = component(module1, module2, instance(someInstance))

/**
 * Random thoughts on API here.
 */
myComponent.instance<AutoResolvedClass>()
myComponent.invoke(::functionReference)
myComponent.get()
```

## Implementation
Compiler plugin yo

Every field and property of type `Container` should keep associated types with it. There's no need to implement it as actual generic type, every value can bring annotations with it.
```kotlin
annotation class Provides(vararg types: KClass)

fun foo(container: @Provides(Bar::class) Container) {
  container.get<Bar>()
}

val container1: @Provides(Int::class, Bar::class) Container
val container2: @Provides(String::class, Bar::class) Container>
```

Module definitions made with DSL need to be expanded to classes similar to class syntax.
```kotlin
val myModule = module {
  provide { MyDependency() }
  single { SomeOtherDependency() }
}

// expanded to
val myModule = MyModule()
class MyModule : Module {
  val myDependency: MyDependency get() = MyDependency()
  val someOtherDependency: SomeOtherDependency by lazy { SomeOtherDependency() }
}
```

**TODO:** implementation of containers (combining interfaces based on modules?)
