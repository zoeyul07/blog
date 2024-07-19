---
emoji: ''
title: 'package manager: yarn berry & pnpm'
date: '2023-03-08 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

# 패키지 관리자(feat: yarn berry , pnpm)

node_modules 구조 하에서 모듈을 검색하는 방식은 기본적으로 디스크 I/O 작업이다. 이는 node_modules가 가진 문제이기 때문에 yarn classic과 npm 모두에 해당되는 내용

이처럼 모듈 탐색을 메모리 상에서 자료구조로 처리하지 않고 I/O로 직접 처리하다보니 추가적인 최적화가 어렵고 실제로 yarn 개발진은 이러한 이유들로 더 이상 최적화 할 여지가 없었다고 문서에서 밝히고 있다. yarn berry에서는 PnP 라는 기술을 통해 이를 개선한다.

### yarn berry(PnP)

- yarn, npm과 다르게 node_modules를 갖고 있지 않다.
- 대신 각각의 의존성들이 zip 파일로 압출되어 .yarn/cache에 ./pnp.js로 저장된다.
- node_modules는 사이즈와 복잡성 등 여러가지 이유로 commit되면 안되지만 zip 파일을 commit 하는 것은 간단하다.
- 그렇기 때문에 브랜치를 바꾼다고 하더라도 yarn install 을 할 필요가 없고, 이를 zero-installs 라고 한다.

but,

- React native는 PnP(Plug’n’Play) 사용 블가
- node_modules를 유지한다면 사용 가능
- 그렇기 때문에 node_modules가 차지하는 용량을 줄일수는 없음
- 그래도 모듈을 검색하는 방식은 개선할 수 있음

### pnpm

pnpm은 호이스트 방식 대신, 다른 dependencies를 해결하는 전략인 [**content-addressable storage**](https://pnpm.io/next/symlinked-node-modules-structure)를 사용했다. 이 방법을 사용하면, home 폴더의 글로벌 저장소 (**`~/.pnpm-store`**)에 패키지를 저장하는 중첩된 node_modules 폴더가 생성된다. 따라서 모든 버전의 dependencies은 해당 폴더에 물리적으로 한번만 저장되므로, single source of truth를 구성하고, 상당한 디스크 공간을 절약할 수 있다.

이는 node_modules의 레이아웃을 통해 이루어지고, **`symlinks`**를 사용하여 dependencies의 중첩된 구조를 생성한다. 여기서 폴더 내부의 모든 패키지 파일은 저장소에 대한 하드 링크로 구성되어 있다.

### Reference

[패키지 매니저, 그것이 궁금하다.](https://medium.com/zigbang/%ED%8C%A8%ED%82%A4%EC%A7%80-%EB%A7%A4%EB%8B%88%EC%A0%80-%EA%B7%B8%EA%B2%83%EC%9D%B4-%EA%B6%81%EA%B8%88%ED%95%98%EB%8B%A4-5bacc65fb05d)

- yarn berry

[npm, yarn, pnpm 비교해보기](https://yceffort.kr/2022/05/npm-vs-yarn-vs-pnpm)

[리멤버 웹 서비스 좌충우돌 Yarn Berry 도입기 - DRAMA&COMPANY](https://blog.dramancompany.com/2023/02/리멤버-웹-서비스-좌충우돌-yarn-berry-도입기/)

[node_modules로부터 우리를 구원해 줄 Yarn Berry](https://toss.tech/article/node-modules-and-yarn-berry)

[Plug'n'Play](https://yarnpkg.com/features/pnp#incompatible)

[Support for Yarn Plug 'n Play · Issue #27 · react-native-community/cli](https://github.com/react-native-community/cli/issues/27)

[How to use Yarn 3 with React Native (and how to migrate)](https://levelup.gitconnected.com/how-to-use-yarn-3-with-react-native-and-how-to-migrate-c5f108108533)

[Yarn Berry(v3) Workspace #4 - React Native 설정](https://velog.io/@projaguar/Yarn-Berryv3-Workspace-4-React-Native-%EC%84%A4%EC%A0%95)

- pnpm

[Motivation | pnpm](https://pnpm.io/motivation)

[runreactnative.dev](https://runreactnative.dev/posts/pnpm-react-native)
