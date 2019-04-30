---
layout: post
title:  "Navigation in Multi-Modules Projects"
date:   2019-04-30 00:00:00 +0300
categories: article
---
Navigation in developing Android apps is quite important and you should think twice what library suits (or your own solution) most and how it will be convenient to use when the app becomes bigger. Also, it might be good to think about how easy it will be to change your implementation to another one.

Before we will start, let me tell a story. Let's call it like this "How we made project modular and why I hated our navigation".

We had a single module project and everything worked fine except building time, that is because we used Dagger and it took too long to generate a single component. So we decided to separate the project into modules by features, for example, a login feature, a help feature and etc.

We had a lot of difficulties because we had to move network logic into its own Gradle module, domain login into its module and also we had issues with the navigation.

In our implementation, we had a router that knew about every Fragment and it was responsible for transitions between them. Unfortunately, every Fragment knew that there was the router and knew about other Fragments, because the Fragment said to the router what Fragment should be opened or the Fragment was waiting for the results from another Fragment. These things ruined the idea of making independent feature modules.

Therefore, I have been thinking how to make it better, how to break these connections between Fragments, how to get rid of knowledge that there is some router. In this article, I will show what I have made.

## What we will do
### Second
#### Last

Ballasd