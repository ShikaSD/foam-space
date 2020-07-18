# Thoughts on compose architecture

Majority of the apps I have seen rely on flow or live data being passed in as a function parameter.

I am a bit surprised by this, but at the same time, compose is most effective when you have specific points for observation.

Points to think about:
1. How do you pass dependencies inside a function?
   
   Imagine the following case:
  ```kotlin
  fun myScreen(externalSignals: Flow<Something>) {
    myScreenLogic(externalSignals).state { state ->
      // View definition based on state
    }
  }
  ```
  - How do you scale a huge number of dependencies properly?
  - How do you propagate them through outer layers?
  - How do you test all of this?

2. How do you structure logic/layering in a concise template?
3. How do you define a boundary of component and create a independent one?
