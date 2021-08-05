---
title: Jetpack Compose
date: 2021-07-12 22:06
tags: 
    - Jetpack Compose
---

This is my study notes about *Jetpack Compose by Tutorials FIRST EDITION* by raywenderlich.

## Section Ⅰ: Getting Started with Jetpack Compose

### Chapter 1: Developing UI in Android

> - **Design concepts** of the existing **Android UI toolkit**.
> - Basics, and how to make **custom views**.
> - Process to display the layout on screen.
> - **Principles** the current toolkit is built upon.
> - The reasoning behind these concepts, their drawbacks.
> - How they influenced the idea behind **Jetpack Compose**.
> - Compose, the next evolutionary step in Android development.

#### Unwrapping the Android UI toolkit

- Build UI as a hierarchy of **layouts** and **widgets**.
- ViewGroups are containers that control the **position** and **behavior**  of  their children.

### Chapter 2: Learning Jetpack Compose Fundamentals

> - How to write **composable functions**.
> - How to implement common composable functions.

#### Composable functions

> Any function annotated with **@Composable** (a special **annotation class**) is called a **composable function**.

```kotlin
@Composable
fun Hello(){
    // TODO
}
```

**Annotation:**

- Annotation classes simplify the code by attaching metadata to it.
- **Javac**, the java compiler uses **annotation processor** to scan an process annotations at compile time.
- This creates new source files with metadata,thus we can add behavior to classes and generate useful code, without writing a lot of boilerplate.

**Note:** Composable function can only be called by another composable function.

#### Setting the content

> *setContent()*  is The Compose way to bind the UI to an Activity or Fragment, similar to how *setContentView()* works.

```kotlin
class DemoActivity : AppCompatActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContent { Hello() }
  }
}
```

- The parent of the root composition is Recomposer which determines thread where **recomposition** happens.
- Recomposition happens every time a value changes.
- Recomposition is an event that asks the app to **re-draw** the UI with new values.

### Chapter 3: Building Layout Groups in Compose

> - Layouts in Jetpack Compose
> - Select the right layout for the UI
> - Group composable functions inside different kinds of layouts to make a more complex UI.

#### Using basic layouts in Jetpack Compose

**Layout** is the replacement for ViewGroup in Jetpack Compose.

```kotlin
@Composable inline fun Layout(
    content: @Composable () -> Unit,
    modifier: Modifier = Modifier,
    measurePolicy: MeasurePolicy
) {...}
```

- **Layout** is the main core component for layout.
- It can be used to measure and position layout children.

#### Linear layouts

**Row**

```kotlin
 Row(
        horizontalArrangement = Arrangement.SpaceAround,
        verticalAlignment = Alignment.CenterVertically,
        modifier = Modifier.fillMaxWidth()
    ) {
        Text(text = "hello", fontSize = 18.sp, modifier = Modifier.weight(1f))
        Text(text = "hi", fontSize = 18.sp, modifier = Modifier.weight(2f))
    }
```

**Column**

```kotlin
listOf("joe", "peter", "nora").forEachIndexed { index, i ->
  Text(
    text = i,
    fontSize = 18.sp,
    modifier = Modifier
      .weight(if (index % 2 == 1) 2f else 1f)
      .padding(2.dp)
      .background(color = Color.LightGray)
  )
}
```

线性布局可在子控件上设置 weight, 以确定该子控件所占父控件的空间比例；如果布局中有未设置 weight 的子控件，此时先满足该子控件的尺寸，再将剩余可用空间按其它各个子控件的 weight 计算其尺寸。

#### Using Boxes

Box is the composable counterpart for FrameLayout

```kotlin
Box(modifier = modifier.fillMaxSize()) {
    Text(
      text = stringResource(id = R.string.first),
      fontSize = 22.sp,
      modifier = Modifier.align(Alignment.TopStart)
    )
    Text(
      text = stringResource(id = R.string.second),
      fontSize = 22.sp,
      modifier = Modifier.align(Alignment.Center)
    )
    Text(
      text = stringResource(id = R.string.third),
      fontSize = 22.sp,
      modifier = Modifier.align(Alignment.BottomEnd)
    )
  }
```

**Surface**

> Material surface is the center metaphor in material design.
>
> Surface exists at a given elevation, which influences how that piece of surface visually relates to other surfaces and how that surface casts shadows.

Surface 通过设置 elevation (在 Z 轴方向的景深)，进而实现其相对周围环境不同的视觉效果，以及投射阴影；但是不适用于处理点击事件。

The surface is response for:

- Clipping: Clip its children to the specified shape.
- Elevation: Draw a shadow to represent depth, elevation represents the depth of the surface; if Android < 10 and elevation < 0, the shadow will not be drawn.
- Borders
- Background
- Content color: Surface uses **contentColor** to specify a preferred color for the content of this surface, this is used by **Text** and **Icon** component as a default color.
- Blocking touch propagation behind the surface.

```kotlin
Surface(
    modifier = modifier.size(200.dp),
    color = Color.LightGray,
    contentColor = colorResource(id = R.color.colorPrimary),
    elevation = 1.dp,
    border = BorderStroke(2.dp, Color.Red),
    shape = RoundedCornerShape(20.dp)
  ) {
    Text(text = "Jetpack Compose")
  }
```

#### Scaffold

> Scaffold implements the basic material design visual layout structure.
>
> Scaffold provide API to put together several material component to construct screen, by ensuring proper layout strategy for them and collecting necessary data so these components will work together correctly.

```kotlin
  val scaffoldState: ScaffoldState = rememberScaffoldState()
  val scope = rememberCoroutineScope()
  Scaffold(
    scaffoldState = scaffoldState,
    contentColor = colorResource(id = R.color.colorPrimary),
    content = { MyRow() },
    topBar = { MyTopAppBar(scaffoldState = scaffoldState, scope = scope) },
    bottomBar = { MyBottomAppBar() },
    drawerContent = { MyColumn() }, drawerShape = MaterialTheme.shapes.medium
  )
```

### Chapter 4: Building Lists with Jetpack Compose

- How to make lists and grids in Jetpack Compose to fit content on the screen.
- How show content that scrolls vertically and horizontally.
- How to build an alternative for the traditional RecyclerView.

#### Using vertical scrolling modifiers

> Use *Column* with extra *Modifier.verticalScroll()* to Implement a simple scrolling.

```kotlin
val scrollState = rememberScrollState()
  Column(modifier.verticalScroll(scrollState)) {
    BookImage(
      imageResId = R.drawable.advanced_architecture_android,
      contentDescriptionResId = R.string.advanced_architecture_android
    )
    BookImage(
      imageResId = R.drawable.kotlin_aprentice,
      contentDescriptionResId = R.string.kotlin_apprentice
    )
    BookImage(
      imageResId = R.drawable.kotlin_coroutines,
      contentDescriptionResId = R.string.kotlin_coroutines
    )
```

#### Using horizontal scrolling modifiers

> Using Row with extra *Modifier.horizontalScroll()* to implement a simple scrolling.

```kotlin
  val scrollState = rememberScrollState()
  Row(modifier.horizontalScroll(scrollState)) {
    BookImage(
      imageResId = R.drawable.advanced_architecture_android,
      contentDescriptionResId = R.string.advanced_architecture_android
    )
    BookImage(
      imageResId = R.drawable.kotlin_aprentice,
      contentDescriptionResId = R.string.kotlin_apprentice
    )
    BookImage(
      imageResId = R.drawable.kotlin_coroutines,
      contentDescriptionResId = R.string.kotlin_coroutines
    )
  }
```

- Scrollable columns and rows are great when we have static content, Because scrollable composables compose and render all the elements inside eagerly.
- When we have a large number of elements to display, LazyColumn and LazyRow are for use.

#### List in Compose

**Lazy loading:** loading data only when it's needed.

##### Introducing LazyColumn & LazyRow

> - The framework composes only the elements that it should show on the screen.
> - When you scroll, new elements are composed and the old ones are disposed of.
> - When scroll back, the old elements are recomposed.
> - Jetpack Compose's recomposition handles caching efficiently.

##### Creating list with LazyColumn & LazyRow

```kotlin
@Composable
fun LazyColumn(
    modifier: Modifier = Modifier,
    state: LazyListState = rememberLazyListState(),
    contentPadding: PaddingValues = PaddingValues(0.dp),
    reverseLayout: Boolean = false,
    verticalArrangement: Arrangement.Vertical =
        if (!reverseLayout) Arrangement.Top else Arrangement.Bottom,
    horizontalAlignment: Alignment.Horizontal = Alignment.Start,
    flingBehavior: FlingBehavior = ScrollableDefaults.flingBehavior(),
    content: LazyListScope.() -> Unit
)

@Composable
fun LazyRow(
    modifier: Modifier = Modifier,
    state: LazyListState = rememberLazyListState(),
    contentPadding: PaddingValues = PaddingValues(0.dp),
    reverseLayout: Boolean = false,
    horizontalArrangement: Arrangement.Horizontal =
        if (!reverseLayout) Arrangement.Start else Arrangement.End,
    verticalAlignment: Alignment.Vertical = Alignment.Top,
    flingBehavior: FlingBehavior = ScrollableDefaults.flingBehavior(),
    content: LazyListScope.() -> Unit
)
```

#### Grids in Compose

## Section Ⅱ: Composing User Interfaces

- Attach LiveData structures to state management.
- Rely on different styling modifiers.
- Create a powerful UI.

### Chapter 5: Combining Composables

Jetpack Compose in action:

- How to **think** about UI design when building with Jetpack Compose.
- How to **combine** basic composables to create **complex UI**.

### Chapter 6: Using Compose Modifiers

- How to **style** composables using  **modifiers**.

### Chapter 7: Managing State in Compose

- What **State** is.
- What **unidirectional data flow** is.
- How to think about **state** and **events**  when creating **stateless** composables.
- How to use ViewModel and LiveData to manage state in Compose.

#### Understanding state

> **State** is any **value** that **can change over time**.

**Unidirectional data flow** is a design where **state flow down and events flow up**.

#### Compose & ViewModel

> - A ViewModel extra the **state** from the **UI** and define **events** that the UI can call to update that state.
> - LiveData allows you to crate **observable state holders** that provide a way to observe changes to the state.

**Stateful vs stateless**

- A **stateful**  composable would be a composable that has a dependency on the final class, which can directly change a specific state.

- A **stateless** composable is a composable that cannot change any state itself.

> **State Hoisting:** It's a programming pattern where you **move state to the caller of a composable** by replacing internal state in a composable with **a parameter** and **events**.

For composables, this often means introducing two parameters to the composable:

- **value: T**  The current value to display.
- **onValueChange:(T) -> Unit**  An event that requests a change to a value, where T is the proposed new value.

By applying **state hoisting** to a composable, you make it stateless, which means it can't change any state itself.

Whenever you extra a stateless composable, keep two things in mind:

- The **state** you're passing down.
- The **event** you're passing up.

### Chapter 8: Applying Material Design to Compose

- How to use **Material Design composables**.
- Go over **state management**.
- **Material theming**

#### Using remember

```kotlin
@Composable
inline fun <T> remember(calculation: @DisallowComposableCalls () -> T): T
```

- Remember the value produced by *calculation*, *calculation*  will only be evaluated during the composition.
- Recomposition will always return the value produced by composition.

With remember(), Compose lets you **store values in the composition tree**.

Handle how app behaves if the user presses the system back button.

```kotlin
BackHandler(onBack = {...})
```

- BackHandler is an effect for handling press of the system back button; 
- Using theOnBackPressedDispatcherOwner and providing it through an **Ambient**, you gain access to system back button handling.
- Jetpack Compose offers a Material Design implementation that allows you to theme your app by specifying the **color palette, typography** and **shapes**.

## Section Ⅲ: Building Complex Apps with Jetpack Compose

> Building **complex custom components** and applying **animations** to represent state changes when users interact with the UI.

- Connect Compose UI to legacy Android code.
- React to Compose UI lifecycles.
- Animate different state changes and user interactions.

### Chapter 9: Using ConstraintSets in Composables

- Learn how ConstraintLayout works in Jetpack Compose.
- Implement layouts using **constraint sets**.

ConstraintLayout arranges elements **relative to one another**.

#### ConstraintLayout in Jetpack Compose



```kotlin
@Composable
inline fun ConstraintLayout(
    modifier: Modifier = Modifier,
    optimizationLevel: Int = Optimizer.OPTIMIZATION_STANDARD,
    crossinline content: @Composable ConstraintLayoutScope.() -> Unit
)
```

**Note:** To use ConstraintLayout modifiers in referenced composables, pass ConstraintLayoutScope as a parameter.

#### Advanced features of ConstraintLayout

**Guidelines**

> A **guideline** is an invisible object working as a helper tool.
>
> Guideline can be created from any side of  the screen.

Two way to set guideline offset:

- specify the fixed amount of dp .
- Give **screen percentage**.

```kotlin
val startVerticalGuideline = createGuidelineFromStart(0.2f)
val absoluteVerticalGuideline = createGuidelineFromAbsoluteLeft(20.dp)
...
```

**Barriers**

> A barrier is an element that can contain multiple constrain references.

```kotlin
val startBarrier = createStartBarrier()
```

**Chains**

> A  **chain** allows you to reference multiple elements that constrained to each other.

Three types of ChainStyles:

- **Packed:** All elements are packed in a group,.
- **Spread:** All elements are spread evenly from each other and the edges.
- **SpreadInside:** All the elements are spread evenly from each other but start from the edge.

```kotlin
createVerticalChain(element1,element2,chainStyle = ChainStyle.Spread)
```

### Chapter 10: Building Complex UI in Jetpack Compose

- Build app by first implementing the most basic components.
- Use a component-based approach to extract repeat parts into separate composables.
- Use Preview for each of the components until built the whole screen.
- Use Preview as separate composable if component has arguments, to avoid making custom classes for PreviewParameters.
- Use zIndex() if multiple composables overlap and want to change their order of display in the z-orientation.

### Chapter 11: Reacting to Compose Lifecycle

> Jetpack Compose offers a list of events that can trigger at specific points in the lifecycle, called **effects**.

- Learn how to react to the lifecycle of composable functions.
- Execute code at specific moments while composable is active.

```kotlin
val scope = rememberCoroutineScope()
```

- rememberCoroutineScope() is a SuspendingEffect.
- It creates a CoroutineScope, which is bound to the composition.
- CoroutineScope is only created once, and stays the same even after recomposition.
- Any Job belonging to this scope will be canceled when the scope leaves the composition.

#### Effects in Compose

> Side effects are operations that change the values of anything outside the scope of the function.

The biggest problem with side effects is that you don't have control over when they actually occur. This is problematic in composables because the code inside them executes every time a recomposition takes place.

Effects can help by giving control over when the code executes.

#### SideEffect

> SideEffect() ensures event only executes when a composition is successful. Use it when you want event to run with every recomposition.

```kotlin
@Composable
fun MainScreen(router: Router){
  val drawerState = rememberDrawerState(DrawerValue.Closed)
  SideEffect {
    router.isRoutingEnabled = drawerState.Closed
  }
}
```

#### LaunchEffect

> LauchedEffect launches a coroutine into the composition's CoroutimeScope. Just like rememberCoroutineScope(), its coroutines is canceled when LaunchedEffect leaves the composition and will relaunch on recomposition.

```kotlin
@Composable
fun SpeakerList(searchText: String) {  
    var communites by remember {mutableStateOf(emptyList<String>())}  
    LaunchedEffect(searchText){    
        communities = viewModel.searchCommunities(searchText)  
    }  
    Communities(comunities)
}
```

- LaunchedEffect initiates the first time it enters the composition and every time the parameter changes. It cancels all running Jobs during the parameter change or upon leaving the composition.

**RememberUpdatedState()**

> remember a mutableStateOf and update its value to newValue on each recomposition of the rememberUpdatedState call.

#### produceState

> Return an observable snapshot State that produces values over time.
>
> producer is launched when produceState enters the composition and is cancelled when produceState leaves the composition.
>
> producer should use ProduceStateScope.value to set new values on the returned State.
>
> The returned State conflates values; no change will be observable if ProduceStateScope.value is used to set a value that is equal to its old value, and observers may only see the lastest value if several values are set in rapid succession.

```kotlin
@Composable
fun <T> produceState(
    initialValue: T,
    @BuilderInference producer: suspend ProduceStateScope<T>.() -> Unit
): State<T>
```



#### Migrate effects

> Using LaunchedEffect, DisposableEffect and SideEffect to replace the removed effects.

Replace onActive() without subject parameter by using LaunchedEffect with a constant value like  Unit or true.

```kotlin
onActive {
  someFunction()
}

// ==>
LaunchedEffect(Unit){
  someFunction()
}
```

If onActive() with subject parameter, replace it with LaunchedEffect.

``` kotlin
onActive(parameter) {
  someFunction()
}
// ==>
LaunchedEffect(parameter) {
  someFunction()
}
```

Replace onCommit() without a subject parameter by using SideEffect with a constant value like Unit or true.

```kotlin
onCommit() {
  someFunction()
}
// ==>
SideEffect {
  someFunction()
}
```

#### Key points

- Use **rememberCoroutineScope()** when you are using coroutines and need to cancel and relaunch the coroutine after an event.
- User **LaunchEffect()** when you are using coroutines and need to cancel and relaunch the coroutine every time the parameter changes and it isn't stored in a mutable state.
- **DisposableEffect()** is useful when you aren't using coroutines and need to dispose and relaunch the event every time your parameter changes.
- **SideEffect()** triggers an event only when the composition is successful and you don't need to disposable the subject.
- Use **rememberUpdatedState()** when you want to launch your effect only once but still be able to update the values.
- Use **produceState()** to directly convert non-composable states into composable states.
- **Names of the composables with a return type shold start with the lowercase letter**.

### Chapter 12: Animating Properties Using Compose

- **Animate composable properties** using animate*AsState().
- Use updateTransition() to **animate multiple properties** of composables.
- **Animate composable content**.
- **Implement an animated button**
- **Implement an animated toast**.

**Transition**: It manages one or more animations as its children and run them simultaneously between multiple states.

**Key points**

- Use animate*AsState() for **fire-and-forget** animations targeting **single properties** of your composables. This is very useful for animating size, color, alpha and similar simple properties.
- Use **Transition** and **updateTransition()** for **state-based** transitions.
- Use **Transitions** when you have to animate **multiple properties** of composables, or when you have multiple states between which you can animate.
- **Transitions** are very good when showing content for the first time or leaving the screen, menu, option pickers and similar. They are also great when animating between multiple states when filling in forms, selecting options and pressing buttons!
- Use **AnimatedVisibility()** when you want to animate the appearance and disapearance of composable content.
- **AnimatedVisibility()** lets you combine different types of visibility animations and lets you define directions if you use predefined transition animations.

### Chapter 13: Adding View Compatibility

> Basic principles of **combining** Jetpack Compose and the old View framework.

#### 在传统 View 布局中加入 Compose

- 在 XML 布局文件中使用 **ComposeView** 占位要用 composable 的地方。
- 在代码中通过 ID 获取到对应的 **ComposeView** 后，通过 setContent() 填充进 Composable 即可。

```xml
 <androidx.compose.ui.platform.ComposeView
      android:id="@+id/composeButton"
      android:layout_width="wrap_content"
      android:layout_height="48dp"
      android:layout_marginTop="16dp"
      app:layout_constraintEnd_toEndOf="parent"
      app:layout_constraintStart_toStartOf="parent"
      app:layout_constraintTop_toBottomOf="@+id/subtitle" />
```

```kotlin
binding.composeButton.setContent {
  MaterialTheme {
    ComposeButton { showToast() }
  }
}
```

#### 在 Compose 中加入 传统 View

- 使用 **AndroidView** 可以将传统 View 添加到 Compose。
- 通过在 **AndroidView** 中的 **factory**  参数中传入传统 View 对象实现。

```kotlin
@Composable
private fun TrendingTopic(trendingTopic: TrendTopicModel) {
  AndroidView({ context ->
    // TrendingTopicView 是一个自定义 View，其中通过 inflate 加载 XML 布局。
    TrendingTopicView(context).apply {
      text = trendingTopic.text
      image = trendingTopic.imageRes
    }
  })
}
```