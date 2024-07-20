---
emoji: ''
title: 'nextjs 페이지 이동'
date: '2023-04-18 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

### 1. Link

```jsx
import Link from "next/link";
...
// path
<Link href={`/detail/${id}`}>
  <a>item name</a>
</Link>

// object
<Link href={{ pathname: "/about", query: { name: "song" } }}>
    <a>item name</a>
</Link>
```

- Link 태그에 하여 가고자 하는 주소를 넣습니다. (a 태그의 href와 동일합니다.)
    
    href 속성을 추가
    
- 안쪽에는 a 태그로 감싸줍니다.
- className과 같은 속성들을 a 태그에 추가합니다.
- 페이지  페이지가 전환됩니다.
    
    새로고침 없이
    

Link는 Client-side navigation으로 자바스크립트로 페이지 전환이 이루어집니다.

기본 navigation보다 빠르며 SPA의 특성을 유지합니다.

프러덕션 빌드에서 Link 요소가 뷰포트에 나타나면 넥스트는 백그라운드에서 해당 페이지의 코드를 자동으로 설정합니다.

Link와 연결된 페이지를 미리 로드하여 링크 클릭 시 빠르게 전환이 가능합니다.

이것이 **pre-fetching** 입니다.

**a 태그가 커스텀 컴포넌트일 경우?**

> Next.js 공식 문서 - next/link 부분
> 

만약 styled-components와 같이 a 태그가 커스텀 컴포넌트일 경우 Link 태그에 passHref 속성을 주어야 합니다.

이 속성이 없을 경우 a 태그에는 href 속성이 없어 접근성과 SEO에 영향을 줍니다.

### 2. Router

```jsx
import Link from "next/link";
import Router, { useRouter } from "next/router";
...
const router = useRouter();
...
// Router
<div onClick={() => Router.push("/about")}>Router</div>

// router
<div onClick={() => router.push("/about")}>router</div>// object
<div
    onClick={() =>
      router.push({ pathname: "/about", query: { name: "song" } })
    }
>
	router
</div>
```

라우터를 이용하여 페이지를 전환합니다.

react-router-dom의 history.push()와 유사합니다.

- 크롤러가 링크를 감지하지 못하여 SEO가 좋지 않을 수도 있습니다.
- <Link>는 클릭 시 바로 페이지가 전환되지만 라우터는 합니다.
    
    로직을 처리 후 원하는 시점에서 전환이 가능
    
- 
    
    외부 URL을 사용할 경우 window.location 혹은 a 태그를 사용해야 합니다.
    

router에는 push 말고도 많은 기능들이 있습니다.

그중 몇 가지 소개해드리겠습니다.

**1. router.push**

페이지가 이동되며 히스토리 스택이 쌓입니다.

Main -> Login -> List에서 마이페이지로 push 하면 Main -> Login -> List -> Mypage

**2. router.replace**

페이지가 이동되며 히스토리 스택이 교체됩니다.

Main -> Login -> List 에서 마이페이지로 replace 하면 Main -> Login -> Mypage

**3. router.back**

뒤로 가는 기능이며 "window.history.back()"과 동일합니다.

**4. router.reload**

새로고침 기능이며 "window.location.reload()"와 동일합니다.

## Route vs Link vs a

### 1. Route

router.push()를 이용하는 경우는 window.location과 비슷하게 동작합니다. <a>태그를 만들지 않기때문에 검색엔진최적화(SEO)을 신경쓰고 있다면(아마 Next.js를 사용하는 이상 대부분 신경을 쓰지 않을까요) 해당 링크는 크롤링되지 않아서 SEO에 불리합니다. 대부분 onClick과 같은 이벤트 핸들러와 같이 사용됩니다.

### 2. <Link>

하지만 <Link>태그는 <a>태그를 생성하기 때문에 웹사이트가 크롤링되어 SEO에 유리합니다. 페이지를 다시 로드하지않고 SPA동작처럼 "보이게" 만듭니다.

### 3. <a>

순수 HTML 요소로, 사용자를 새 페이지의 URL로 이동시키는 하이퍼링크를 생성합니다. 이때 페이지는 완전히 새로고침 됩니다. 만약 Next.js에서 <a>를 사용할 일이 있으면 <Link>태그로 바꾸는 것이 좋습니다. (하지만 <Link>가 언제나 <a>태그를 포함하는 것은 아닙니다)


### reference

[Next.js 페이지 이동 Link, Router](https://talkwithcode.tistory.com/98)

[Routing: Linking and Navigating | Next.js](https://beta.nextjs.org/docs/routing/linking-and-navigating)

[[Router] router에 담긴 정보들](https://velog.io/@e_juhee/router-pathname)

[Next.js 라우팅](https://velog.io/@jay/Next.js-%EB%9D%BC%EC%9A%B0%ED%8C%85#catch-all-routes)