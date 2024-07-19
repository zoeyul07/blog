---
emoji: ''
title: 'msw (api mocking)'
date: '2024-01-04 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

### 사용시 이점

1. 중간에 service worker를 통해서 네트워크 요청을 가로채고 모의 응답을 받아 반환하기 때문에 기존 data fetch 로직을 그대로 사용하여 mock 데이터를 반환 받을 수 있다.
2. 그렇기 때문에 api 가 혹시라도 늦게 나온다고 하더라도 사전에 api spec 만 미리 맞춰놓는다면 네트워크 요청 로직을 미리 작성해 놓고 그대로 사용하여 테스트할 수 있다.
3.  데이터를 하드코딩 하거나 mock server를 만든다고 하더라도 추후에 url 및 엔드포인트 변경이라던가 네트워크 요청 로직을 다시 작성할 필요가 있는데, msw는 그에 비해 on/off 하듯이 기존에 필요한 로직을 그대로 사용하여 원하는 데이터를 조작하여 사용할 수 있어 백엔드와 개발 시점이 겹친다고 하더라도 일정이 미뤄지지 않고 개발할 수 있다.
4. 원하는 테스트를 진행하기 위해 db 조작을 따로 요청할 필요 없이 mock 데이터를 생성해 테스트할 수 있다.
5. header, cookie 등 네트워크 요청시 사용하는 것들은 대부분 조작 가능해서 프로덕션에서만 가능하고 로컬에서는 테스트가 불가능하던 것 까지 테스트가 가능하다.
6. storybook, 브라우저 에서 구동되는 페이지, nodejs 까지도 지원될 정도로 범위가 넓다.

### 설정

1. 라이브러리 설치

```powershell
yarn add msw --dev
```

1. 기반 코드 자동 생성

```powershell
npx msw init public/ --save
```

1. 요청이 들어왔을 경우 임의의 응답을 해주는 핸들러 코드 작성
- src/mocks에 handler 작성

```powershell
import { http, HttpResponse } from "msw";

import { BATCH_API } from "src/constants/api";

const batchBaseUrl = process.env.NEXT_PUBLIC_BATCH_API;

const batchResponse = {
  code: "DONE",
  data: {
    
  },
};

const getBatchList = http.get(`${batchBaseUrl}${BATCH_API.schedules}`, () =>
  HttpResponse.json(batchResponse),
);

export const handlers = [getBatchList];
```

1. 서비스 워커 생성 (브라우저 환경 설정)

```powershell
//src/mocks/worker.ts

import { setupWorker } from "msw/browser";

import { handlers } from "src/mocks/handlers";

export const worker = setupWorker(...handlers);
```

1. 서버 생성 (node 환경 설정)

```powershell
//src/mocks.server.ts

import { setupServer } from "msw/node";

import { handlers } from "src/mocks/handlers";

export const server = setupServer(...handlers);
```

1.  msw 동작 방식 설정

```powershell
import React, { type PropsWithChildren, useState, useEffect } from "react";

import tw from "twin.macro";

const IS_MOCKING_MODE = process.env.NEXT_PUBLIC_API_MOCKING;

const MSWComponent = ({ children }: PropsWithChildren) => {
  const [mswReady, setMswReady] = useState(!IS_MOCKING_MODE);

  useEffect(() => {
    const init = async () => {
      const initMocks = await import("src/mocks/index").then(
        (res) => res.initMocks,
      );
      await initMocks();
      setMswReady(true);
    };

    if (!mswReady) {
      init();
    }
  }, [mswReady]);

  if (!mswReady) {
    return null;
  }
  return <Container>{children}</Container>;
};

export default MSWComponent;

```

```jsx
//src/mocks/index.ts
export const initMocks = async () => {
  if (process.env.NODE_ENV === "development") {
    if (typeof window === "undefined") {
      (async () => {
        const { server } = await import("src/mocks/server");
        return server.listen();
      })();
    } else {
      (async () => {
        const { worker } = await import("src/mocks/worker");
        return worker.start();
      })();
    }
  }
};
```

```jsx
//.env.local
NODE_ENV=development
NEXT_PUBLIC_API_MOCKING=enabled
```

1.  동작 방식 적용
- 최상단 root에서 작성한 도작 방식대로 msw가 적용되도록 컴포넌트로 감싸준다.

```jsx
//pages/_app.tsx

function App(){
	return(<MSWComponent>
			{...children}
	  </MSWComponent>
	)
}
```


### reference

[Mocking responses](https://mswjs.io/docs/basics/mocking-responses)

[(약간은 험난했던) Next.js 13에 MSW 도입기](https://jaypedia.tistory.com/382)