---
emoji: ''
title: 'react-native 배포 자동화: fastlane'
date: '2022-11-29 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

# 배포 자동화 (fastlane vs bitrise)

## ios

1.  Testflight + fastlane
2.  firebase app distribution + fastlane

## android

1.  google play console 내부 테스트
2.  firebase **App Distribution + fastlane —> 많이 사용하고 명령어 하나로 가능함**

## 설정

### fastlane 설치

```jsx

flastlane 설치
>brew install fastlane

프로젝트 폴더 안에서 다음 명령어 실행
Navigate to your iOS or Android app and run
> fastlane init

android/fastlane/Appfile
- 안드로이드의 경우 google cloud platform에서 받은 json key 파일 경로 설정 및 해당 경로에 파일 추가
해당 파일은 gitignore에 지정되어있으므로 아래에서 다운르도
>json_key_file(ENV['ANDROID_APP_GOOGLE_KEY'])
>package_name("") ,AndroidManifest 파일내에 있는 패키지 이름 설정

ios/fastlane/Appfile
- ios의 경우 xcode에서 app bundle identifier 확인 후 지정 및 apple id 설정
각자의 애플 아이디를 환경 변수로 설정해서 사용한다. 아래 env설정 쪽 확인
>app_identifier("") # The bundle identifier of your app
>itc_team_id(ENV["ITC_TEAM_ID"]) # App Store Connect Team ID
>team_id(ENV["TEAM_ID"]) # Developer Portal Team ID
>apple_id(ENV["APPLE_ID"])

Fetch your app metadata
>fastlane supply init
```

### 프로젝트 폴더 내 gemfile에 코드 추가

```jsx
Install Bundler by running gem install bundler

Create a ./Gemfile in the root directory of your project with the content
source "https://rubygems.org"

gem "fastlane"
Run bundle update and add both the ./Gemfile and the ./Gemfile.lock to version control
Every time you run fastlane, use bundle exec fastlane [lane]
On your CI, add bundle install as your first build step
To update fastlane, just run bundle update fastlane
```

### firebase 연결

```jsx
/*firebase 연결*/

//plugin 설치
fastlane add_plugin firebase_app_distribution

//login
>bundle exec fastlane run firebase_app_distribution_login

//firebase cli global 로 설치
>npm install -g firebase-tools

//firebase login
>firebase login

ID : 
PW : 
```

### fastlane 에서 env 사용

```jsx
//use env from react-native-config

I managed to get react-native-config picking up the correct config file using the environment variables feature provided by fastlane:

Add a matching .env file for each of your stages in your fastlane folder. e.g. .env.beta .env.production.
In the fastlane config, add the ENVFILE variable with the corresponding react-native-config env file, e.g. ENVFILE=.env.beta
Now running fastlane [lane] --env beta will use the .env.beta fastlane-specific config.

This will add the ENVFILE=.env.beta variable to the build environment, controlling which file react-native-config will pick up and use at runtime.

Complete example below:

[project-dir]/fastlane/.env.beta

ENVFILE=.env.beta
[project-dir]/.env.beta

APP_STAGE=stage
Running fastlane using fastlane [lane] --env beta will result in react-native-config using the .env.beta file from the root of your React Native app when populating the object for use at runtime, in this case the Config will contain { APP_STAGE: 'stage' }.

It's also possible to combine these two files and use one env file for each stage across fastlane and react-native-config, but I preferred the separation between build-time and runtime environment variables.

> fastlane distribute --env production
>>> 그리고 위와 같이 fastlane 명령어 실행시 --env [사용하려는 env 뒤에 이름(ex. production)] flag를 붙여서 실행
		flag를 붙여서 사용해면 해당 env 파일을 가져오게 되고 ENV["FIREBASE_APP_ID"]와 같이 사용 가능

스토어 release 명령어
> fastlane release --env production
braze custom event 추가, 건기식 버튼 문구 동적 변경
```

### Android env 파일

```jsx
*/android/fastlane/.env

FIREBASE_APP_ID=
SLACK_URL=
TESTER=

# Android 풀경로를 사용
ANDROID_APP_SIGNING_KEY=
ANDROID_APP_SIGNING_STORE_PASSWORD=
ANDROID_APP_SIGNING_KEY_ALIAS=
ANDROID_APP_SIGNING_KEY_PASSWORD=
ANDROID_APP_GOOGLE_KEY=
ANDROID_APP_AAB_PATH=.
```

### iOS env 파일

```jsx
*/ios/fastlane/.env
FIREBASE_APP_ID=
SLACK_URL=
ITC_TEAM_ID=
TEAM_ID=
APPLE_ID=[본인의 apple id] //apple env에는 아래 추가
```

### **Codesigning**

**Using [_match_](https://fastlane.tools/match)**

The concept of [_match_](https://fastlane.tools/match) is described in the [codesigning guide](https://codesigning.guide/).

With [_match_](https://fastlane.tools/match) you store your private keys and certificates in a git repo to sync them across machines. This makes it easy to onboard new team-members and set up new Mac machines. This approach [is secure](https://docs.fastlane.tools/actions/match/#is-this-secure) and uses technology you already use.

Getting started with [_match_](https://fastlane.tools/match) requires you to revoke your existing certificates.

Make sure to follow [Setting up your Xcode Project](https://docs.fastlane.tools/codesigning/xcode-project/) to set up your project properly.

**Using [_cert_](https://fastlane.tools/cert) and [_sigh_](https://fastlane.tools/sigh)**

If you don't want to revoke your existing certificates, but still want an automated setup, [_cert_](https://fastlane.tools/cert) and [_sigh_](https://fastlane.tools/sigh) are for you.

- [_cert_](https://fastlane.tools/cert) will make sure you have a valid certificate and its private key installed on the local machine
- [_sigh_](https://fastlane.tools/sigh) will make sure you have a valid provisioning profile installed locally, that matches the installed certificate

Add the following lines to your `Fastfile`

```
lane :beta do
  get_certificates# invokes cert
  get_provisioning_profile# invokes sigh
  build_app
end

```

Make sure to follow [Setting up your Xcode Project](https://docs.fastlane.tools/codesigning/xcode-project/) to set up your project properly.

⇒ cert& sigh 방식 이용

다음과 같이 사용하므로 apple developer에서 확인해서 provisioning profile 사용하고 있는 맥에 추가 필요

```jsx
provisioningProfiles: {
          "" => "",
        }
```

fastfile 내에 저렇게 지정하면 manual 로 선택해서 가져오므로 xcode에서 auto managing 설정해놨던 것 끌 필요 없음

### fastlane 폴더 구조

```jsx
fastlane
	metadata    -> 앱스토어 메타데이터 정보들이 담긴 폴더
	screenshots -> 앱스토어 스크린샷이 담긴 폴더
	Appfile     -> 번들ID, 애플ID, 팀ID 등의 정보가 담긴 파일
	Deliverfile -> 앱스토어 메타데이터 저장을 위한 파일
	Fastfile    -> 자동화할 명령어들이 담긴 파일

처음 fastlane 초기화시 메타데이터를 받아오지만 수동으로 받아오기 위해서는 아래 명령어를 입력

앱스토어 스크린샷 다운로드: fastlane deliver download_screenshots
앱스토어 메타데이터 다운로드: fastlane deliver download_metadata
```

- upload_to_app_store: 앱스토어에 업로드합니다.
  - force: HTML report를 스킵합니다.
  - skip_screenshots: 스크린샷 업로드를 스킵합니다. 기존에 앱스토어에 올라가있는 스크린샷을 사용하기 때문에 true로 설정합니다.
  - skip_metadata: 앱스토어 메타데이터를 스킵합니다. 새로운 버전에대한 정보를 업데이트 하기 위해서 false로 설정합니다. (/fastlane/metadata 폴더에서 정보 수정)
  - submit_for_review: 업로드가 완료되면 자동으로 심사까지 제출합니다.
  - automatic_release: 심사가 완료되면 자동으로 배포할지 수동으로 배포할지 선택합니다.

### Android 앱 배포 스크립트

```jsx
//build_app 옵션
increment_build_number: 자동으로 빌드넘버를 증가시킵니다.
build_app: 앱을 빌드합니다. 빌드할 워크스페이스, 스키마를 지정해줍니다
upload_to_play_store: 앱스토어에 업로드합니다.

track: 사용할 애플리케이션의 트랙입니다. 사용 가능한 기본 트랙: 프로덕션, 베타, 알파, 내부
json_key: Google 인증에 사용되는 서비스 계정 JSON이 포함된 파일의 경로
aab: 업로드할 AAB 파일의 경로
skip_upload_apk: APK 업로드 건너뛰기 여부
skip_upload_aab: AAB 업로드 건너뛰기 여부
skip_upload_metadata: 메타데이터 업로드 건너뛰기 여부, 변경 로그는 포함되지 않음
skip_upload_changelogs: 변경 로그 업로드 건너뛰기 여부
skip_upload_images: 이미지 업로드 건너뛰기 여부, 스크린샷은 포함되지 않음
skip_upload_screenshots: SCREENSHOTS 업로드 건너뛰기 여부

validate_only: 실제로 게시하지 않고 Google Play에서만 변경사항을 확인합니다.
```

### IOS 앱 배포 스크립트

```jsx
//build_app 옵션
increment_build_number: 자동으로 빌드넘버를 증가시킵니다.
build_app: 앱을 빌드합니다. 빌드할 워크스페이스, 스키마를 지정해줍니다
upload_to_app_store: 앱스토어에 업로드합니다.
force: HTML report를 스킵합니다.
skip_screenshots: 스크린샷 업로드를 스킵합니다. 기존에 앱스토어에 올라가있는 스크린샷을 사용하기 때문에 true로 설정합니다.
skip_metadata: 앱스토어 메타데이터를 스킵합니다. 새로운 버전에대한 정보를 업데이트 하기 위해서 false로 설정합니다. (/fastlane/metadata 폴더에서 정보 수정)
submit_for_review: 업로드가 완료되면 자동으로 심사까지 제출합니다.
automatic_release: 심사가 완료되면 자동으로 배포할지 수동으로 배포할지 선택합니다.

force: fastlane가 생성하는 HTML report를 생성하지 않도록 합니다.(true)
reject_if_possible: 심사 대기중인 버전이 있다면 취소합니다(true)
skip_metadata: 앱 스토어의 정보를 등록할지 결정합니다. 자동 배포를 할 때, 버전의 수정 내용을 작성해야 하므로 이 정보를 등록하도록 설정해야합니다.(false)
skip_screenshots: 저는 이미 배포한 앱에 대해서 자동 배포를 적용하고 있습니다. 따라서 스크린 샷을 다시 업로드할 필요가 없습니다(true)
languages: 현재 스토어에 등록된 앱의 지역화를 설정합니다. 사용 가능한 언어는 ar-SA, ca, cs, da, de-DE, el, en-AU, en-CA, en-GB, en-US, es-ES, es-MX, fi, fr-CA, fr-FR, he, hi, hr, hu, id, it, ja, ko, ms, nl-NL, no, pl, pt-BR, pt-PT, ro, ru, sk, sv, th, tr, uk, vi, zh-Hans, zh-Hant 이며 자세한 내용은 공식 홈페이지를 참고하시기 바랍니다(공식 홈페이지)
release_notes: iOS는 앱을 재배포할 때, Release notes를 꼭 작성해야합니다. 제가 사용하는 스크립트는 3개국어를 지원하는 앱이므로 default와 3개 국어에 해당하는 Release notes를 작성하였습니다.
submit_for_review: 앱 심사에 제출하도록 합니다.(true)
automatic_release: 심사후 앱을 자동으로 배포하도록 설정합니다. 이 값이 설정되지 않으면, 앱 심사를 통과한 후 개발자가 수동으로 배포해야 합니다.(true)
submission_information: 배포전에 암호화, 광고 포함 여부등을 물어보는 옵션을 설정합니다.

submission_information: 광고식별자(IDFA), 암호화, 콘텐츠 권한을 설정합니다
	ㄴadd_id_info_serves_ads: 광고 제공 여부
	ㄴadd_id_info_tracks_action: 특정 행위 추적
	ㄴadd_id_info_tracks_install: 설치 추적
	ㄴadd_id_info_uses_idfa: IDFA 사용 여부

```

```jsx
//android update version 로직
def updateVersion(options)
    if options[:version]
      version = options[:version]
    else
      version = prompt(text: "버전을 입력해주세요 (ex 1.0.0) : ")
    end

    re = /\d+.\d+.\d+/
    versionNum = version[re, 0]

    if (versionNum)
      increment_version_number(
        version_number: versionNum
      )
    elsif (version == 'major' || version == 'minor' || version == 'patch')
      increment_version_number(
        bump_type: version
      )
    elsif (version == 'maintain')
    else
      UI.user_error!("[ERROR] Wrong version!!!!!!")
    end
  end

//실행 명령어
fastlane deploy version:major // 2.0.0
fastlane deploy version:minor // 1.3.0
fastlane deploy version:patch // 1.2.4

여기서 추가적으로 maintain이라는 케이스를 추가했는데, increment_version_number를 실행하지 않는 케이스이다.
maintain을 version으로 주면, 이전 버전과 동일한 number를 가지고, build number만 증가하게 된다.
//현재 버전 1.2.3
fastlane deploy version:maintain

//android 도 다음과 같이 동일하게 실행
//PROJECT_ROOT/android

fastlane deploy version:1.1.1
fastlane deploy version:major
fastlane deploy version:minor
fastlane deploy version:patch
fastlane deploy version:maintain
```

### reference

[자동 version number up 관련 문서](https://docs.fastlane.tools/actions/increment_version_number/)

[fastlane 공식문서](https://docs.fastlane.tools/getting-started/android/running-tests/)

[How do I use fastlane with react-native-config ? · Issue #13494 · fastlane/fastlane](https://github.com/fastlane/fastlane/issues/13494)

[올리브영 안드로이드 테스트앱 자동배포하기 | 올리브영 테크블로그](https://oliveyoung.tech/blog/2021-07-15/Automatic-Distribution-AOS-Test-App-To-Fastlane/)

[안드로이드 앱 베타 테스트 하기](https://brunch.co.kr/@shindesign/107)

[구글플레이 내부테스트 기능을 활용하여 내부테스트 하기](https://trend21c.tistory.com/2181)

[앱 개발자분들 테스트랑 배포 관리 어떻게 하시나요? : 클리앙](https://www.clien.net/service/board/cm_app/16977334)

[react native fastlane을 통한 배포 자동화](https://evanjin.oopy.io/react-native/fastlane)

[iOS - 배포 자동화, Fastlane 시작부터 적용까지](https://medium.com/hcleedev/ios-%EB%B0%B0%ED%8F%AC-%EC%9E%90%EB%8F%99%ED%99%94-fastlane-%EC%8B%9C%EC%9E%91%EB%B6%80%ED%84%B0-%EC%A0%81%EC%9A%A9%EA%B9%8C%EC%A7%80-3d9107cdc3b4)

[[fastlane] \*8. fastlane과 Bitrise를 이용한 자동 배포 구축 방법](https://ios-development.tistory.com/749)

[[iOS] Fastlane을 적용하여 배포 자동화하기 - Kyungmo's Blog](https://kyungmosung.github.io/2021/11/15/fastlane/)

[[iOS] fastlane 이용한 배포 자동화 (Firebase App Distribution 편)](https://velog.io/@parkgyurim/iOS-fastlane-Firebase-App-Distribution)

[[React Native] Fastlane match를 사용한 iOS 인증서 동기화](https://millo-l.github.io/ReactNative-fastlane-match/)

[Fastlane 배포하기](https://velog.io/@phs880623/Fastlane-배포하기)

[[React Native] Fastlane match를 사용한 iOS 인증서 동기화](https://millo-l.github.io/ReactNative-fastlane-match/)

[React Native) fastlane으로 firebase app distribution 자동화하기](https://velog.io/@2ast/React-Native-fastlane%EC%9C%BC%EB%A1%9C-firebase-app-distribution-%EC%9E%90%EB%8F%99%ED%99%94%ED%95%98%EA%B8%B0#%ED%8A%B8%EB%9F%AC%EB%B8%94-%EC%8A%88%ED%8C%85)

[Release v0.3.2 · fastlane/fastlane-plugin-firebase_app_distribution](https://github.com/fastlane/fastlane-plugin-firebase_app_distribution/releases/tag/v0.3.2)

[React Native) Fastlane으로 배포를 자동화해보자 (IOS)](https://velog.io/@2ast/React-Native-Fastlane%EC%9C%BC%EB%A1%9C-%EB%B0%B0%ED%8F%AC%EB%A5%BC-%EC%9E%90%EB%8F%99%ED%99%94%ED%95%B4%EB%B3%B4%EC%9E%90-IOS)
