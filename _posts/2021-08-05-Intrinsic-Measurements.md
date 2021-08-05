---
title: Intrinsic measurements
date: 2021-08-05 21:51
tags:
  - Jetpack Compose Design
---

> One of the rules of Composable is that you should only measure your children once; measuring children twice throws a runtime exception.

**Intrinsics lets you query children before they're actually measured.**

- (min|max) IntrinsicWidth: Given this height, what's the minimum/maximum width you can paint your content properly?
- (min|max) IntrinsicHeight: Given this width, what's the minimum/maximum height you  can paint your content property?

**Note:** Asking for intrinsics measurements doesn't measure the children twice. Children are queried for their intrinsic measurements before they're measured and then, based on that information the parent calculates the constraints to measure its children with.

##  Intrinsics in action

```kotlin
@Composable(
  text1: String,
  text2: Stirng,
  modifier:Modifer = Modifier
) {
  Row(modifer = modifer.height(IntrinsicSize.Min)) {
    Text(
      modifier = Modifier
          .weight(1f)
          .padding(start = 4.dp)
          .wrapContentWidth(Alignment.Start),
      text = text1
    )
    Divider(
      color = Color.Black,
      modifier = Modifier.fillMaxHeight().width(1.dp)
    )
    Text(
      modifier = Modifier
          .weight(1f)
          .padding(end = 4.dp)
          .wrapContentWidth(Alignment.End),
      text = text2
    )
  }
}

```

### Intrinsics in custom layouts

When creating a custom Layout or layout modifier, intrinsic measurements are calculated automatically based on approximations.

To specify the instrinsics measures of your custom Layout, override the minIntrinsicWidth, minIntrinsicHeight, maxIntrinsicWidth, and maxIntrinsicHeight of the MeasurePolicy interface when creating it.

```kotlin
@Composable
fun MyCustomComposable(
  modifier: Modifier = Modifier,
  content: @Composable () -> Unit
) {
  return object : MeasurePolicy {
    override fun MeasureScope.measure(
      measurables: List<Measurable>,
      constraints: Constraints
    ): MeasureResult {
      // measure and layout here
    }
    
    override fun IntrinsicMeasureScope.minIntrinsicWidth(
      measurables: List<IntrinsicMeasurable>,
      height: Int
    ) = {
      // logic here
    }
    // Other intrinsics related mehtods have a default value,
    // you can override only the mothods that you need.
  }
}
```

When creating your custom layout modifier, override the related methods in the LayoutModifier interface.

```kotlin
fun Modifier.myCustomModifer(/* ... */) = this.then(object : LayoutModifier {
  override fun MeasureScope.measure(
    measurable: Measurable,
    constraints: Constraints
  ): MeasureResult {
    // measure and layout here
  }
  
  override fun IntrinsicMeasureScope.minIntrinsicWidth(
    measuable: IntrinsicMeasurable,
    height: Int
  ): Int = {
    // logic here
  }
  // Other intrinsics related methods have a default value,
  // you can override only the methods that you need.
})
```