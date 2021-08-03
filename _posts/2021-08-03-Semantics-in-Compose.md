---
title: Semantics in Compose
date: 2021-08-03 09:02
tags:
  - Jetpack Compose Foundation
---

# Semantics in Compose

> A Composition **describes the UI** of your app and is produced by running composables.
>
> The Composition is a tree-structure that consist of the composables that describe your UI.
>
> *Semantics tree* describes your UI in an alternative manner that is understandable for **Accessibility** services and for the **Testing** framework.
>
> The Semantics tree does not contain the information on how to draw your composables, but it contains information about the **semantic meaning** of your composables.
>
> If your app consists of composables and modifiers from the Compose foundation and material library, the Semantics tree is automatically filled and generated for you.
>
> **When you're adding custom low-level composables, you will have to manually provide its semantics**.

## Semantics properties

> All nodes in the UI tree with some semantic meaning have a parallel node in the Semantics tree.
>
> The node in the Semantics tree contains those properties that convey the meaning of the corresponding composable.
>
> Composables and modifiers that are built on top of the Compose **foundation library** already set the relevant properties for you.
>
> You can set or override the properties with the **semantics** and **clearAndSetSemantics** modifiers.
>
> To visualize the Semantics tee, we can use the **Layout Inspector** Tool or use the **printToLog()** method inside our tests.

```kotlin
class MyComposeTest {
  @get:Rule
  val composeTestRule = createComposeRule()
  
  @Test
  fun MyTest() {
    composeTestRule.setContent {
      MyTheme {
        Text("Hello world!")
      }
    }
    // Log the full semantics tee
    composeTestRule.onRoot().printToLog("MY TAG")
  }
}
```

**Note:** While developing your app, you should focus on getting the right meaning across using the semantics properties. Add accessibility-affecting semantics properties only if you are able to test them thoroughly, otherwise rely on the semantics added by the Compose foundation library instead.

## Merged and unmerged Semantics tree

> When a composable has no semantics properties set, it isn't included as part of the Semantics tree.
>
> The Semantics tree contains only the nodes that actually contain semantic meaning.
>
> Sometimes to convey the correct meaning of what is shown on the screen, it also useful to merge certain sub-trees of nodes and tree them as one.
>
> Composables and modifiers can indicate that they want to merge their descendants' semantics properties by calling **Modifier.semantics(mergeDescendants = true) {} .** 
>
> Several modifiers and composables in the Foundation and Material Compose libraries have this property set. For example, the **clickable**, **toggleable** and **ListItem**. 

### Inspecting the trees

> Merged Semantics tree, which merges descendant nodes when **mergeDescendants** is set to **true**. 
>
> Unmerged Semantics tree, which does not apply the merging, but keeps every node intact.
>
> Accessibility services use the unmerged tree and apply their own merging algorithms, taking into consideration the **mergeDescendants** property.
>
> The Testing framework uses the merged tree by default.

> By default, the merged tee will be logged. To print the unmerged tree, set the **useUnmergedTree** parameter of the **onRoot()** matcher to **true**.

```kotlin
composeTestRule.onRoot(useMeredTree = true).printToLog("MY TAG")

composeTestRule.onNodeWithText("Like").performClick()
```

### Merging behavior

> Each semantics property has a defined merging strategy.
>
> Descendants that themselves have set **mergeDescendants = true** are not included in the merge.

## Adapting the Semantics tree

> You can override or clear certain semantics properties, or change the merging behavior of the tree.
>
> This is particularly relevant when creating custom components.
>
> Without setting the correct properties and merge behavior, your app might not be accessible, and tests might behave differently than you expect.