---
emoji: ''
title: 'react-native: env'
date: '2022-11-29 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

# env 설정

## react-native-config

staging, production 환경 변수를 나누기 위해 react-native-config 패키지를 이용한다.

```
//패키지 설치
yarn add react-native-config
cd ios && pod install && cd ..

//auto link
react-native link react-native-config
```

```jsx
//Android

//app/build.gradle
//apply plugin: "com.android.application" 아래에 위치
project.ext.envConfigFiles = [
				//개발
        debug: ".env.staging",
				//배포
        release: ".env.production"
]
apply from: project(':react-native-config').projectDir.getPath() + "/dotenv.gradle"

//Build Variants에 debug, release 선택하여 실행
```

```jsx
//IOS

방법1
//env별로 scheme 생성
//원하는 환경의 scheme을 선택하여 빌드하면 된다.

1. Xcode의 Product > Scheme > Edit Scheme 을 열고 기본 Scheme이 선택된 상태에서 Duplicate Scheme을 클릭한다.
2. 복사된 Scheme의 이름을 ([앱이름].staging) 변경한다.
3. Product > Scheme > Edit Scheme 을 열고 [앱이름].staging을 선택한 뒤 Build > Pre-actions에서 New Run Script Action을 아래와 같이 추가해 준다.

# replace .env.development for your file
cp "${PROJECT_DIR}/../.env.production" "${PROJECT_DIR}/../.env"

"${SRCROOT}/../node_modules/react-native-config/ios/ReactNativeConfig/BuildXCConfig.rb" "${SRCROOT}/.." "${SRCROOT}/tmp.xcconfig"

4. 추가적으로 각 작업별로 Build Configuration을 추가해 줄 수 있는데, 기본적으로 debug, release를 선택할 수 있으며, staging 등 다른 환경의 configuration을 따로 설정해 준 경우 여기서 선택해 줄 수 있다.

방법2
//podfile에 다음 코드 추가
ENVFILES = {
  'Debug' => '$(PODS_ROOT)/../../.env.debug',
  'staging' => '$(PODS_ROOT)/../../.env.staging',
  'Release' => '$(PODS_ROOT)/../../.env.production',
}

post_install do |installer|
    installer.pods_project.targets.each do |target|
      target.build_configurations.each do |config|
        if target.name == 'react-native-config'
          config.build_settings['ENVFILE'] = ENVFILES[config.name]
        end
      end
    end
  end

==> 스키마를 여러개 생성해여 이동하며 테스트하기보다는 debug, release모드에 env를 지정하는게 이용하기
간편할 것 같아 방법 2로 진행

**주의**
react native 는 하나의 post_install 만 허용하므로 기존에 post_install이 있다면 기존것과 합친다.
```

```jsx
//root 에 파일 생성

//.env는 git에 올라가지 않으므로 새로 source code를 받은 사람들이 env가 필요하나는 것을 알려주기 위한 template
.env
//로컬 및 개발 버전
.env.debug
//stage 서버 버전
.env.staging
//release (production)
.env.production
```

## react-native-dotenv

- The variables will automatically be pulled from the appropriate environment and `development`is the default. The choice of environment is based on your Babel environment first and if that value is not set, your NPM environment, which should actually be the same, but this makes it more robust.
- In general, **Release** is `production` and **Debug** is `development`
- 여러 환경에서 해당 패키지를 이용하기 위해서는 dotenv-flow가 필요함 (패키지 gihub 문서 Readme 에서 확인할 수 있음)

  ```jsx
  //babel.config.js
  {
    "plugins": [
      ["module:react-native-dotenv", {
        "moduleName": "@env",
        "path": ".env",
        "safe": true,
        "allowUndefined": false,
        "verbose": false
      }]
    ]
  }

  //usage
  import {API_URL, API_TOKEN} from "@env"
  ```

### Reference

[react-native 공식문서](https://reactnative.dev/docs/security#storing-sensitive-info) 에서는 react-native-dotenv 와 React-native-config 를 권장하고 있음

[Security · React Native](https://reactnative.dev/docs/security#storing-sensitive-info)

[React Native 환경 설정하기](https://maruzzing.github.io/study/rnative/React-Native-%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0/)

[GitHub - luggit/react-native-config: Bring some 12 factor love to your mobile apps!](https://github.com/luggit/react-native-config)

[minify/packages/babel-plugin-transform-inline-environment-variables at master · babel/minify](https://github.com/babel/minify/tree/master/packages/babel-plugin-transform-inline-environment-variables)

[React Native 환경별 빌드 설정](https://velog.io/@ricale/React-Native-%EB%B9%8C%EB%93%9C-%ED%99%98%EA%B2%BD-%EB%B6%84%EB%A6%AC)
