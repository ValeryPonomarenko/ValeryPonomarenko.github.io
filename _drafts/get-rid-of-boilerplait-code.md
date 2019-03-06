---
layout: post
title: "Get rid of boilerplait code when using ComponentsManager"
categories: article
tags: [Dagger, Components Manager]
---
When you have a huge multi-modular project and you use ComponentsManager it is lickely that you have tons of code that just creates an anonumous object that bind the Dagger's components. In this article I will show how to make this code better.

The idea is to move the lines like these
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
to another place and in the Application class just get the list of the anonimous classes and iterate over it. So we will end up with something like this
```kotlin
val listOfBinders: List<IHasComponent<Any>> = ...
listOfBinder.forEach {
    XInjectionManager.bind(it)
}
```

##### Let's start

