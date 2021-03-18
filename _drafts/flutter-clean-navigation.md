---
layout: post
title: "Flutter: Clean Navigation"
poster: "/assets/flutter-navigation/poster.jpg"
categories: article
tags: ["flutter", "navigation"]
date:   2021-02-18 14:50:00 +0300
excerpt_separator: <!--more-->
---

Navigation is like a map, but for the application. If you have a map that is easy to read, to understand, so you will arrive to the destination faster. In an app it works the same way. When you have a method that opens a screen - you just call it and that is it. In this article I will share with you my way of implementing a navigation system in the flutter app.
<!--more-->

## Concept
Let's imaging that we are developing a news app. We have a list of news and when we click on one of them, the app opens it on a details screen.

{::nomarkdown}<iframe src=’https://dartpad.dev/embed-flutter.html?gh_owner=JoseAlba&gh_repo=flutter_code&gh_path=lib/dartpad&theme=dark&run=true&split=50' style=”position:absolute;top:0;left:0;width:100%;height:100%;”></iframe>{:/}

## Implementation

### Feature navigation

```dart
abstract class FeatureNavigation {

}
```

### AppNavigator