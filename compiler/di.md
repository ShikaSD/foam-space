# Kotlin compiler plugin DI proposal

Dagger is stupid, we should do better!
Koin API sets actually quite good example of how we could do that.

```kotlin
/**
 * Module contains parts of the graph
 */
val myModule = module {
  provide { MyDependency() }
  single { SomeOtherDependency() }
}

/**
 * Component provides complete graph
 * If module is complete it probably can be a component as well?
 */
val myComponent = component(module1, module2, instance(someInstance))

/**
 * Random thoughts on API here.
 */
myComponent.instance<AutoResolvedClass>()
myComponent.invoke(::functionReference)
myComponent.get()
```

## On the backend
```kotlin
// Each call which has a reference to some component, will be annotated with dependencies it requires
fun call(dep: Container)

// Converted to:
@Requires(paramName="dep", types=[KClass])
fun call(dep: Container)

/**
 * Module can be converted to realized class
 */
val myModule = module {
  provide { MyDependency() }
  single { value -> SomeOtherDependency(value) }
}

@Module
class MyModule {
  fun myDependency(): MyDependency = MyDependency()
  fun someOtherDependency(value: String): SomeOtherDependency = single(SomeOtherDependency(value))
}

/**
 * Component can be converted to realized class as well
 */
val myComponent = component(module1, module2, instance(someInstance))
```
