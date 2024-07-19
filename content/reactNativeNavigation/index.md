---
emoji: ''
title: 'react-native: navigation.getState() vs useNavigationState'
date: '2022-11-09 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

## How is `useNavigationState` different from `navigation.getState()`?[](https://reactnavigation.org/docs/use-navigation-state/#how-is-usenavigationstate-different-from-navigationgetstate)

The `navigation.getState()` function also returns the current [navigation state](https://reactnavigation.org/docs/navigation-state). The main difference is that the `useNavigationState` hook will trigger a re-render when values change, while `navigation.getState()` won't. For example, the following code will be incorrect:

```jsx
function Profile() {
  const routesLength = navigation.getState().routes.length; // Don't do this

  return <Text>Number of routes: {routesLength}</Text>;
}
```

In this example, even if you push a new screen, this text won't update. If you use the hook, it'll work as expected:

[Try this example on Snack](https://snack.expo.io/?platform=android&name=useNavigationState%20%7C%20React%20Navigation&dependencies=%40expo%2Fvector-icons%40*%2C%40react-native-community%2Fmasked-view%40*%2Creact-native-gesture-handler%40*%2Creact-native-pager-view%40*%2Creact-native-paper%40%5E4.7.2%2Creact-native-reanimated%40*%2Creact-native-safe-area-context%40*%2Creact-native-screens%40*%2Creact-native-tab-view%40%5E3.0.0%2C%40react-navigation%2Fbottom-tabs%406.3.1%2C%40react-navigation%2Fdrawer%406.4.1%2C%40react-navigation%2Felements%401.3.3%2C%40react-navigation%2Fmaterial-bottom-tabs%406.2.1%2C%40react-navigation%2Fmaterial-top-tabs%406.2.1%2C%40react-navigation%2Fnative-stack%406.6.1%2C%40react-navigation%2Fnative%406.0.10%2C%40react-navigation%2Fstack%406.2.1&hideQueryParams=true&sourceUrl=https%3A%2F%2Freactnavigation.org%2Fexamples%2F6.x%2Fuse-navigation-state.js)

```jsx
function Profile() {
  const routesLength = useNavigationState(state => state.routes.length);

  return <Text>Number of routes: {routesLength}</Text>;
}
```

So when do you use `navigation.getState()`? It's mostly useful within event listeners where you don't care about what's rendered. In most cases, using the hook should be preferred

[reactNavigation](https://reactnavigation.org/docs/use-navigation-state/)