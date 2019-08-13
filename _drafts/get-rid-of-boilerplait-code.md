---
layout: post
title: "Boilerplait code when using ComponentsManager"
categories: article
tags: [dagger, components manager]
excerpt_separator: <!--more-->
---
When you have a huge multi-modular project and you use ComponentsManager it is likely that you have tons of code that just creates an anonymous object that binds the Dagger's components. In the article I will show you how to make the code cleaner.
<!--more-->

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
class MyApp : Application() {
    @Inject
    lateinit var listOfBinders: List<IHasComponent<Any>>

    override fun onCreate() {
        listOfBinder.forEach {
            XInjectionManager.bind(it)
        }
    }
}
```

##### Let's start
Firstly create a module class that will store all of the binders
```kotlin
@Module
class ComponentBinders {
    @Provides
    @IntoSet
    fun provideNetworkComponentBinder(): IHasComponent<Any> =
        object : IHasComponent<NetworkComponent> {
            override fun getComponent() = DaggerNetworkComponent().create()
        }

    @Provides
    @IntoSet
    fun provideAnalyticsComponentBinder(): IHasComponent<Any> =
        object : IHasComponent<AnalyticsComponent> {
            override fun getComponent() = DaggerAnalyticsComponent().create()
        }
}
```
Don't forget to add the module at the App's component. That's all.

Unfortunately, the solution is not perfect and it has a drawback. Let's imagine that the `AnalyticsComponent` needs a dependency from the `NetworkComponent`, so the `NetworkComponent` must be created before the `AnalyticsComponent`, at this point we have to prioritize the list, but how can we do it?