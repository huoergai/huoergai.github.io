---
title: Graphics in Compose
date: 2021-08-06 10:30
tags:
  - Jetpack Compose Design
---

Jetpack Compose makes it easier to work with custom graphics. With Compose's declarative approach, all the graphic configuration happens in one place, instead of being split between a method call and a Paint helper object. Compose takes care of creating and updating the needed objects in an efficient way.

## Declarative graphics with Compose

Compose extends its declarative approach to how it handles graphics. Compose's approach offers a number of advantages:

- Compose minimizes the **state** in its graphic elements, helping you **avoid state's programming pitfalls**.
- When you draw something, all the options are right where you'd expect them, in the composable function.
- Compose's graphics APIs take care of creating and freeing objects in an efficient way.

**Note:** Under the hood, Compose relies on the view-based UI's Canvas and other associated objects. However , Compose simplifies many of the more confusing aspects of **Canvas**.

## Canvas

The core composable for custom graphics is **Canvas**. You place the Canvas in you layout the same way you would with any other Compose UI element. Within the Canvas, you can draw elements with precise control over their style and location.

**Note:** The **Canvas** composable makes use of a special Compose **Canvas** object, which in turn creates and manages a view-based **Canvas**. However, Compose does a lot of the work for you, maintaining state and creating and freeing necessary helper objects.

```kotlin
Canvas(modifier = Modifier.fillMaxSize()) {
  
}
```

The **Canvas** automatically exposes a **DrawScope**, a scoped drawing environment that maintains its own state. This lets you set the parameters for a group of graphical elements.

The **DrawScope** provides several useful fields, like **Size**, a **Size**  object specifying the current and maximum dimensions of the DrawScope.

```kotlin
Canvas(modifier = Modifier.fillMaxSize()) {
  val canvasWidth = size.width
  val vanvasHeigh = size.height
  
  drawLine(
    start = Offset(x = canvasWidth, y = 0f),
    end = Offset(x = 0f, y = canvasHeight),
    color = Color.Blue,
    strokeWidth = 5F
  )
}
```

There are many other simple drawing functions, such as **drawRect** and **drawCircle**.

```kotlin
Canvas(modifier = Modifier.fillMaxSize()) {
  val canvasWidth = size.width
  val canvasHeight = size.height
  drawCircle(
    color = Color.Blue,
    center = Offset(x = canvasWidth / 2, y = canvasHeight / 2),
    radius = size.minDimension / 4
  )
}
```

## DrawScope

Each Compose **Canvas** exposes a **DrawScope**, a scoped drawing environment, where you  actually issue your drawing commands.

You can use the function **DrawScope.inset()**, to adjust the default parameters of current scope, changing the drawing boundaries and translating the drawings accordingly. Operations like **inset()**  apply to all drawing operations within the corresponding lambda.

```kotlin
val canvasQuadrantSize = size / 2F
inset(50F,30F) {
  drawRect(
    color = Color.Green,
    size = canvasQuadrantSize
  )
}
```

**DrawScope** offers other simple transformations, like **rotate()*

```kotlin
rotate(degress = 45F) {
  drawRect(
    color = Color.Gray,
    topLeft = Offset(x = canvasWidth / 3F, y = canvasHeight / 3F),
    size = canvasSize / 3F
  )
}
```

If you want to apply multiple transformations to your drawings, the best approach is not to create nested **DrawScope** environments. Instead, you should use the **withTransform** function, which creates and applies a single transformation that combines all your desired changes.

Using **withTransform()** is more efficient than making nested calls to individual transformations, because all the transformations are performed together in a single operation, instead of Compose needing to calculate and save each of the nested transformations.

```kotlin
withTransform({
  translate(left = canvasWidth / 5F)
  rotate(degress = 45F)
}) {
  drawRect(
    color = Color.Gray,
    topLeft = Offset(x = canvasWidth / 3F, y = canvasHeight / 3F),
    size = canvasSize / 3F
  )
}
```

