# Bloc / cubit architecture

## Cubit for ya
```kotlin
abstract class Cubit<State> {
  val state = BehaviourRelay.create<State>()

  fun emit(newState: State) {
    state.accept(newState)
  }
}

class CounterCubit : Cubit<Int> {
  fun increment() {
    emit(state.value + 1)
  }

  fun decrement() {
    emit(state.value - 1)
  }
}

cubit.increment()
cubit.decrement()
```

## Bloc
```kotlin
abstract class Bloc<Event, State> : Cubit<State> {
  val events = PublishRelay.create<Event>()

  init {
    events.flatMap(eventToState).subscribe(state)
  }

  fun accept(event: Event)
    events.accept(event)
  }

  abstract fun eventToState(event: Event): Observable<State>
}


sealed class CounterEvent {
  object Increment : CounterEvent()
  object Decrement : CounterEvent()
}

class CounterBloc : Bloc<CounterEvent, Int> {
  fun eventToState(event: CounterEvent): Observable<Int> =
    when (event) {
      is Increment -> Observable.just(state.value + 1)
      is Decrement -> Observable.just(state.value - 1)
    }
}
```
