---
emoji: ''
title: 'React Native Components'
date: '2022-07-01 22:00:00'
author: seoyul
tags: frontEnd
categories: FrontEnd
---

## React Native Component
- HTML에서 사용하는 tag 들을 어떻게 대체하여 사용할 수 있는지, react native에서 사용할 수 있는 컴포넌트들에 어떤 것들이 있는지 알아보자

### div -> View

### img -> Image
- svg가 아닌 png 이미지 이용시 source props를 통해서 사용할 수 있다.

### span,p -> Text
- react native 에서 텍스트 보여주기 위해서는 반드시 Text 컴포넌트를 이용해야한다.
- allowFontScaling props를 통해서 휴대폰 설정의 폰트크기가 앱에 영향을 미치지 않도록 설정할 수 있다.
- includeFontPadding props를 통하여 폰트에 extra padding이 들어가지 않도록 방지할 수 있다.

### ul/ol, li -> FlatList

### button -> Button

### input -> TextInput
- 휴대폰 키보드를 통해 텍스트를 입력할 수 있는 컴포넌트

### ScrollView vs FlatList vs SectionList
- FlatList 와 SectionList는 현재 화면에 보여지는 요소들만을 렌더링하며 데이터가 많은 리스트를 보여줄 때 사용한다.

- ScrollView는 모든 요소를 한번에 렌더링하고 표현하는 반면에 FlatList는 한번에 모든 데이터를 렌더링하지 않고 화면에 보여지는 부분 혹은 설정한 수의 데이터 만큼만 렌더링을 하기 때문에 ScrollView는 출력해야하는 데이터가 많지 않거나 고정적일떄, FlatList는 데이터가 많거나 가변적이여서 예측할 수 없을 경우에 사용하여 ScrollView를 사용하는 것에 비해 퍼포먼스를 높일 수 있다. 또한 화면에서 벗어날 때 요소들이 삭제되기 때문에 메모리와 프로세스 시간을 줄일 수 있다.

- FlatList는 PureComponent이기 때문에 renderItem의 상태 변화를 알지 못하므로 extraData props를 통해서 해당 renderItem의 상태변화를 알고 리렌더링을 할 수 있다.

- SectionList는 FlatList와 동일하지만 Section 별로 나누어 표현할 수 있는 컴포넌트이다.

### TouchableOpacity vs TouchableWithoutFeedback vs TouchableHighlight 
- TouchableOpacity를 이용하면 터치했을 경우에 보여줄 투명도를 설정할 수 있다.
- TouchableWithoutFeedback을 이용하면 터치했을 경우에 아무효과도 적용하지 않는다.(TouchableOpacity에서 activeOpacity를 1로 두게되면 TouchableWithoutFeedback과 비슷하게 이용할 수 있다.)
- TouchableHighlight를 이용하면 터치했을 때 배경색을 변경할 수 있다.

### Pressable vs Touchable
- Pressable은 Touchable에 비해 press interaction의 다양한 스텝을 감지할 수 있다.
- onPressIn, onPressOut, onLongPress... 및 hover, focus등이 지원된다.


### Modal vs react-native-modal vs react-navigation modal
- React Native에서 제공하는 Modal 컴포넌트를 content를 기본적인 modal 형태로 보여줄 수 있다.
- react-native-modal은 React Native에서 제공하는 Modal 컴포넌트를 확장한 것으로 animation, custom style을 더 추가적으로 활용이 가능하다.
- Modal은 screen 컴포넌트가 보이게되면서 mounted되고 props가 바뀔때 update 되며, 컴포넌트가 dom에서 제거될 때 unmounted 된다.
- react-navigation을 통해 modal을 이용하면 routing, navigation이 된다. screen 간에 navigating을 하며 app의 main view가 바뀌게된다. screen을 navigating 하는 것은 modal과는 다르게 동작하는데, screen은 맨 처음에 navigation을 했을때만 mounted 되며 다른 화면으로 navigating 하는것은 unmount 되지 않는다. 그렇기 때문에 다시 화면으로 돌아온다고 해도 또 다시 mounted 되지 않는다.
- 그렇기 때문에 react-navigation은 mounting하는 동안 여러 자식컴포넌트나 계산이 필요한 복잡한 process를 실행하게 될 때 modal을 이용하는 것보다 performance 적으로 이점이 있다.

### ActivityIndicator
- circular loading indicator를 제공한다.

### ImageBackground
- web에서 background-image를 사용하는 것과 같은 기능으로, Image 컴포넌트와 같은 props를 받는다.

### KeyboardAvoidingView
- 이 컴포넌트는 virtual keyboard가 보여질 동안 View가 정상적으로 보여지기 위해(keyboard에 의해 view가 가려지는 것을 방지하기 위해) keyboard height를 기반으로 자동으로 height, position, bottom padding을 조정해준다.

### RefreshControl
- ScrollView 나 ListView안에서 pull to refresh를 사용할 수 있게 해준다.

### Switch 
- toggle switch 를 구현할 수 있다.








***
[React Native 공식문서](https://reactnative.dev/docs/components-and-apis)
[Optimizing React Native: Modals vs Navigation](https://medium.com/@amismail/optimizing-react-native-part-1-modals-vs-navigation-18d074a771ce)
```


```toc

```
