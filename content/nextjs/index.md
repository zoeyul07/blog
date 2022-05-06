---
emoji: ''
title: 'Next.js'
date: '2021-05-12 22:00:00'
author: seoyul
tags: frontEnd
categories: FrontEnd
---

## Next.js
- Vercel 이라는 프론트엔드 팀에서 만들었으며 따로 설정해주지 않아도 SSR, SEO, Typescript등을 지원하는 React 프레임워크로 static asset과 severless function으로 만들어진 hybrid application에 알맞는 solution이다.

### 기능
#### 코드 스플리팅
- 일반적인 리액트 싱글 페이지는 초기 렌더링 때 모든 컴포넌트를 내려받지만 용량이나 규모가 커지면 로딩 속도가 지연될 수 있다. Next에서는 필요에 따라 파일을 불러올 수 있게 여러개의 파일을 분리하는 코드 스플리팅을 사용하였다.

- 폴더 구조를 보면 pages 폴더 안에 각 page(라우트) 들이 들어가며 컴포넌트 폴더에는 리액트 컴포넌트들이 들어가는데 브라우저가 실행되고 사용자가 접속하면 첫 페이지인 index 페이지만 불러오고 그 이후에 다른 페이지로 넘어갔ㅇ르 떄는 해당 페이지만 불러오게 된다.

#### 간단한 클라이언트 사이드 라우팅
- Router와 Link를 모두 사용할 수 있는데 Link는 href를 이용하여 해당 페이지로 이동하게 할 수 있고 as는 href의 URL을 조금 더 직관적으로 만들어준다.
- hre: pages 디렉토리 내부에 페이지 이름. 예 /blog/[slug]
- as: 브라우저에서 보여질 주소. 예 /blog/hello-world.

- Router는 링크와 동일하게 해당 페이지로 이동해주는 역할울 하지만 개발자에게 제어권을 더 넘겨줘 쉽게 redirect도 가능하다.

```jsx
import Link from 'next/link'

function Home() {
  return (
    <ul>
      <li>
        <Link href="/blog/[slug]" as="/blog/welcome">
          <a>Blog</a>
        </Link>
      </li>
    </ul>
  )
}

export default Home
```
- <Link prefetch href="..."> 형식으로 prefetch 값을 전달해주면 데이터를 먼저 불러온다음에 라우팅을 시작한다.
  
```jsx
import Router from 'next/router'

function Link() {
  return (
    <div>
      Click <span onClick={() => Router.push('/about')}>here</span>
    </div>
  )
}

export default Link
```

- Shallow Routing: 라우터 객체를 통해 상태 손실 없이 pathname query를 받을 수 있다.(같은 페이지 URL 내에서만 작동한다.)

```jsx
import { useEffect } from 'react'
import { useRouter } from 'next/router'


function Page() {
  const router = useRouter()

  useEffect(() => {
    router.push('/?counter=10', null, { shallow: true })
  }, [])

}

export default Page
```

- 브라우저 주소는 /?counter=10로 변하지만 라우트의 상태만 변할 뿐이고 페이지는 대체되지 않는다.

#### 커스텀 API 서버(as-라우트 마스킹)
- 커스텀 서버를 통해 라우트를 마스킹할 수 있다.
- Link 에서 as를 사용하여 살제 페이지가 없는 곳에 요청하게 되면 직접 커스텀 서버를 생성해서 as URL이 href를 바라볼 수 있게 처리할 수 있다.(그래야 새로고침이나 뒤로 갔을 떄도 렌더링이 가능하다.)

![](https://images.velog.io/images/zoeyul/post/7e8daec9-fe2e-4c1a-8a6a-4f260cc9c40b/Screen%20Shot%202021-05-12%20at%2012.05.41%20AM.png)

### 기본 구조
#### pages

1. _document.js : HTML Document
- SPA 에서 시적점이 되는 index.html로 custom document를 만들때만 작성이 팔요하며 생략된 경우 기본 값을 사용한다.

- React lifecyle과 data fetching은 불가능하다.
![](https://images.velog.io/images/zoeyul/post/b8d45fe1-c62a-4c92-b91b-e5cc34ac0b18/Screen%20Shot%202021-05-11%20at%2010.37.09%20PM.png)

```jsx
import Document, { Head, Main, NextScript } from 'next/document';

export default class RootDocument extends Document {
    render() {
        return (
            <html>
                <Head>
                    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.15.6/styles/railscasts.min.css" />
                </Head>
                <body>
                    <Main />
                    <NextScript />
                </body>
            </html>
        );
    }
}
```

2. _app.js: Application Container 공통 레이아웃
- React에서 App.js로 공통의 레이아웃을 작성하듯 Next.js에서는 Application의 공통 레이아웃을 _app.js에서 작성할 수 있으며 style sheet import등을 할 수 있고 생략된 경우 기본값을 사용한다.

```jsx
import { AppProps } from 'next/app'; 
import Layout from '../components/Layout';

function MyApp({ Component, pageProps }: AppProps) {
  return (
    <Layout>
      <Component {...pageProps} />
    </Layout>
  );
}

export default MyApp;
```

3. _error.js : Error Page
- 에러 처리를 공통으로 하고자할 때 사용할 수 있다.
```jsx
import React, { Component } from 'react';

import ErrorPage from '../components/ErrorPage';

export default class RootError extends Component {
    render() {
        const { statusCode } = this.props;
        return <ErrorPage statusCode={statusCode} />;
    }
}
```

4. index.js : RootPage /로 시작되는 경로

#### public
- 정적 파일을 업로드 한다.

#### next.config.js
- Next.js의 환경 설정파일로 라우팅, typescript, less 등의 webpack 플러그인을 설정한다.

### Routing
- 파일 구조에 따라 동적으로 라우팅을 구현할 수 있다.

```
/pages/index.js -> /
/pages/apple/banana.tsx -> /apple/banana
/pages/apple/[id].tsx -> /apple/동적 path
```

### Data Fetch
- 정적 생성은 한번 생성 후 계속 사용되며 SSR은 요청때마다 생성된다.

#### 정적 생성
- 사전 렌더링 혹은 정적 페이지에 사용한다.
1. getStaticProps() : 사용자의 요청이 아닌 빌드할 때 데이터를 가져오며 페이지를 사전 렌더링하기 떄문에 SEO를 적용할 수 있고 캐시될 수 있다.(pages 폴더에서만 사용 가능하다.)
```jsx
import { GetStaticProps } from 'next'; // 타입스트립트 사용시 getStatcProps의 타입임포트


const ContentList = ({ contentList }) => {
  return (
    <ul>
      {contentList.map((content) => (
        <li>{content.title}</li>
      ))}
    </ul>
  );
};

export const getStaticProps: GetStaticProps = async (context) => {
  const { params, preview, previewData } = context;

  const res = await fetch('url');
  const contentLists = await res.json();
  return {
    props: {
      contentList,
    }, 
    revalidate:1, // 1초에 한번 데이터 fetch
  };
};

export default ContentList;
```
2. getStaticPaths(): 데이터를 기반으로 사전 렌더링할 동적 경로를 지정하며 정의하지 않은 하위 경로는 접근할 수 없다.
```jsx
export async function getStaticPaths() {
  return {
    //빌드 타임 때 아래 정의한 동적인값 경로만 pre렌더링한다
    paths: [
      { params: { dynamic: 1 } },
      { params: { dynmic: 2 } }
      ......
      { params: { dynmic: 동적인값 } }
    ],
    fallback: true,
  }
}
```

- fallback은 default가 false 이며 경로 지정 페이지만 사전 렌더링하고 없는 경로 접근시 404 페이지를 렌더링한다.

- fallback 키가 true인경우, getStaticPaths 에 없는 경로는 설정된 FallbackPage 를 반환하고, getStaticProps 를 통해 해당 url 에 필요한 data 를 요청한다. 이후의 동일한 요청에 대해서는 방금 작업을 이용해서 미리 렌더링된 페이지가 제공된다.
  
  - fallback 이 true 인 경우는, 동적 라우팅을 통해 미리 렌더링을 할 페이지가 너무 많아서 전부다 getStaticPaths 를 통해 pre-rendering 하기에 버거울 때 사용하면 좋다.


```jsx
import { useRouter } from 'next/router'

const Compo ({ data }) {
  const router = useRouter()

  if (router.isFallback) {
    return <div>Loading...</div>
  }
  return (
 
  )
}

export async function getStaticPaths() {
  const response = await fetch(".../posts")
  const data = await response.json()

  const paths = data.map(({ id }) => ({ 
    params: { id: String(id) }, // 값은 string으로 넣어야 한다.
  }))
 paths = [
   {params : {id:'1'} },
   {params : {id:'2'} },
   {params : {id:'3'} },
   {params : {id:'4'} },
 ]
  return { paths, fallback: false }
}

export async function getStaticProps({ params }) {
  const res = await fetch(`...url/${params.id}`)
  const data = await res.json()
  return {
    props: { data },
    revalidate: 1,
  }
}

export default Post
```

- getStaticProps()와 getStaticPaths()는 같이 사용하도록 한다.

#### SSR
- getServerSideProps(): 매 요청시 데이터를 fetch 한다.

```
export async function getStaticProps(context) {
  return {
    props: {}, // will be passed to the page component as props
  }
}
```
#### client side
- 빈번하게 자료가 업데이트된다면 pre-render하지 않고 client side에서 data를 fetch 하도록 한다.


```


```toc

```
