---
title: Architectural Layering
date: 2021-08-02 21:12
tags:
    - Jetpack Compose Foundation
---

# Architectural Layering

This page provide a high-level overview of the architectural layers that make up Jetpack Compose, and core principles that inform this design.

Jetpack Compose is not a single monolithic project; it is created from a number of modules which are assembled together to form a complete stack. Understanding the different modules that make up Jetpack Compose enable you to:

- Use the appropriate level of abstraction to build your app or library
- Understand when you 'drop down' to a lower level for more control or customization
- Minimize you dependencies

## Layers

The major layers of Jetpack Compose are: Material --> Foundation --> UI --> Runtime.

Each layer is built upon the lower levels, combining functionality to create higher level components. 

Each layer builds upon public APIs of the lower layers to verify the module boundaries and enable you to replace any layer should need to.

**Runtime**

> This module provides the fundamentals of the Compose runtime such as **remember**, **mutableStateOf**. You might consider building directly upon this layer if you only need Compose's tree management abilities, not its UI.

**UI**

> The UI layer is made up of multiple modules  (**ui-text**, **ui-graphics**, **ui-tooling**, etc.). 
>
> These modules implement the fundamental of the UI toolkit, such as **LayoutNode**, **Modifier**, input handlers, custom layouts, nd drawing. 
>
> You might consider building upon this layer if you only need fundamental concepts of a UI toolkit.

**Foundation**

This module provides design system agnostic building blocks for Compose UI, like **Row** and **Column**, **LazyColumn**, recognition of particular gestures, etc.

You might consider building upon the foundation layer to create your own design system.

**Material**

This module provide an implementation of the Material Design system for Compose UI, providing a theming system, styled components, ripple indications, icons. Build upon this layer when using Material Design in your app.

## Design Principles

A guiding principle for Jetpack Compose is to provide small, focused pieces of functionality that can be assembled (or composed) together, rather than a new monolithic components.

### Control

> Higher level components tend to do more for you, but limit the amount of direct control that you have.
>
> If you need more control, you can 'drop down' to use a lower level component.

```kotlin
// the hight level way
val color = animateColorAsState(if (condition) Color.Green else Color.Red)

// the low level way
val color = remember { Animatable(Color.Gray) }
LaunchedEffect(condition) {
  color.animateTo(if (condition) Color.Green else Color.Red)
}
```

### Customization

> Assembling higher level components from smaller building blocks makes it far easier to customize components should you need to.

**Caution:**

- When dropping down to a lower layer to customize a component, ensure that you do not degrade any functionality by, for example neglecting accessibility support. Use the component you  are forking as a guide.
- Forking a component means that you will not benefit from any future additions or bug fixes from the upstream component.

### Picking the right abstraction

> Compose's philosophy of building layered, reusable components means that you should not always reach for the lower level building blocks.
>
> As a rule, prefer building on the *highest-level* component which offers the functionality you need in order to benefit from the best practices they include.