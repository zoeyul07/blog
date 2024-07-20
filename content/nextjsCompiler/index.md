---
emoji: ''
title: 'nextjs compiler: babel vs swc'
date: '2023-04-06 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

- next 12 버전대에서 변경됨

NextJS 12 버전부터 [SWC](https://swc.rs/)를 Next.js Compiler 로 소개한다.

SWC는 Rust 기반의 컴파일러로 싱글 코어에서 babel 보다 20배, 4 코어에서 babel 보다 70배 빠르다고 소개하고 있다.

### babel의 한계점

1. 배포 뒤, 개발자들이 babel이 변경한 코드를 이해하기 어려워 할 수있다.
2. 코드의 크기가 늘어난다.
3. polyfill을 사용해서 babel이 지원해주지 않는 범위까지 변환해주어야 한다.
4. 다른 컴파일러보다 시간이 오래 걸린다.

### SWC 컴파일러

SWC(Speedy Web Compiler)는 Rust로 작성된 컴파일러이다. 저급 프로그래밍 언어인 Rust로 작성되었기 때문에 컴파일시 매우 빠르게 동작한다. 개발자들은 웹사이트 최적화, 번들링, 컴파일 등 다양한 기능을 위해 SWC를 사용할 수 있다.

**SWC의 장점**1. 확장성: 개발자들은 라이브러리를 fork 해올 필요 없이 Next JS에 미리 설치된 SWC를 사용할 수 있다.2. WebAssembly: Rust의 WASM(WebAssembly) 지원으로 어떤 종류의 플랫폼에서도 Next JS 개발을 할 수 있다.3. 성능: 다른 컴파일러들보다 훨씬 좋은 성능을 제공한다.4. 커뮤니티 지원: 빠르게 성장하는 커뮤니티를 가지고 있다.

### reference

[nextjs 12에서 emotion과 함께하는 tailwindcss - call5h5ong님의 블로그 - 인프런 | 커뮤니티](https://www.inflearn.com/blogs/2858)

[Next JS가 Babel을 SWC로 대체한 이유](https://velog.io/@kwonhygge/Next-JS%EA%B0%80-Babel%EC%9D%84-SWC%EB%A1%9C-%EB%8C%80%EC%B2%B4%ED%95%9C-%EC%9D%B4%EC%9C%A0)

[NextJS Babel 에서 SWC 전환 하는 과정에서 겪은 에러](https://velog.io/@ansrjsdn/NextJS-Babel-%EC%97%90%EC%84%9C-SWC-%EC%A0%84%ED%99%98-%ED%95%98%EB%8A%94-%EA%B3%BC%EC%A0%95%EC%97%90%EC%84%9C-%EA%B2%AA%EC%9D%80-%EC%97%90%EB%9F%AC)

[NextJS Babel에서 SWC로 이사가기 1](https://kir93.tistory.com/entry/NextJS-Babel%EC%97%90%EC%84%9C-SWC%EB%A1%9C-%EC%9D%B4%EC%82%AC%EA%B0%80%EA%B8%B0)