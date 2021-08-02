---
title: Side-effects in Compose
date: 2021-08-02 21:09
tags:
    - Jetpack Compose Foundation
---

# Side-effects in Compose

Learn about the different side-effect APIs Jetpack Compose offers.

> Composables **should be side-effect free**.

**Key Term:** A **side-effect** is a change to the state of the app that happens outside the scope of a composable function.

## State and effect use cases

**Key Term:** An **effect** is a composable function that doesn't emit UI and causes side effects to run when a composition completes.

> When you need to make change to the state of the app, **You should use the Effect APIs so that those side effects are executed in a predictable manner**.
>
> Make sure that the work you do in effects is UI related and doesn't break **unidirectional data flow**.

**Note:** A responsive UI is inherently asynchronous, and Jetpack Compose solves this by embracing coroutines at the API level instead of using callbacks.

### LaunchedEffect

> run suspend functions in the scope of a composable.

> To call suspend functions safely from inside a composable, use the **LaunchedEffect** composable.
>
> When **LaunchedEffect** enters the Composition, it launches a coroutine with the block of code passed as a parameter. 
>
> The coroutine will be cancelled if **LaunchedEffect** leaves the composition.
>
> If **LaunchedEffect** is recomposed with different keys, the exiting coroutine will be cancelled and the new suspend function will be launched in a new coroutine.

LaunchedEffect 用于在 Composable 中调用 suspend 方法，并将参数中的 block 放入协程执行。



```kotlin
@Composable
fun MyScreen(
    state: UiState<List<Movie>>,
    scaffoldState: ScaffoldState = rememberScaffoldState()
) {
    if (state.hasError) {
        // LaunchedEffect will cancel and re-launch if 
        // scaffoldState.snackbarHostState changes
        LaunchedEffect(scaffoldState.snackbarHostState) {
            // Show snackbar using a coroutine
            scaffoldState.snackbarHostState.showSnackbar(
               message = "Error message",
               actionLabel = "Retry message"
            )
        }
    }
    
    Scaffold(scaffoldState = scaffoldState) {
        /* ... */
    }
}
```

### rememberCoroutineScope

> Obtain a composition-aware scope to launch a coroutine outside a composable.

> As **LaunchedEffect** is a composable function, it can only be used inside other composable functions.
>
> **rememberCoroutineScope** is a composable function that returns a **CoroutineScope** bound to the point of the Composition where it's called. The scope will be cancelled when the leaves the Composition.
>
> Use **rememberCoroutineScope** whenever you need to control the lifecycle of one or more coroutines manually.

```kotlin
@Composable
fun MoviesScreen(scafoldState: ScaffoldState = rememberScaffoldState()) {
  // Creates a CoroutineScope bound to the MoviesScreen's lifecycle
  val scope = rememberCoroutineScope()
  Scaffold(scaffoldState = scaffoldState) {
    Column {
      /* ... */
      Button(
        onClick = {
          // Create a new coroutine in the event handler to show a snackbar
          scope.launch {
            scaffoldState.snackbarHostState.showSnackbar("Something.")
          }
        }
      ) {
        Text("Press me")
      }
    }
  }
}
```

### rememberUpdatedState

> reference a value in an effect that shouldn't restart if the value changes.

> It's helpful for effects that contain long-lived operations that may be expensive or prohibitive to recreate and restart. 

**rememberUpdatedState** 用于在 **effect** 中引用值，即使这个值改变也不会导致重新执行；适用于在 **effect** 中执行长时间或者禁止重新创建或者重新执行的操作。

```kotlin
@Composable
fun LandingScreen(onTimeout: () -> Unit) {
  // This will always refer to the lastest onTimeout function that the
  // LandingScreen was recomposed with
  val currentOnTimeout by rememberUpdatedState(onTimeout)
  
  // Create an effect that matches the lifecycle of LandingScreen.
  // If LandingScreen recomposes, the delay shouldn't start again.
  LaunchedEffect(true) {
    delay(SplashWaitTimeMillis)
    currentOnTimeout()
  }
  
  /* Landing screen content */
}
```

### DisposableEffect

> For side effects that need to be *cleaned up* after the keys change or if the composable  leaves the Composition.
>
> If the **DisposableEffect** keys change, the composable needs to *dispose* (do the cleanup for) its current effect, and reset by calling the effect again.

```kotlin
@Composable
fun BackHandler(backDispatcher: OnBackPressedDispatcher, onBack: () -> Unit) {
  // Safely update the current 'onBack' lambda when a new one is provided
  val currentOnBack by rememberUpdatedState(onBack)
  // Remember in Composition a back callback that calls the 'onBack' lambda 
  val backCallback = remember {
    // Always intercept back events
    object : OnBackPressedCallback(true) {
      override fun handleOnBackPressed() {
        currentOnBack()
      }
    }
  }
  
  // If 'backDispatcher' changes, dispose and reset the effect
  DisposableEffect(backDispatcher) {
    backDispatcher.addCallback(backCallback)
    // When the effect leaves the Composition, remove the callback
    onDispose {
      backCallback.remove()
    }
  }
}
```

> A **DisposableEffect** must include an **onDispose** clause.

**DisposableEffect** 用于当 composable 离开 Composition 或者它的 key 参数发生变化时，做清理工作。

### SideEffect

> To share Compose state with objects not managed by compose, use the **SideEffect** composable, as it's invoked on every successful recomposition.

```kotlin
@Composable
fun BackHandler(
  backDispatcher: OnBackPressedDispatcher,
  enabled: Boolean = true,
  onBack: () -> Unit
) {
  /* ... */
  val backCallback = remember { /* ... */ }
  
  // On every successful composition, update the callback.
  SideEffect {
    backCallback.isEnabled = enable
  }
  
  /* ... */
}
```

### produceState

> convert non-Compose state into Compose state
>
> **produceState** launches a coroutine scoped to the Composition that can push values into a returned **State**. 
>
> Event though **produceState** creates a coroutine, it can also be used to observe non-suspending sources of data. To remove the subscription to that source, use the **awaitDispose** function.

```kotlin
@Composable
fun loadNetworkImage(
  url:String,
  imageRepository: ImageRepository
) {
  // If either 'url' or 'imageRepository' changes, the running producer
  // will cancel and will be re-launched with the new keys
  return produceState(initialValue = Result.Loading, url,imageRepository) {
    val image = imageRepository.load(url)
    // This will trigger a recomposition where this State is read
    value = if (image == null) {
      Rsult.Error
    } else {
      Result.Success(image)
    }
  }
}
```

**Note:** Composables with a return type should be named the way you'd name a normal Kotlin function, starting with a lowercase letter.

> **Key Point: ** Under the hood, **produceState** makes use of other effects! It holds a **result** variable using **remember { mutableStateOf(initialValue) }**, and trigger the **producer** block in a **LaunchedEffect**. Whenever **value** is updated in the **producer** block, the **result** state is updated to the new value.
>
> You can easily create your own effects building on top of the existing APIs.

**produceState** 用于将数据转为 Compose State, 当其进入 composition 时，它会开启协程执行挂起函数 producer，并返回 State 结果。

### derivedStateOf

> Convert one or multiple state objects into another state.
>
> Use **derivedStateOf** when a certain state is calculated or derived from other state objects. Using this function guarantees that the calculation will only occur whenever one of the states used in the calculation changes.

```kotlin
@Composable
fun TodoList(
  highPriorityKeywords: List<String> = listOf("Review", "Unblock", "Compose")
) {
  val todoTasks = remember{ mutableStateListOf<String>() }
  // Calculate high priority tasks only when the todoTasks or 
  // highPriorityKeywords change, not on every recomposition.
  val highPriorityTasks by remember(todoTasks, highPriorityKeywords) {
    derivedStateOf {
      todoTasks.filter { it.containsWord(highPriorityKeywords) }
    }
  }
  
  Box(Modifier.fillMaxSize()) {
    LazyColumn {
      items(highPriorityTasks) { /* ... */ }
      items(todoTasks) { /* ... */ }
    }
  }
}
```

> An update to the state produced by **derivedStateOf** doesn't cause the composable where it's declared to recompose, Compose only recomposes those composables where its returned state is read.

### snapshotFlow

> Convert Compose's State into Flows
>
> **snapshotFlow** runs block when collected and emits the result of the **State** objects read in it.
>
> When one of the **State** objects read inside the **snapshotFlow** block mutates, the Flow will emit the new value to its collector if the new value is not equal to the previous emitted value.

```kotlin
val listState = rememberLazyListState()
LazyColumn(state = listState) {
  // ...
}

LaunchedEffect(listState) {
  snapshotFlow { listState.firstVisibleItemIndex }
  .map { index -> index > 0 }
  .distintUntilChanged()
  .filter { it == true }
  .collect { MyAnalyticsService.sendScrolledPastFirstItemEvent() }
}
```

## Restarting effects

> Some effects in Compose take a variable number of arguments, keys, that are used to cancel the running effect and start a new one with the new keys.
>
> Problems can occur if the parameters used to restart the effect are not the right ones:
>
> - Restarting effects less than they should could cause bugs in app.
> - Restarting effects more than they should could be inefficient.
>
> 
>
> 1. As a rule of thumb, mutable and immutable variables used in the effect block of code should be added as parameters to the effect composable. Apart from those, more parameters can be added to force restarting the effect.
>
> 2. If the change of a variable shouldn't cause the effect to restart, the variable should be wrapped in **rememberUpdatedState**.
>
> 3. If the variable never changes because it's wrapped in a  **remember** with no keys, you don't need to pass the variable as a key to the effect.

**Key Point:** variables used in an effect should be added as a parameter of the effect composable, or use **rememberUpdatedState**.

以 key 参数形式传入 effect 的变量，其值的变化会引起 effect 重新执行；用 **rememberUpdatedState** 包装的变量，其值变化则不会引起 effect 重新执行。

### Constants as keys

> Use a constant like true as an effect key to make it **follow the lifecycle of the call site**.

使用常量可以使得该 composable 的生命周期与被调用处的一直。