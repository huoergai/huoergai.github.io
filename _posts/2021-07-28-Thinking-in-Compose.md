---
title: Thinking in Compose
date: 2021-07-28 22:25
tags: 
    - Jetpack Compose Foundation
---

# Thinking in Compose

This is my study notes about **Jetpack Compose's** official doc, the original reference is [Thinking in Compose](https://developer.android.com/jetpack/compose/mental-model)

> Jetpack Compose is a modern **declarative UI** Toolkit for Android.
>
> Compose make it easier to write and maintain your app UI by providing a *declarative API* that allows you to render your app UI without imperatively mutating frontend views.

Jetpack Compose 是 Android 的一个现代化『声明式』UI 工具箱。通过『声明式』API 可以不用『命令式』的改变前端 View 来渲染应用的 UI，使得创建和维护应用 UI 更容易。

## The declarative programming paradigm

- 在传统的 View 中，当因为用户交互事件 state 改变的时候，需要调用方法来更新 UI ；此时会修改 widget 的内部状态（internal state）。
- 手动操作 View 提高了出错的机率，因为需要手动维护数据和表现的一致性，尤其当一个数据在多个地方被展示时。
- 整个工业界都在逐渐向 declarative UI model 转移；通过重建整个 screen，并只应用必要的修改，进而避免了复杂的手动更新”有状态的“ view hierarchy；因此简化了创建和更新 UI 相关的工作。
- 考虑到时间、算力和电量，重建整个 screen 可能开销昂贵；Compose 通过智能的选择在给定的之间点重绘哪个部分的 UI 来减轻开销；因此开发过程中 **Recomposition** 需要特别关注。

## A simple composable function

```kotlin
@Composable
fun Greeting(name: String) {
  Text("Hello $name")
}
```

- All Composable functions must have @Composable annotation;
- @Composable annotation informs the Compose compiler that this function is intended to convert data into UI.
- Composable functions can accept parameters, which allow the app logic to describe the UI.
- Composable functions emit UI hierarchy by calling other composable functions.
- Compose functions that emit UI do not need to return anything, because they describe screen state instead of constructing UI widgets.

## The declarative paradigm shift

- In Compose's declarative approach, widgets are relatively stateless and do not expose setter or getter functions.
- In fact, widgets are not exposed as objects. You update the UI by calling the same composable function with different arguments.

## Dynamic content

> Because composable functions are written in Kotlin instead of XML, they can be as dynamic as any other Kotlin code.

```kotlin
@Composable
fun Greeting(names: List<String>) {
  for (name in names) {
    Text("Hello $name")
  }
}
```

## Recomposition

- In Compose, you call the composable function again with new data to change a widget, Doing so causes the function to be **recomposed**--the widgets emitted by the function are  redrawn, if necessary, with new data.
- The Compose framework can intelligently recompose only the components that changeed.

```kotlin
@Composable
fun ClickCounter(clicks: Int, onClick: () -> Unit) {
  Button(onClick = onClick) {
    Text("I've been clicked $clicks times")
  }
}
```

> **Recomposition** is the process of calling composable functions again when inputs changes.

When Compose recompose based on new inputs, it only calls the functions or lambdas might have changed, and skips the rest.

> A **side-effect** is any change that is visible to the rest of your app.

For example:

- Writing to a property of a shared object.
- Updating an observable in ViewModel
- Updating shared preferences

Never depend on side-effect from executing composable functions, since a function's recomposition may be skipped.

- Composable functions might be re-executed as often as every frame, such as when an animation is being rendered.
- Composable functions should be fast to avoid jank during animations.

**A number of things to aware of when program in Compose:**

- Composable functions can execute in any order.
- Composable functions can execute in parallel.
- Recomposition skips as many composable functions and lambdas as possible.
- Recomposition is optimistic and may be canceled.
- A Composable function might be run quite frequently, as often as every frame of an animation.

The best practice is keep composable functions fast, idempotent, and side-effect free.

### Composable functions can execute in any order

> If a composable function contains call to other composable functions, those functions might run in any order. Compose has the option of recognizing some UI elements are higher priority than others, and drawing them first.

### Composable functions can run in parallel

> Compose optimize recomposition by running composable functions in parallel. This lets Compose take advantage of multiple cores. 
>
> This optimization means a composable function might execute within a pool of background threads.
>
> To ensure application behaves correctly, all composable functions should have no side-effects. Instead, trigger side-effects from callbacks such as onClick that always execute on the UI thread.
>
> When a composable function is invoked, the invocation might occur on a different thread from the caller. That means code that modifies variables in a composable lambda should be avoided-both because such code is not thread-safe, and because it is an impermissible side-effect of the composable lambda.

Compose function 可以并行执行，即一个 Compose function 中的多个 Compose function 可能在一个后台线程池中分别被执行；由此应该避免在 composable lambda 中修改 变量，因为这是线程不安全的，也是不允许的。

因此应该通过将 **side-effects** 放到回调中执行，以 尽量确保 composable 都是没有 **side-effects** 的。编写 Composable 时应该将 state  向下传递，event 向上传递，Composable 本身只负责渲染；此时 Composable 就是 stateless 的、幂等的。

### Recomposition skips as much as possible

> When portions of UI are invalid, Composable does its best to recompose just the portions that need to be update.

### Recomposition is optimistic

> Compose expects to finish recomposition before the parameters changes again. If a parameter does change before recomposition finishes, Compose might cancel the recomposition and restart it with the new parameter.
>
> When recomposition is canceled, Compose discards the UI tree from the recomposition. But the side-effect, depend on the UI being displayed, will be applied even if composition is canceled. This can lead to inconsistent app state.

在新的修改发生时，如果之前的 recomposition 没有执行完成，那么它将会被取消并依据新的修改重新执行。但是如果有 side-effect，这些 side-effect 依然会被执行，即使 composition 被取消。

### Composable functions might run quite frequently

> A composable function might run for every frame of a UI animation. If the function expensive operations, the function can cause UI jank.
>
> If composable function needs data, it should define parameters for the data.