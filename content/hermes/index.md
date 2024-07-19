---
emoji: ''
title: 'react-query'
date: '2023-02-21 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

## Hermes 란?

[Hermes](https://hermesengine.dev/)
 is an open source JavaScript engine designed to optimize performance by reducing app launch time and precompiling JavaScript into efficient bytecode.

### proguard 를 사용중이라면 주의

Also, if you're using ProGuard, you will need to add these rules in `proguard-rules.pro` :

```
-keep class com.facebook.hermes.unicode.** { *; }
-keep class com.facebook.jni.** { *; }

```

Next, if you've already built your app at least once, clean the build:

```
$ cd android && ./gradlew clean

```

That's it! You should now be able to develop and deploy your app as usual:

```
$ npx react-native run-android
```
### reference

[hermes](https://www.notion.so/Hermes-ffc3ef483a494b759c920f14bfa09c67?pvs=4)

[Using Hermes · React Native](https://reactnative.dev/docs/hermes)

[Using Hermes in React Native - LogRocket Blog](https://blog.logrocket.com/using-hermes-react-native/)