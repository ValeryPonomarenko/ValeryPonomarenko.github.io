---
layout: post
title:  "Navigation for Dynamic Features"
date:   2019-09-23 14:50:00 +0300
categories: article
tags: ["navigation", "multi-modules", "dynamic features"]
poster: "/assets/navigation/poster.jpg"
excerpt_separator: <!--more-->
---

Dynamic features give you an opportunity to reduce the size of the APK. But nothing is that easy as you might think.
<!--more-->

In my previous article I was talking how to make a navigation system in multi-modular app. In this article we will continue to use described approach.

## New Dynamic Feature

We will continue developing the app from the peviouse article, but we will add a new dynamic feature that will add a leaderboard to the app.

**App's screenshots**

## Implementation
### The First Try
Let's add a dynamic feature module. But the key differenct between "regular" modules and dynamic feature ones that the App module does not know about the dynamic feature modules, but the dynamic features know about the App module. Here is the diagram with connections between modules.

![Structure](/assets/dynamic-feature-navigation/first-diagram.png){:class="img-fluid mx-auto d-block"}

#### Leaderboard Module
Add a leaderboard dynamic feature module. Then add an app module as an dependencie, and also the 

```gradle
apply plugin: 'com.android.dynamic-feature'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
android { /* code */ }
dependencies {
    implementation project(':app')

    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'androidx.appcompat:appcompat:1.1.0-alpha01'
    implementation "com.github.valeryponomarenko.componentsmanager:androidx:2.0.1"
}
```

To be able to implement the feature's navigation interface, we put the it inside the app module. So create an interface inside the app module and name it LeaderboardNavigation.
![App Modules](/assets/dynamic-feature-navigation/first-app-structure.png){:class="img-fluid mx-auto d-block"}

And here is the methods that the navigation interface should have.

```kotlin
interface LeaderboardNavigation {
    fun openQuestionPreview(questionId: Long)
    fun openLeader(leaderName: String)
}
```

Finally, here is the fragments.
The first one is the Leaderboard fragment.

```kotlin
class LeaderboardFragment : Fragment() {

    private val leaderboardNavigation: LeaderboardNavigation by lazy {
        XInjectionManager.findComponent<LeaderboardNavigation>()
    }

    override fun onCreateView(
        inflater: LayoutInflater, 
        container: ViewGroup?, 
        savedInstanceState: Bundle?
    ): View? = inflater.inflate(R.layout.fragment_leaderboard, container, false)

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        button_john.setOnClickListener {
            leaderboardNavigation.openLeader("John Doe")
        }
    }
}
```

And here is the second one - the Leader fragment.

```kotlin
class LeaderFragment : Fragment() {

    companion object {
        private const val EXTRA_QUESTION_ID = 
            "me.vponomarenko.modular.navigation.leaderboard.name"

        fun createBundle(name: String) =
            Bundle().apply { putString(EXTRA_QUESTION_ID, name) }
    }

    private val leaderboardNavigation by lazy {
        XInjectionManager.findComponent<LeaderboardNavigation>()
    }

    private val leaderName by lazy {
        arguments?.getString(EXTRA_QUESTION_ID)
            ?: throw IllegalStateException("no name")
    }

    override fun onCreateView(
        inflater: LayoutInflater, 
        container: ViewGroup?, 
        savedInstanceState: Bundle?
    ): View? = inflater.inflate(R.layout.fragment_leader, container, false)

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        text_leader_name.text = leaderName
        button_question.text = getString(R.string.question, 1)
        button_question.setOnClickListener {
            leaderboardNavigation.openQuestionPreview(1)
        }
    }
}
```

The createBundle method is going to be used to pass the args between screens.

#### App Module

We have everything we need to implement the navigation interfaces. Firstly, let's add the fragments inside the navigation graph.

But here is the problem. These fragments are not avaliable in the App module, so we are able to drag-and-drop them, but we can add them in code. Do not worry that the studio shows that it cannot find the classes.

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/nav_graph"
    app:startDestination="@id/questionsFragment">
    <!-- fragments -->
    <fragment
        android:id="@+id/LeaderboardFragment"
        android:name="me.vponomarenko.modular.navigation.leaderboard.LeaderboardFragment"
        android:label="LeaderboardFragment" />
    <fragment
        android:id="@+id/LeaderFragment"
        android:name="me.vponomarenko.modular.navigation.leaderboard.LeaderFragment"
        android:label="LeaderFragment" />
</navigation>
```

Once we did it, they will appear on the graph and it will be possible to add connections between them. You can see all connection on the screenshot bellow.

**graph image**

Final step is to implement the navigation interface in the Navigator. Also the QestionsFragment was change a little, you can see changed code [here](link).

```kotlin
class Navigator : QuestionsNavigation, QuestionNavigation, RightAnswerNavigation,
    WrongAnswerNavigation,
    LeaderboardNavigation {

    // code

    override fun openQuestionPreview(questionId: Long) {
        navController?.navigate(
            R.id.action_leaderFragment_to_questionFragment,
            QuestionFragment.createBundle(questionId, true)
        )
    }

    override fun openLeader(leaderName: String) {
        navController?.navigate(
            R.id.action_leaderboardFragment_to_leaderFragment,
            LeaderFragment.createBundle(leaderName)
        )
    }
}
```

#### Result
Now you can run the application and make sure that everything works.
What I do not like in this solution is that we have to put the navigation interface inside the app module, add some xml code by ourselves. But we could fix these issues.

### Make Navigation Great Again
Instead of adding navigaion interface inside the app module, let's create a new one that will be called leaderboard api module. The diagram shows how the modules are connected.

![App Modules](/assets/dynamic-feature-navigation/second-diagram.png){:class="img-fluid mx-auto d-block"}

#### Leaderboard Api Module
Create a new module. Here is the gradle file of the leaderboard api module

```gradle
apply plugin: 'com.android.library'
apply plugin: 'kotlin-android'
android { /* code */ }
dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
}
```

And move the LeaderboardNavigation interface from the app module inside the new one.

#### Leaderboard Module
Here is the changed gradle file.

```gradle
apply plugin: 'com.android.dynamic-feature'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
android { /* code */ }
dependencies {
    implementation project(':app')
    implementation project(':leaderboard_api')

    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'androidx.appcompat:appcompat:1.1.0-alpha01'
    implementation "com.github.valeryponomarenko.componentsmanager:androidx:2.0.1"
}
```

Unfortunately, we cannot remove the app module from the dependencies block, because it is required.

#### App Module
We just need to add the leaderboard api module inside the dependencies block, so the navigation interface will be avaliable.

```gradle
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'

android {
    // code
    dynamicFeatures = [":leaderboard"]
}

dependencies {
    //code
    implementation project(':leaderboard_api')
    // code
}
```

#### Result
That is it. We have added the API module that has all classes that the app module and the leaderboard module would use. But we still have to write XML code by ourselves. If you are OK with that, that is not a probles, but if you are not, let's fix it.
