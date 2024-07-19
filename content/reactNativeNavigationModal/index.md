---
emoji: ''
title: 'modal vs react-native-navigation'
date: '2022-12-01 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

# modal vs react-native-navigation

## modal

When the modal becomes visible, the modal screen component is *mounted*
, and when the modal is hidden, the screen is *updated*
 and *unmounted*
. These are the three phases in the component lifecycle: mounting, updating, and unmounting. A component is mounted when “it is being created and inserted into the DOM” 

[1]. An update is triggered when the component’s props or state are changed. Unmounting is when a component is being removed from the DOM. Executing a phase executes the methods associated with it. 

[2], provides an overview on the methods for each phase.  demonstrates their order of execution as the screen is mounted, updated, and unmounted each time the modal becomes visible and then hidden.


## react-native-navigation

provides routing and navigation for react native apps. The main view of the app is changed when navigating between screens

Navigating to screens using the `react-navigation`
 library behaves differently to modals. A screen is only mounted *the first time*
 that you navigate to a screen. Navigating away does not unmount the screen. Therefore, navigating back to the screen later does not mount it again.

Therefore, navigation includes an inherent performance advantage over modals. This is especially apparent with complex screen components that process many child components and calculations while mounting.

## conclusion

A screen can be brought to the foreground as a child window using a modal or by navigating to the screen and changing the main view of the app. The former iterates through the entire component lifecycle each time that the modal becomes visible and then hidden. The latter only mounts the screen component the first time that the app navigates to it. Navigating away from the screen does not unmount the component. Therefore, navigating to that screen in the future does not mount it again. Thus, navigation generally performs better than modals.

Due to this difference, two rules of thumb apply when designing an app.

1. Screens used with `react-native-modal` should be “lightweight”.
2. Any complex screens that have many children components and/or involve complex calculations should use `react-navigation` instead.

**Notes:** This is the default behavior in `react-navigation` and may be changed using the `unmountInactiveRoutes` config. You can read more about this in [the docs](https://reactnavigation.org/docs/en/drawer-navigator.html).


### reference

[Optimizing React Native - Part 1: Modals vs. Navigation](https://medium.com/@amismail/optimizing-react-native-part-1-modals-vs-navigation-18d074a771ce)

[React native modal vs react-navigation modal, which is more performant?](https://stackoverflow.com/questions/71549636/react-native-modal-vs-react-navigation-modal-which-is-more-performant)