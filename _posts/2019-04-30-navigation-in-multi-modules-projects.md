---
layout: post
title:  "Navigation in Multi-Modules Projects"
date:   2019-04-30 14:50:00 +0300
categories: article
tags: ["navigation", "multi-modules"]
poster: "/assets/navigation/poster.jpg"
excerpt_separator: <!--more-->
---

Navigation in developing Android apps is quite important and you should think twice what library suits (or your own solution) most and how it will be convenient to use when the app becomes bigger. Also, it might be good to think about how easy it will be to change your implementation to another one.
<!--more-->

Before we will start, let me tell a story. Let's call it like this "How we made project modular and why I hated our navigation".

We had a single module project and everything worked fine except building time, that is because we used Dagger and it took too long to generate a single component. So we decided to separate the project into modules by features, for example, a login feature, a help feature and etc.

We had a lot of difficulties because we had to move network logic into its own Gradle module, domain login into its module and also we had issues with the navigation.

In our implementation, we had a router that knew about every Fragment and it was responsible for transitions between them. Unfortunately, every Fragment knew that there was the router and knew about other Fragments, because the Fragment said to the router what Fragment should be opened or the Fragment was waiting for the results from another Fragment. These things ruined the idea of making independent feature modules.

Therefore, I have been thinking how to make it better, how to break these connections between Fragments, how to get rid of knowledge that there is some router. In this article, I will show what I have made.

## What we will do
![Application](/assets/navigation/app.gif){:class="img-fluid mx-auto d-block"}

The application will be super simple but it will be modular. There will be one application module and three feature modules:

* **App** module — this module knows everything about the app and there we will implement our navigation. We will use the Navigation Component from JetPack.
* **Questions** module — this module is responsible for the questions feature. The module knows nothing about the App module or any other module. There will be a `QuestionsFragment` and a `QuestionsNavigation` interface.
* **Question** module — there will be a question feature. As the previous module, it does not know about other modules. That module will contain a `QuestionFragment` and a `QuestionNavigation` interface.
* **Result** module — this is a result feature. Also, it completely independent module. There will be a `RightAnswerFragment` with `RightAnswerNavigation` interface and there will be a `WrongAnswerFragment` with `WrongAnswerNavigation` interface.

The source code for the app can be found here: [GitHub](https://github.com/ValeryPonomarenko/ComponentsManager)

## Implementation

In this article, I use **ComponentsManager** library to get the feature's navigation. In a real project, I will provide feature's navigation by Dagger but for this small project it is not necessary. Also, the **ComponentManager** library helps you to use **Dagger** in multi-module projects. [Read more about ComponentsManager](https://github.com/ValeryPonomarenko/ComponentsManager)

### Questions Module

Create an Android module and call it **questions**. After that make sure that Kotlin is configured in the module and the **ConstraintLayout*** and the **ComponentsManager** are added as dependencies.

```gradle
apply plugin: 'com.android.library'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
android { ... }
dependencies {
    ...
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'androidx.appcompat:appcompat:1.1.0-alpha01'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    implementation "com.github.valeryponomarenko.componentsmanager:androidx:2.0.1"
    ...
}
```

Instead of using `Route` or `Navigator` in the Fragment, we will create an interface `QuestionsNavigation` that defines the needed transitions. Thus for the Fragment, it does not matter how those transitions will be implemented, it just needs that interface and relies on when the method is called, the needed screen will be open.

```kotlin
interface QuestionsNavigation {
    fun openQuestion(questionId: Long)
}
```

To open a screen with a question, we just call the `openQuestion(questionId: Long)` method and that is it. We do not care how the screen will be opened, we do not even care whether it is Fragment or Activity or something else.

Here is the `QuestionsFragment` and its [layout](https://github.com/ValeryPonomarenko/modular-navigation/blob/master/questions/src/main/res/layout/fragment_questions.xml).

```kotlin
class QuestionsFragment : Fragment() {

    private val navigation: QuestionsNavigation by lazy {
        XInjectionManager.findComponent<QuestionsNavigation>()
    }

    override fun onCreateView(
        inflater: LayoutInflater, 
        container: ViewGroup?, 
        savedInstanceState: Bundle?
    ): View? = inflater.inflate(R.layout.fragment_questions, container, false)

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        button_first_question.setOnClickListener { navigation.openQuestion(1) }
        button_second_question.setOnClickListener { navigation.openQuestion(2) }
        button_third_question.setOnClickListener { navigation.openQuestion(3) }
    }
}
````

### Question Module

Firstly, create an Android library module and call it **question**. The module's build.gradle must contain the same dependencies as the question module's build.gradle file.

```gradle
apply plugin: 'com.android.library'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
android { ... }
dependencies {
    ...
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'androidx.appcompat:appcompat:1.1.0-alpha01'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    implementation "com.github.valeryponomarenko.componentsmanager:androidx:2.0.1"
    ...
}
````

After that we create an interface that defines module's navigation. The navigation idea in other modules will be the same as in the previous one.

From the `QuestionFragment` user can open a wrong answer screen or a right answer screen so the interface will have two methods.

```kotlin
interface QuestionNavigation {
    fun openWrongAnswer()
    fun openRightAnswer()
}
```

The last thing is fragment. For this fragment, we will add a companion method that returns a `Bundle` so we will be able to pass question's id and use it. The fragment's [layout](https://github.com/ValeryPonomarenko/modular-navigation/blob/master/question/src/main/res/layout/fragment_question.xml).

```kotlin
class QuestionFragment : Fragment() {

    companion object {
        private const val EXTRA_QUESTION_ID = 
            "me.vponomarenko.modular.navigation.question.id"

        fun createBundle(questionId: Long) =
            Bundle().apply { putLong(EXTRA_QUESTION_ID, questionId) }
    }

    private val navigation: QuestionNavigation by lazy {
        XInjectionManager.findComponent<QuestionNavigation>()
    }

    private val questionId: Long by lazy {
        arguments?.getLong(EXTRA_QUESTION_ID) 
            ?: throw IllegalStateException("no questionId")
    }

    override fun onCreateView(
        inflater: LayoutInflater, 
        container: ViewGroup?, 
        savedInstanceState: Bundle?
    ): View? = inflater.inflate(R.layout.fragment_question, container, false)

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        text_question.text = getString(R.string.question, questionId)
        button_right_answer.setOnClickListener { navigation.openRightAnswer() }
        button_wrong_answer.setOnClickListener { navigation.openWrongAnswer() }
    }
}
```

### Result Module

The final feature module is a result module. There will be two fragments that show a right and wrong answer. Create an Android library module and call it **result**, then change the module's build.gradle so it will have the same dependencies as the previous feature modules.

```gradle
apply plugin: 'com.android.library'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
android { ... }
dependencies {
    ...
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'androidx.appcompat:appcompat:1.1.0-alpha01'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    implementation "com.github.valeryponomarenko.componentsmanager:androidx:2.0.1"
    ...
}
```

Let's start from the right answer fragment. The navigation interface will have one method because the user can open only an all question screen.

```kotlin
interface RightAnswerNavigation {
    fun openAllQuestions()
}
````

The `RightAnswerFragment`, its [layout](https://github.com/ValeryPonomarenko/modular-navigation/blob/master/result/src/main/res/layout/fragment_right.xml).

```kotlin
class RightAnswerFragment : Fragment() {

    private val navigation: RightAnswerNavigation by lazy {
        XInjectionManager.findComponent<RightAnswerNavigation>()
    }

    override fun onCreateView(
        inflater: LayoutInflater, 
        container: ViewGroup?, 
        savedInstanceState: Bundle?
    ): View? = inflater.inflate(R.layout.fragment_right, container, false)

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        button_all_questions.setOnClickListener { navigation.openAllQuestions() }
    }
}
```

As I said earlier, this module has two Fragments, so let's implement a wrong answer screen.

In my implementation, from the wrong answer screen user can only go back to the question's screen and try to answer the question again, so the navigation interface has only one method.

```kotlin
interface WrongAnswerNavigation {
    fun tryAgain()
}
```

And the `WrongAnswerFragment`, its [layout](https://github.com/ValeryPonomarenko/modular-navigation/blob/master/result/src/main/res/layout/fragment_wrong.xml).

```kotlin
class WrongAnswerFragment : Fragment() {

    private val navigation: WrongAnswerNavigation by lazy {
        XInjectionManager.findComponent<WrongAnswerNavigation>()
    }

    override fun onCreateView(
        inflater: LayoutInflater, 
        container: ViewGroup?, 
        savedInstanceState: Bundle?
    ): View? = inflater.inflate(R.layout.fragment_wrong, container, false)

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        button_try_again.setOnClickListener { navigation.tryAgain() }
    }
}
```

### App Module

We have made the modules and it is time to connect them and run the app.

First thing first, we need to edit the app module's build.gradle. It must have all the created modules and a Navigation Component library with the Components Manager.

```gradle
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
android { ... }
dependencies {
    ...
    implementation project(':questions')
    implementation project(':question')
    implementation project(':result')

    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'androidx.appcompat:appcompat:1.1.0-alpha01'
    implementation 'com.github.valeryponomarenko.componentsmanager:androidx:2.0.1'
    implementation 'android.arch.navigation:navigation-fragment-ktx:1.0.0-alpha11'
    ...
}
```

Before implementation the classes that work with navigation, we have to add the navigation itself. We will use the Navigation Component to built it.

Create a navigation resource file and call it **nav_graph.xml**. There will be three connections:

* The `QuestionsFragment` with the `QuestionFragment`;
* The `QuestionFragment` with the `WrongAnswerFragment`;
* The `QuestionFragment` with the `RightAnswerFragment`. This connection has a little difference. If the user in on the `RightAnswerFragment` and the user presses the back button, he or she will be returned to the `QuestionsFragment`. How make that happen? Just select the arrow that connects the `questionFragment` with `rightAnswerFragment`, then in the drop-down list near the **Pop to** select the `questionsFragment`.

![Navigation graph](/assets/navigation/navigation.png){:class="img-fluid"}

Here is the XML representation of the navigation graph.

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/nav_graph"
    app:startDestination="@id/questionsFragment">
    <fragment
        android:id="@+id/questionsFragment"
        android:name="me.vponomarenko.modular.navigation.questions.QuestionsFragment"
        android:label="QuestionsFragment">
        <action
            android:id="@+id/action_questionsFragment_to_questionFragment"
            app:destination="@id/questionFragment" />
    </fragment>
    <fragment
        android:id="@+id/questionFragment"
        android:name="me.vponomarenko.modular.navigation.question.QuestionFragment"
        android:label="QuestionFragment">
        <action
            android:id="@+id/action_questionFragment_to_wrongAnswerFragment"
            app:destination="@id/wrongAnswerFragment" />
        <action
            android:id="@+id/action_questionFragment_to_rightAnswerFragment"
            app:destination="@id/rightAnswerFragment"
            app:popUpTo="@+id/questionsFragment" />
    </fragment>
    <fragment
        android:id="@+id/wrongAnswerFragment"
        android:name="me.vponomarenko.modular.navigation.result.wrong.WrongAnswerFragment"
        android:label="WrongAnswerFragment" />
    <fragment
        android:id="@+id/rightAnswerFragment"
        android:name="me.vponomarenko.modular.navigation.result.right.RightAnswerFragment"
        android:label="RightAnswerFragment" />
</navigation>
```

Then, create a `Navigator` class. The class is responsible for making transitions between fragments. It implements all the `navigation`s interfaces from the feature modules, do not forget to add calls of the corresponding actions to open the required screen. Also, the class has methods to bind the `navController` and unbind it.

```kotlin
class Navigator : QuestionsNavigation, QuestionNavigation, RightAnswerNavigation, WrongAnswerNavigation {

    private var navController: NavController? = null

    override fun openQuestion(questionId: Long) {
        navController?.navigate(
            R.id.action_questionsFragment_to_questionFragment,
            QuestionFragment.createBundle(questionId)
        )
    }

    override fun openWrongAnswer() {
        navController?.navigate(R.id.action_questionFragment_to_wrongAnswerFragment)
    }

    override fun openRightAnswer() {
        navController?.navigate(R.id.action_questionFragment_to_rightAnswerFragment)
    }

    override fun openAllQuestions() {
        navController?.popBackStack()
    }

    override fun tryAgain() {
        navController?.popBackStack()
    }

    fun bind(navController: NavController) {
        this.navController = navController
    }

    fun unbind() {
        navController = null
    }
}
```

On the 8th line you can see how I pass question's id to the `QuestionFragment`.

After that, create a `NavApplication` class. Actually, I had to add this class to make `Navigator` available to other classes, otherwise, it would be harder to get the navigator in the feature modules. Do not forget to add this class to Manifest.

```kotlin
class NavApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        XInjectionManager.bindComponentToCustomLifecycle(object : IHasComponent<Navigator> {
            override fun getComponent(): Navigator = Navigator()
        })
    }
}
```

With the 4th-6th lines, it would be possible to get the implementations of features' navigation interface by calling `XInjectionManager.findComponent<QuestionNavigation>()`.

The last but not least, change the `MainActivity` class.

```kotlin
class MainActivity : AppCompatActivity() {

    private val navigator: Navigator by lazy {
        XInjectionManager.findComponent<Navigator>()
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }

    override fun onResume() {
        super.onResume()
        navigator.bind(findNavController(R.id.nav_host_fragment))
    }

    override fun onPause() {
        super.onPause()
        navigator.unbind()
    }

    override fun onSupportNavigateUp(): Boolean = findNavController(R.id.nav_host_fragment).navigateUp()
}
```

And the activity's layout.

```xml
<?xml version="1.0" encoding="utf-8"?>
<fragment xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/nav_host_fragment"
    android:name="androidx.navigation.fragment.NavHostFragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:defaultNavHost="true"
    app:navGraph="@navigation/nav_graph" />
```

That is all, now you can run the app and see how it works.

## Summary

Good job! We have made a navigation system that:

* breaks the connections between fragments in different feature modules,
* can be easily changed,
* is easy to test.