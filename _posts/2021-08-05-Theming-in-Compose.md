---
title: Theming in Compose
date: 2021-08-05 21:59
tags:
   - Jetpack Compose Design

---

Jetpack Compose make it easy to give your app a consistent look and feel by applying themes. You can customize Compose's implementation of **Material Design** to fit your product's brand. If that doesn't suit your needs, you can build a custom design system using Compose's public APIs.

## Application-wide theming

Jetpack Compose offers an implementation of **Material Design**, a comprehensive design system for creating digital interfaces. The Material Design **components** are built on top of **Material Theming**, which is a systematic way to customize Material Design to better reflect your product's brand. A Material Theme comprises **color**, **typography** and **shape** attributes. When you customize these attributes, your changes are automatically reflected in the components you use to build your app.

```kotlin
MaterialTheme(
  colors = ...,
  typography = ...,
  shapes = ...
) {
  // app content
}
```

Configure the parameters pass to **MaterialTheme** to theme application.

### Color

> Colors are modelled in Compose with the **Color** class, a simple data holding class.

```kotlin
val Red = Color(0xffff0000)
val Blue = Color(red = 0f, green = 0f, blue = 1f)
```

> While you can organize these however you like (as top-level constants, within a singleton, or defined inline), we **strongly** recommend specifying colors in your theme and retrieving the colors from there. This approach makes it possible to support multiple themes, like dark theme.

可以将颜色放在全局常量里、单例里或者在内联定义，但最好通过主题进行定义和获取，这样可以使应用支持多个主题。

> Compose provides the **Colors** class to model the **Material color system**. **Colors** provides builder functions to create sets of light or dark colors.

```kotlin
private val Yellow200 = Color(0xffffeb46)
private val Blue200 = Color(0xff91a4fc)
// ...

private val DarkColors = darkColors(
  primary = Yellow200,
  secondary = Blue200,
  // ...
)
private val LightColors = lightColors(
  primary = Yellow500,
  primaryVariant = Yellow400,
  secondary = Blue700,
  // ...
)

MaterialTheme(
  colors = if (darkTheme) DarkColors else LightColors
) {
  // app content
}
```

**Using theme colors**

> Retrieve the **Colors** provided to the **MaterialTheme** composable by using **MaterialTheme.colors.**

```kotlin
Text(
  text = "Hello theming",
  color = MaterialTheme.colors.primary
)
```

**Surface and content color**

Many components accept a pair of color and "content color";

This enables you to not only set the color of a composable, but also to provide a default color for the content, the composables contained within it. Many composables use this content color by default.

The **contentColorFor()**  method retrieves the appropriate "on" color for any theme colors.

**Note:** When setting the background color of any elements, prefer using a **Surface** to do this, as the **Surface** sets an appropriate content color. Be wary of direct **Modifier.backround()** calls, which do not set an appropriate content color.

**Content Alpha**

> Often we want to vary how much we emphasize content to communicate importance and provide visual hierarchy. Material Design **recommends** employing different levels of opacity to convey these different importance levels.
>
> Jetpack Compose implements this via **LocalContentAlpha**. You can specify a content alpha for a hierarchy by **providing** a value for this **CompositionLocal**. Material specifies some standard alpha values ( **high, medium, disable** ) which are modelled by the **ContentAlpha** object. MaterialTheme defaults **LocalContentAlpha** to **ContentAlpha.high**.

```kotlin
CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
  Text(/* ... */)
}
CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.disabled) {
  Icon(/* ... */)
  Text(/* ... */)
}
```

**Dark theme**

> In Composable, you implement light and dark themes by providing different sets of **Colors** to the **MaterialTheme** composable, and consuming colors through the theme.

```kotlin
@Composable
fun MyTheme(
  darkTheme: Boolean = isSystemInDarkTheme(),
  content: @Composable () -> Unit
) {
  MaterialTheme(
    colors = if (drakTheme) DarkColors else LightColors,
    /* ... */
    content = content
  )
}
```

> When implementing a dark theme, you can check if the current **Colors** are light or dark, The value is set by the **lightColors()** and **darkColors()** builder functions.

```kotlin
val isLightTheme = MaterialTheme.colors.isLight
```

> In Material, surfaces in dark themes with higher elevations receive **elevation overlay**, which lightens their background. These overlays are implemented automatically by the **Surface** composable when using dark colors.

```kotlin
Surface(
  elevation = 2.dp,
  color = MaterialTheme.colors.surface, // color will be adjusted for elevation
) {
  /* ... */
}
```

**Extending Material colors**

Compose closely models Material's color theming to make it simple and type-save to follow material guidelines. If you need to extend the color set, you can either implement your own color system as shown below, or add extensions.

```kotlin
val Colors.snackbarAction: Color
    @Composable get() = if (isLight) Red300 else Red700
```

### Typography

Material defines a **type system**, encouraging you to use a small number of semantically-named styles.

Compose implements the type system with the **typography, TextStyle,** and **font-related** classes. The **typography** constructor offers defaults for each styles so you can omit any you don't want to customize.

**Note:** Compose does not support Android's Downloadable Fonts feature at this time, but does support bundled font resources.

**Using text styles**

Retrieve **TextStyle**  from the theme.

```kotlin
Text(
  text = "Subtitle2 styled",
  style = MateiralTheme.typography.subtitle2
)
```

### Shape

Material defines a **shape system**, allowing you to define shapes for large, medium, and small components.

Compose implements the shape system with the **Shapes** class, which lets you specify a **CornerBasedShape** for each category. Many components use these shapes by default.

```kotlin
val Shapes = Shapes(
  small = RoundedCornerShape(percent = 50),
  medium = RoundedCornerShape(0f),
  large = CutCornerShape(
    topStart = 16.dp,
    topEnd = 0.dp,
    bottomEnd = 0.dp,
    bottomStart = 16.dp
  )
)

MaterialTheme(shapes = Shapes, /* ... */)
```

**Using shapes**

Retrieve the shapes from the theme.

```kotlin
Surface(
  shape = MaterialTheme.shapes.medium, /* ... */
) {
  /* ... */
}
```

## Component styles

There is no explicit concept of component style in Compose, as you provide this functionality by creating your own composables.

```kotlin
@Composable
fun LoginButton(
  onClick: () -> Unit,
  modifier: Modifier = Modifier,
  content: @Composable RowScope.() -> Unit
) {
  Button(
    colors = ButtonDefaults.buttonColors(
      backgroundColor = MateiralTheme.colors.secondary
    ),
    onClick = onClick,
    modifier = modifier,
    content = content
  )
}
```

## Custom design system

It's entirely possible to create your design system in the same manner; Material is built entirely on public APIs which you can use to achieve this.

