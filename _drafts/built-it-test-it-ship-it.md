---
layout: post
title: "Build It, Test It, Ship It"
categories: article
tags: ["continious integration", "continious delivery"]
---
When your project is getting bigger and you want to control the codestyle, make sure that changes do not break the other parts of the app, publish your app by some event and moreover you do not want to make all that by yourself or by team-mates.

Luckily, there are a lot services that are ready the give you the opportunity, some of them are free, some have limitation for private repositories.

In this article I'm going to work with Github and CircleCI.

What we will do
We will make a workflow that is going to be run on each commit to the remote repository. The workflow will be able to check the codestyle, run static analyzer, run test both unit and ui, publish the app into Google Play. Also, the workflow will wait until the previous step finished, so if the codestyle step fails, there is no need to run unit test as the codestyle is broken. If the unit tests step fails, the UI test will not be run ann so on. This dependencies between steps save time as for developers and for CI as well.


Boring part: getting keys
Before we can start implementing the workflow, we need to get the keys. What key do we need? The Dropbox key — it will be used for downloading the archive with signing key and its credentials from your Dropbox account. The Firebase key — with this key we will be able to run the instrumentation tests in Firebase Test Lab. The final one is the Google Play key — this key is going to be used for publishing the app on Google Play.


Helpful scripts
Also, we will write two bash scripts. The first one is for downloading the signing data and the second one is for running the instrumentation tests. Actually, they are not so important and you can do everything without them, but in my opinion, with them, the solution will be cleaner.

```bash
#!/bin/bash
curl -X POST https://content.dropboxapi.com/2/files/download \
  --header "Authorization: Bearer $DROPBOX_KEY" \
  --header "Dropbox-API-Arg: {\"path\": \"/$SIGNING_ARCHIVE_NAME\"}" \
  -o "./$SIGNING_ARCHIVE_NAME" \
  && unzip -o $SIGNING_ARCHIVE_NAME \
  && rm $SIGNING_ARCHIVE_NAME
```

```bash
#!/bin/bash
gcloud firebase test android run \
  --type instrumentation \
  --app ../app/build/outputs/apk/debug/app-debug.apk \
  --test ../app/build/outputs/apk/androidTest/debug/app-debug-androidTest.apk \
  --device-ids Nexus6 \
  --os-version-ids 21 \
  --locales en \
  --orientations portrait \
  --async
```

CI's variables
Before we can add the variables, we must create a project in CircleCI.
Log in with your Github or Bitbucket account (if you log in with the Github account, you will see the repositories that are hosted in Github, but if you log in with Bitbucket account, you will see the Bitbucket's repositories). After you were logged in, go to ADD PROJECT page and create a project for one of your repositories by clicking the Set Up Project button. On the next page, click the Start building button. For the first time, your project will not be built, but we will fix it later.

After the project was created, go to Settings > Projects and click on the gear button that is located on the right to project’s name. On the next page, go to the Environment Variables section.

Here you need to add 5 variables:
* DROPBOX_KEY — copy the Dropbox key,
* GCLOUD_SERVICE_KEY — copy the text that is inside the gcloud_key.json,
* GOOGLE_PLAY_KEY — copy the text that is inside the gplay_key.json,
* GOOGLE_PROJECT_ID — copy the Firebase project id,
* SIGNING_ARCHIVE_NAME — the name of the archive with the signing key and its credentials.

Config file
To make a workflow we need jobs that represent one particular step of the flow, like running unit tests, checkstyle and they can be reused.

Checkstyle job
I am using ktlint for code style, because me example project is writting in Kotlin, for Java you can use CheckStyle.

```yaml
jobs:
  #...
  code-style:
    <<: *defaults
    steps:
      - checkout
      - run:
          name: Check Kotlin code style
          command: ./gradlew ktlint
```

Static analyzer
```yaml
  code-quality:
    <<: *defaults
    steps:
      - checkout
      - run:
          name: Code quality
          command: ./gradlew detekt
```

Unit tests
```yaml
  unit-tests:
    <<: *defaults
    steps:
      - checkout
      - run:
          name: Unit tests
          command: ./gradlew testDebugUnitTest
```

UI tests
```yaml
  unit-tests:
    <<: *defaults
    steps:
      - restore_cache:
          key: jars-{{ checksum "build.gradle" }}-{{ checksum  "app/build.gradle" }}
      - run:
          name: Store service account
          command: echo $GCLOUD_SERVICE_KEY > ${HOME}/gcloud-service-key.json
      - run: 
          name: Authorize to gcloud
          command: gcloud auth activate-service-account --key-file=${HOME}/gcloud-service-key.json --project=${GOOGLE_PROJECT_ID}
      - run:
          name: Build apk
          command: ./gradlew assembleDebug
      - run:
          name: Build android test apk
          command: ./gradlew assembleDebugAndroidTest
      - run: 
          name: Run tests in Firebase Test lab
          comman: ./ci/run_firebase_tests.sh
      - save_cache:
          paths:
            - ~/.gradle
          key: jars-{{ checksum "build.gradle" }}-{{ checksum  "app/build.gradle" }} 
```

Lifehacks


Summary
