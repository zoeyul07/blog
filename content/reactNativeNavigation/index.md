---
emoji: ''
title: 'react-native: navigation'
date: '2022-11-09 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

# React Native Navigation 화면 이동

- `navigation.navigate('RouteName')` pushes a new route to the native stack navigator if it's not already in the stack, otherwise it jumps to that screen.
- We can call `navigation.push('RouteName')` as many times as we like and it will continue pushing routes.
- The header bar will automatically show a back button, but you can programmatically go back by calling `navigation.goBack()`. On Android, the hardware back button just works as expected.
- You can go back to an existing screen in the stack with `navigation.navigate('RouteName')`, and you can go back to the first screen in the stack with `navigation.popToTop()`.
- The `navigation` prop is available to all screen components (components defined as screens in route configuration and rendered by React Navigation as a route)

```jsx
// 스텍에 쌓고 이동하기 ( 뒤로가기시 페이지로 돌아옴 )
this.props.navigation.navigate('스크린 이름 ')

// 이전페이지로 돌아가기 ( 위의  navigate 를 이용하여  이동한 경우  가능  payLoad 가 2개 이상인 경우에 가능 하다 ) 
this.props.navigation.pop();

// 스텍의 제일 윗 페이지로 이동
this.props.navigation.popToTop()

// 새롭게 컴포넌트를 스텍이 쌓는다. ( 이전에 스텍에 쌓여있으면 그걸 불러온다 )
this.props.navigation.push();

// 스텍쌓지 않고 이동하기  ( 뒤로가기시  이전페이지로 돌아갈 수 없음 )
this.props.navigation.replace('스크린이름')

// 중첩된 네비게이터에서 이동
this.props.navigation.navigate('A_Screen', { screen: 'A_1Screen' })
```

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

## useNavigation vs navigation props

there isn't any benefit to use the `useNavigation` hook as the default method for accessing the `navigation prop` and I personally would strongly advise against it. [The official documentation](https://reactnavigation.org/docs/use-navigation/) is very clear on when to use the `useNavigation` hook.

> useNavigation is a hook which gives access to navigation object. It's useful when you cannot pass the navigation prop into the component directly, or don't want to pass it in case of a deeply nested child.
> 

If the component is defined as a `Screen` in the `Navigator` and the `navigation prop` is passed by it to the screen anyway, we should not create a dependency to a hook which we do not need.

We could even pass the `navigation prop` from the parent to its children if the children need it. Thus, there is no general need to use `useNavigation` at all and `certainly not as the default method`.

However, in some cases it is very inconvenient to pass the `navigation prop` very deep. That is why `useNavigation` exists.


### reference

[Is there any benefit of getting navigation prop using useNavigation hook in React Native?](https://stackoverflow.com/questions/71511898/is-there-any-benefit-of-getting-navigation-prop-using-usenavigation-hook-in-reac)

[reactNavigation](https://reactnavigation.org/docs/use-navigation-state/)