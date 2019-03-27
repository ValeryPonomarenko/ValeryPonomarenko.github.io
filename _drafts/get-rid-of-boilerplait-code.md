---
layout: post
title: "Get rid of boilerplait code when using ComponentsManager"
categories: article
tags: [Dagger, Components Manager]
---
When you have a huge multi-modular project and you use ComponentsManager it is likely that you have tons of code that just creates an anonymous object that binds the Dagger's components. In the article I will show you how to make the code cleaner.

The idea is to get rid of these lines in your `Application` class.
```kotlin
XInjectionManager
    .bindComponent(object : IHasComponent<SomeComponent> {
        fun getComponent() = DaggerSomeComponent.create()
    })
XInjectionManager
    .bindComponent(object : IHasComponent<AnotherComponent> {
        fun getComponent() = DaggerAnotherComponent.create()
    })
```
It is clear that we cannot just remove the code, so we will move it to the Dagger's modules. In the `Application` class the list of binders will be injected
```kotlin
val listOfBinders: List<IHasComponent<Any>> = /* get the list */
listOfBinder.forEach {
    XInjectionManager.bind(it)
}
```

##### Let's start

