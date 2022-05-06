---
emoji: ''
title: 'react native'
date: '2020-10-06 22:00:00'
author: seoyul
tags: frontEnd
categories: FrontEnd
---

## React Native
### Expo vs RN CLI
#### React Native CLI
- native file들을 더 많이 컨트로로 하고 싶을 때 사용한다.

- React Native CLI를 사용하면 xcode 파일을 생성하고, 안드로이드 파일을 생성하여 양쪽 파일들이 리엑트 네이트브에서 실행된다.

- 양쪽 파일에 다 접근할 수 있기 때문에 어떤 때에는 둘을 합칠수도 있다.

#### Expo
- 모든 native file들은 숨기고 모든것을 관리한다.

- react native cli에 비하여 빠르게 핸드폰에서 앱을 테스트할 수 있다.

- 앱을 build하는 방식, simulation, react native update를 처리해주며, react native에 비해 더 많은 모듈들이 있다.

- expo는 create-react-app과 같으며 그 위에 더 많은 것들을 제공해준다.

- CLI에 비해 수동적으로 진행해야하는 작업이 덜하고 시작하는데 훨씬 빠르다.

- native file에 대한 접근은 할 수 없다.

### Starting React Native with Expo
- 원하는 경로로 가서 다음 명령어를 실행한다.

```
expo init AwsomeProject
```
```
// 자동으로 expo dev tool을 열어준다.
npm start
```

- expo dev tool이 열리면 핸드폰으로 QR코드를 스캔하면 핸드폰에서 앱을 실행해볼 수 있다.

- 코드 수정 후 저장하면 자동으로 리프레쉬 되며(Live Reload) 변경된다.

- 핸드폰에서 개발자 메뉴에 접근하고 싶다면 핸드폰을 흔들면 되며 시뮬레이터에서 개발자 메뉴를 열려면 command+D를 누른다.

### React Native는 어떻게 동작하는가?
- 리액트 네이티브는 자바스크립트와 폰을 연결하기 위해서 브릿지가 필요한데, 다음 코드에서의 Text, View가 이 브릿지와 같은 역할을 한다.
```
import {StyleSheet, View, Text} from "react-nraive";
```

- 여기서 View는 리액트에서의 div와 같은 역할을 한다. 모든걸 View 안에 집어 넣어야 한다.

- 리액트 네이티브에는 그만의 규칙들이 있는데, 웹사이트에서 텍스트를 넣을 때 사용하는 span의 경우에는 Text 를 사용하며 이런 룰이 존재하는 이유는 브릿지 때문이다.

- 리액트 네이티브는 CSS 엔진을 구현했기 때문에 flex box와 같은 것들을 사용할 수 있으며 차이점은 자바스크립트 오브젝트와 같이 사용해야한다는 점이다.

```
export default function App() {
  return (
    <View style={styles.container}>
      <Text>Hello!</Text>
      <StatusBar style="auto" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
   // color: "white",
    justifyContent: 'center',
  },
});
```

- 리액트 네이티브는 모든게 웹사이트의 css 처럼 작동하지 않는다. 위의 container 스타일에 color를 white로 준다고 해도 Text의 hello는 white로 바뀌지 않는다.(웹사이트였다면 텍스트는 부모의 색을 얻었을 것이다.)

### 레이아웃에 대한 규칙
- 리액트 네이티브에서는 default로 flex-direction이 column이다. (웹사이트는 defualt로 flex-direction이 row이다.)

- flex:1 은 모든 공간을 사용한다는 것을 의미한다.

```
const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  yellowView: {
  	flex:1
  },
  blueView: {
  	flex:1
  }
});
```
- 만약 부모 container 안에 각자의 view의 flex가 1이라면 각자가 최대한 차지할 수 있는 만큼 전체 공간에서 반반씩 차지하게 되며, blueView의 flex가 2라면 yellow는 삼분의 일, blue는 삼분의 2의 공간을 차지하게 된다.

- flex-direction, justify-content, align-items, flex-wrap, no-wrap도 똑같이 동작한다.

- width, height를 많이 쓸 필요가 없이 flex를 사용하면 핸드폰의 크기에 관계 없이 맞춰진다.

### 초기 셋팅
## React Native

- [React Native에서 필요한 환경 설정](https://firework-ham.tistory.com/104)
- Expo & React Native CLI → React Native CLI 사용
    - React Native 공식문서 - [Getting Started](https://reactnative.dev/docs/0.8/getting-started)
    - [리액트 네이티브(react native) 두 가지 방법 #2 React Native CLI](https://velog.io/@max9106/React-Native-리액트-네이티브react-native-두-가지-방법-2-React-Native-CLI-bmk0gz4izg)
- cocoapods
    - [CocoaPods 설치 및 Pod 설치](https://blog.yagom.net/534/)
    - [CocoaPod 설치 및 사용법](https://zetal.tistory.com/entry/CocoaPod-설치-및-사용법)

### IOS

- xcode
    - [Xcode ios Simulator 설치](https://velog.io/@skh417/Tips-Xcode-ios-Simulator-설치)
- xcode 설치 후 app/ios 폴더 안에서 pod install 실행

### Android

- Android Studio
    
    
    - [안드로이드 에뮬레이터에서 AVD 만들고 실행하기](https://recipes4dev.tistory.com/145)
- app/android 폴더 안에서 [local.properties](http://local.properties) 파일 생성 후 sdk.dir 경로 지정

```jsx
//location은 android studio에서 configure -> SDK manager에서 확인 가능
sdk.dir = /Users/USERNAME/Library/Android/sdk
```

- ~/.bash_profile이나 ~/.zprofile에 다음을 추가

```jsx
export ANDROID_HOME=/Users/USERNAME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

```toc

```
