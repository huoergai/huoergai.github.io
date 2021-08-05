---
title: Lifecycle of Composables
date: 2021-07-30 21:01
tags: 
     - Jetpack Compose Foundation
---

> Learn about the lifecycle of a composable and how Composable decides whether a composable needs recomposition.

## Lifecycle overview

> A Composition describes the UI of app and is produced by running composables.
>
> A Composition is a tree-structure of the composables that describe UI.

> A Composition can only be produced by an initial composition and updated by recomposition. The only way to modify a composition is through recomposition.

 **The lifecycle of a composable:** entering the Composition, getting recomposed 0 or more times, and leaving the Composition.

**Note:** When a composable needs to manage or interact with external resources that *do* have a more complex lifecycle, you should use **effects**.

If a composable is called multiple times, multiple instances are placed in the Composition. Each call has its own lifecycle in the Composition.

## Anatomy of a composable in Composition

> The instance of a composable in Composition is identified by its **call site**. The Composition compiler considers each call site as distinct. Calling composables from multiple call sites will create multiple instances of the composable in Composition.

> The **call site** is the *source code location* in which a composable is called. This influences its place in Composition, and therefore, the UI tree.

> If during a recomposition a composable calls different composables than it did during the previous composition, Compose will **identify which composables were called or not called** and for the composables that were called in both compositions, Compose will **avoid recomposing them if their inputs haven't changed.**

Composable 的实例是由它的调用位置来区分的，因为 Composition 编译器是用调用位置来区分它们的。所以同一个 composable 在不同位置被调用会创建不同的实例。

Compose 会尽可能的通过优化来提升效率和性能；除了忽略未被调用的 composable, 同时避免 recomposition 输入数据没有改变的 composable.

### Add extra information to help smart recompositions

> Calling a composable multiple times will add it to Composition multiple times.
>
> When calling a composable multiple times from the same call site, the execution order is used in addition to the call site in order to keep the instances distinct.

在同一个位置调用同一个 composable 多次会创建多个实例并将其添加到 composition，此时没有其它信息可以区分这些实例，Compose 会根据调用顺序来区分它们。

根据以上的原理，如果使用一个数组或者列表连续调用同一个 composable 来创建 UI，当在某个位置插入、移除或者重排数据时，所有因此发生位置变化的数据对应的 composable 都会 recomposition。

当然这种情况下的 recomposition 是不必要的，我们只需要 recomposition 实际输入数据发生变化的 composable。

> Compose provide a way to tell the runtime what values to identify a given part of the tree: the **key** composable.
>
> By wrapping a block of code with a call to the key composable with one or more values passed in, those values will be combined to be used to identify that instance in the composition. The value for a **key** needs to be unique amongst the invocations of composables at the call site.

通过将 composable 包装进 **key** composable, 并传入唯一的值来在 composition 中进行标记。

```kotlin
@Composable
fun MoviesScreen(movies: List<Movie>) {
  Column {
    for (movie in movies) {
      key(movie.id) { // Unique ID for this movie
        MovieOverview(movie)
      }
    }
  }
}
```

**Key Point:** Use the **key** composable to help Composable identify composable instances in Composition. It's important when multiple composables are called from the same call site and contain side-effects or internal state.

Some composables have built-in support for the **key** composable. 

```kotlin
@Composable
fun MoviesScreen(movies: List<Movie>) {
  LazyColumn {
    items(movies, key = { movie -> movie.id }) { movie ->
      MovieOverview(movie)
    }
  }
}
```

使用 **key** composable 帮助标记 composable 实例，当在相同的 call site 调用多个 composable 并且有 side-effect 或者 内部状态时(internal state) 时很有用。

### Skipping if the inputs haven't changed

If a composable is already in the composition, it can skip recomposition if all the inputs are stable and haven't changed.

> A stable type must comply with the following contract:
>
> - The result of **equals** for two instances will *forever* be the same for the same two instances.
> - If a public property of the type changes, Composition will be notified. 
> - All public property types are also stable.

可以跳过 recomposition 的条件是输入稳定并且没有改变。

**稳定类型需要满足：**

- 相同两个实例进行 equals 的结果永远是一样的。
- 类型的公共属性改变时，会通知 Composition.
- 所有公共属性的类型也是稳定。

> Some important common types that will be treat as stable, even though they are not explicitly marked as stable:
>
> - All primitive value types: Boolean, Int, Long, Float, Char, etc.
> - Strings
> - All Function types(lambdas)

**Note:** All deeply immutable types can safely be considered stable types.

常见的稳定类型：所有基本类型、字符串和方法(lambda)。所有真正不可变的类型都可当作稳定类型。

> One notable type that is stable but *is* mutable is Compose's **MutableState** type.
>
> If a value is held in a **MutableState**, the state object overall is considered to be stable as Compose will be notified of any changes to the **.value** property of **State**.

**Key Point:** Compose skips the recomposition of a composable if all the inputs are stable and haven't changed. The comparison uses the **equals** method.

> If Compose is not able to infer that a type is stable, but you want to force Compose to treat it as stable, mark ii with **@Stable** annotation.

**Key Point:** If Compose is not able to infer the stability of a type, annotate the type with **@Stable** to allow Compose to favor smart recompositions.

使用 **@Stable** 注解可以强制 Compose 把一个类型当作 Stable 的，同时让 Compose 支持智能的 recomposition.