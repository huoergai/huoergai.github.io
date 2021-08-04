---
title: CompositionLocal
date: 2021-08-04 11:25
tags:
  - Jetpack Compose Foundation
---

# Locally scoped data with CompositionLocal

> **CompositionLocal** is tool for passing data down through the Composition implicitly.

## Introducing CompositionLocal

> Usually in Compose, **data flow down** through the UI tree as parameters to each composable function. This makes a composable's dependencies explicitly. This can however be cumbersome for data that is very frequently and widely used such as colors or type styles.
>
> To support not needing to pass data as an explicit parameter dependency to most composables, **Compose offers CompositionLocal which allows you to create tree-scoped named objects that can be used as an implicit way to have data flow through the UI tree.**
>
> **CompositionLocal** elements are usually provided with a value in a certain node of the UI tee. That value can be used by its descendants without declaring the **CompositionLocal** as a parameter in the composable function.

在 Compose 中，通常数据是沿着 UI 树向下传递到各个 composable的。这中显示的依赖对于频繁或者使用较广的数据，就变得有点麻烦。

**CompositionLocal** 就是为了解决这种问题，它可以使数据不必通过显示的参数依赖传递到大多数 composable。

**Compose 通过提供 CompositionLocal 在 UI 树中传递数据，通过 CompositionLocal 可以创建 tree-scoped 的命名对象，以非显示方式在 UI tree 中传递数据**。在 UI 树的节点提供带值的CompositionLocal 元素，这个值子节点可以不申明 CompositionLocal 参数就可以使用。

**Key terms:** **The Composition** is the record of the call graph of composable functions. The **UI tree** or **UI hierarchy** is the tree of **LayoutNode** constructed, updated, and maintained by the composition process.

```kotlin
@Composable
fun MyApp() {
  MaterialTheme {
    // New values for colors, typography, and shapes are available in 
    // MaterialTheme's cotent lambda
    
    // ... content here ...
    
  }
}

// Some composable deep in the hierarchy of MaterialTheme
@Composable
fun SomeTextLabel(labelText: String) {
  Text(
    text = labelText,
    // 'primary' is obtained from MaterialTheme's LocalColors CompositionLocal
    color = MaterialTheme.colors.primary
  )
}
```

> A **CompositionLocal instance is scoped to a part of the Composition**  so you can provide different values at different levels of the tree. The **current** value of a CompositionLocal corresponds to the closest value provided by an ancestor in that part of the Composition.
>
> **To provide a new value to a CompositionLocal, use the CompositionLocalProvider** and its **provides** infix function that associates a CompositionLocal key to a value.
>
> The content lambda of the CompositionLocalProvider will get the provided value when accessing the current property of the CompositionLocal. 
>
> When a new value is provided, Compose recomposes parts of the Composition that read the CompositionLocal.

因为 CompositionLocal 实例限定在 Composition 的部分范围，所以可以在 UI 树的不同层级提供不同的值。CompositionLocal 的 **current** 值是与就近的上一级关联的。

 使用 **CompositionLocalProvider** 和 **provides** infix 函数就可以将一个 CompositionLocal 的键值关联起来。在 content 中通过 CompositonLocal 的 current 属性获取对应的值，当有新值时，Compose 会 recompose 读取这个 CompositionLocal 的 Composition。

```kotlin
@Composable
fun CompositionLocalExample() {
  // MaterialTheme sets the ContentAlpha.high as default
  MaterialTheme {
    Column {
      Text("Use MaterialTheme's provided alpha")
      CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
        Text("Medium value provided for LocalContentAlpha")
        Text("This text also uses the medium value")
        CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.disabled) {
          DescendantExample()          
        }
      }
    }
  }
}

@Composable
fun DescendantExample() {
  // CompositionLocalProviders also work across composable functions
  Text("This Text uses the disabled alpha now")
}

@Composable
fun FruitText(fruitSize: Int) {
  // Get 'resource' from current value of LocalContext
  val resource = LocalContext.current.resources
  val fruitText = remember(resources, fruitSize) {
    resources.getQuantityString(R.pluras.fruit_title, fruitSize)
  }
  Text(text = fruitText)
}
```

**Note: CompositionLocal** objects or constants are usually prefixed with **Local** to allow better discoverability with auto-complete in the IDE.

## Creating your own CompositionLocal

> **Another key signal for using CompositionLocal when the parameter is cross-cutting and intermediate layers of implementation should not be aware it exists** 
>
> CompositionLocal downsides:
>
> - CompositionLocal make a composable's behavior harder to reason about.
> - Debugging the app when a problem occurs can be more challenging.

### Deciding whether to use CompositionLocal

- A **CompositionLocal should have a default value**.
- **Avoid CompositionLocal for concepts that aren't thought as tree-scoped or sub-hierarchy scoped**.

### Create a CompositionLocal

Two APIs to create a CompositionLocal:

- **compositionLocalOf**:  Changing the value provided during recomposition invalidates only the content that reads its **current** value.
- **staticCompositionLocalOf**: Reads of a staticCompositionLocalOf are not tracked by Compose. Changing the value causes the entirety of the content lambda where the CompositionLocal is provided to be recomposed.

If the value provided to the CompositionLocal is highly unlikely to change or will never change, use staticCompositionLocalOf to get performance benefits.

```kotlin
// LocalElevations.kt file
data class Elevations(val card: Dp = 0.dp, val default: Dp = 0.dp)
// Define a CompositionLocal global object with a default
// This instance can be accessed by all composables in the app
val LocalElevations = compositionLocalOf { Elevations() }
```

### Providing values to a CompositionLocal

**The CompositionLocalProvider composable binds values to CompositionLocal instances for the given hierarchy**. 

```kotlin
// MyActivity.kt file
class MyActivity : ComponentActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContent {
      // Calculate elevations based on the system theme
      val elevations = if (isSystemInDarkTheme()) {
        Elevations(card = 1.dp, default = 1.dp)
      } else {
        Elevations(card = 0.dp, default = 0.dp)
      }
      // Bind elevation as the value for LocalElevations
      CompositionLocalProvider(LocalElevations provides elevations) {
        // ... Content goes here ...
        // This part of Composition will see the 'elevations' instance
        // when accessing LocalElevations.current
      }
    }
  }
}
```

### Consuming the CompositionLocal

> **CompositionLocal.current** returns the value provided by the nearest CompositionLocalProvider that provides a value to that CompositionLocal.

```kotlin
@Composable
fun SomeComposable() {
  // Access the gloablly defined LocalElevations variable to get the
  // current Elevations in this part of the Composition
  Card(elevation = LocalElevations.current.card) {
    // Content
  }
}
```

## Alternatives to consider

### Pass explicit parameters

Being explicit about composable's dependencies is a good habit. We recommend that you **pass composables only what they need**. 

 

```kotlin
@Composable
fun MyComposable(myViewModel: MyViewModel = viewModel()) {
  // ...
  MyDescendant(myViewModel.data)
}

// Don't pass the whole object! Just what the descendant needs.
// Also, don't pass the ViewModel as an implicit dependency using a CompositionLocal.
@Composable
fun MyDescendant(myViewModel: MyViewModel) { ... }
// Pass only what the descendant needs
@Composable
fun MyDescendant(data: DataToDisplay) {
  // Display data
}
```

### Inversion of control

Another way to avoid passing unnecessary dependencies to composable is via *inversion of control*. Instead of the descendant taking in a dependency to execute some logic, the parent does that instead.

```kotlin
@Composable
fun MyComposable(myViewModel: MyViewModel = viewModel()) {
  // ...
  MyDescendant(myViewModel)
}

@Composable
fun MyDescendant(myViewModel: MyViewModel) {
  Button(onClick = { myViewModel.loadData() }) {
    Text("Load data")
  }
}

@Composable
fun MyComposable(myViewModel: MyViewModel = viewModel()) {
  // ...
  ResuableLoadDataButton(
    onLoadClick = { myViewModel.loadData() }
  )
}

@Composable
fun ResuableLoadDataButton(onLoadClick: () -> Unit) {
  Button(onClick = onLoadClick) {
    Text("Load data")
  }
}
```