---
title: Managing State
date: 2021-07-30 20:36
tags: 
    - Jetpack Compose Foundation
---

# Managing State

> **State** in an app is any value that can change over time. Encompasses from Room database to a variable on a class.

## State and composition

The only way to update Compose is calling the same composable with new arguments.

>  **Composition**: a description of the UI built by Jetpack Compose when it executes composables.
>
>  **Initial composition**: creation of a Composition by running composables the first time.
>
>  **Recomposition**: re-running composables to update the Composition when data changes.

Composition: 执行 composables 构建 UI 的过程。

Initial composition: 第一次执行 composables 构建 UI 的过程。

Recomposition: 当数据变化，再次执行 composables 更新 UI 的过程。

## State in composables

> Composable function can store a single object in memory by using the **remember** composable.
>
> A value computed by **remember** is stored in the Composition during initial composition, and the stored value is returned during recomposition.
>
> **remember** can be used to store both mutable and immutable objects.

Note: **remember** stores objects in the Composition, and forgets the object when the composable that called **remember** is removed from the Composition.

- 在 Composable function 中，可以使用 **remember** composable 来保存数据到内存中；

- 在 **initial composition** 过程计算并保存的值，在 **recomposition** 过程中会被返回；所以使用 **remember** 可以保证值在 **initial composition** 和 **recomposition** 过程中保持一致。
- 当调用 **remember** 的 composable 被移除时，**remember** 保存的对象也会被释放。

> Any changes to 「value」will schedule recomposition of any composable functions that read 「value」。

Three ways to declare a **MutableState** objects in a composable:

```kotlin
// These declarations are equivalent.
val mutableState = remember { mutableStateOf(default) }
val value by remember { mutableStateOf(default) }
val (value, setValue) = remember { mutableStateOf(default) }
```

- While **remember** helps retain state across recompositions, the state is not retained across configuration changes.
- Use **rememberSaveable** to automatically save any value that can be saved in Bundle, thus the state can be retained across configuration changes. For other values, pass in a custom saver object.

## Other supported types of state

> Jetpack Compose support other observable types. Before reading another observable type in Jetpack Compose, you must convert it to a **State<T>** so that Jetpack Compose can automatically recompose when the state changes.
>
> Compose ships with functions to create **State<T>** from common observable types used in Android apps:
>
> - LiveData
> - Flow
> - RxJava2

要使得 Jetpack Compose 能够在数据改变时自动 recompose 刷新 UI，数据必须是 **State<T>**。 对于基本数据类型可通过 **MutableState<T>** 包装，至于 LiveData, Flow, RxJava 可通过内置的扩展函数转为 **State<T>**。其它自定义的 observable 可参考内置的扩展函数自定义扩展函数来转换。

### Stateful versus stateless

> A composable that uses **remember** to store an object creates internal state, making the composable stateful. 
>
> This is useful when a caller doesn't need to control the state and can use it without having to manage the state themselves.
>
> Composables with internal state tend to be less reusable and harder to test.

如果在 Composable 中使用 **remember** 保存数据就是创建内部状态，那么这个 Composable 就是有状态的了；此时调用者不必维护该状态，有内部状态的 Composable 不利于复用和测试。

> A *stateless* composable is a composable that doesn't hold any state. An easy way to achieve stateless is by using **state hoisting**.

无状态的 composable 就是不包含状态的 composable，可通过 **state hoisting** （状态上提？）实现。

> As you develop reusable composables, you often want to expose both a stateful and a stateless version of the same composable.
>
> The stateful version is convenient for caller that don't care about the state;
>
> The stateless version is necessary for callers that need to control or hoist the state.

开发可复用的 composable，往往需要对同一个 composable 提供有状态和无状态的两个版本。不关心状态的调用者使用有状态版本，需要控制或者上提状态的调用者使用状态版本。

## State hoisting

> State hoisting in Compose is a pattern of moving state to a composable's caller to make a composable stateless.

Composable 中的状态上提就是把 composable 的状态移到调用者中，这样被调用的 composable 就是无状态的了。

> The general pattern for state hoisting in Jetpack Compose is to replace the state variable with two parameters:
>
> - **value: T**: the current value to display
> - **onValueChange: (T) -> Unit**: an event that requests the value to change, where T is the proposed new value

要将一个 composable 的状态上提，一般通过两个参数实现；一个是代表当前要展示状态的值，另一个是触发状态改变的事件。

> State that is hoisted this way has some important properties:
>
> - **Single source of truth:** By moving state instead of duplicating it, we're ensuring there's only one source of truth. This helps avoid bugs.
> - **Encapsulated:** Only stateful composables will be able to modify their state. It's completely internal.
> - **Shareable:** Hoisted state can be shared with multiple composables.
> - **Interceptable:** callers to the stateless composables can decide to ignore or modify events before changing the state.
> - **Decoupled:** the state for the stateless may be stored anywhere.

```kotlin
@Composable
fun HelloScreen() {
  var name by rememberSaveable { mutableStateOf("") }
  HelloContent(name = name, onNameChange = { name = it })
}

@Composable
fun HelloContent(name: Stirng, onNameChange: (String) -> Unit) {
  Column(modifier = Modifier.padding(16.dp)) {
    Text(
      text = "Hello, $name",
      modifier = Modifier.padding(bottom = 8.dp),
      style = MaterialTheme.typography.h5
    )
    OutlinedTextField(
      value = name,
      onValueChange = onNameChange,
      label = { Text("Name") }
    )
  }
}
```

> **Unidirectional data flow:** the state goes down,and events go up.

单向数据流是指：状态由上层的 stateful composable 向下层的 stateless composable 传递，同时事件由下层的 stateless composable  向上层的 stateful composable 传递。

> Three rules to figure out where state should go:
>
> 1. State should be hoisted to at *least* the **lowest common parent** of all composables that use the state(read).
> 2. State should be hoisted to at *least* the **highest level it may be changed** (write).
> 3. If **two states changes in response to the same events** they should be **hoisted together**.

状态上提三原则：

1. 至少上提到所有读取该状态的 composable 的公共父 composable 的位置。
2. 至少上提到能修改该状态的最高层的地方。
3. 如果两个状态都响应相同的事件，那么它们应该被一起上提。

### ViewModel and state

> ViewModel are the recommended state holder for composables that are high up in the Compose UI tree or composables that are destinations in the Navigation library.
>
> Your ViewModels should expose the state in an observable holder, such as **LiveData** or **StateFlow**.
>
> You can use **LiveData** and **ViewModel** in Jetpack Compose to implement unidirectional data flow.

在 ViewModel 中，使用 LiveData 或者 StateFlow 将数据转为 observable。这样既不用处理 configuration 变化和 Activity、Fragment 生命周期变化；同时实现只需维护单一资源的值，实现各个订阅地方的自动更新和单向数据流。

```kotlin
class HelloViewModel : ViewModel() {
  // LiveData holds state which is observed by the UI
  // (state flow down from ViewMOdel)
  private val _name = MutableLiveData("")
  val name: LiveData<String> = _name
  
  // onNameChange is an event we're defining that the UI can invoke
  // (events flows down from ViewModel)
  fun onNameChange(newName: String) {
    _name.value = newName
  }
}

@Composable
fun HelloScreen(helloViewModel: HelloViewModel = viewModel()) {
  val name: String by helloViewModel.name.observeAsState("")
  HelloContent(
    name = name,
    onNameChange = { helloViewModel.onNameChange(it) }
  )
}

@Composable
fun HelloContent(name: String, onNameChange: (String) -> Unit) {
  Column(modifier = Modifier.padding(16.dp)) {
    Text(
      text = "Hello, $name",
      modifier = Modifier.padding(bottom = 8.dp),
      style = MaterialTheme.typography.h5
    )
    OutlinedTextField(
      value = name,
      onValueChange = onNameChange,
      label = { Text("Name") }
    )
  }
}
```

> Use *property delegate* syntax(**by**) to implicitly treat **State<T>** as objects of type **T** in Jetpack Compose.

## Restoring state in Compose

> Use **rememberSaveable** to restore UI state after an activity or process is recreated.
>
> **rememberSaveable** retains state across recompositions. 
>
> **rememberSaveable** also retains state across activity and process recreation.

### Ways to store state

> All data types that are added to the **Bundle** are saved automatically. If you want to save something that cannot be added to the **Bundle**, there are several options.

- **Parcelize**

  The simplest solution is to add the **@Parcelize** annotation to the object. The object becomes parcelable, and can be bundled.

  ```kotlin
  @Parcelize
  data class City(val name: String, val country: String) : Parcelable
  
  @Composable
  fun CityScreen() {
    var selectedCity = remembersaveable {
      mutableStateOf(City("Madrid", "Spain"))
    }
  }
  ```

- **MapSaver**

  You can use **mapSaver** to define your own rule for converting an objecting into a set of values that the system can save to the **Bundle**.

  ```kotlin
  data class City(val name: String, val country: String)
  
  val CitySaver = run {
    val nameKey = "Name"
    val countryKey = "Country"
    mapSaver(
      save = { mapOf(nameKey to it.name, countryKey to it.country) },
      restore = { City(it[nameKey] as String, it[countryKey] as String) }
    )
  }
  
  @Composable
  fun CityScreen() {
    var selectedCity = remeberSaveable(stateSaver = CitySaver) {
      mutableStateOf(City("Madrid", "Spain"))
    }
  }
  ```

- **ListSaver**

  To avoid needing to define the keys for the map, you can also use **listSaver** and use its indices as keys.

  ```kotlin
  data class City(val name: String, val country: String)
  
  val CitySaver = listSaver<City, Any>(
    save = { listOf(it.name, it.country) },
    restore = { City(it[0] as String, it[1] as String) }
  )
  
  @Composable
  fun CityScreen() {
    var selectedCity = rememberSaveable(stateSaver = CitySaver) {
      mutableStateOf(City("Madrid", "Spain"))
    }
  }
  ```