---
emoji: ''
title: 'RTK Query '
date: '2023-02-03 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

## Default Cache Behavior

`When data is fetched from the server, RTK Query will store the data in the Redux store as a 'cache'. When an additional request is performed for the same data, RTK Query will provide the existing cached data rather than sending an additional request to the server.`

- 데이터가 서버로부터 fetch 되면 RTKQuery 데이터를 redux에 cache로 저장하게 된다.
- 같은 데이터를 위해 또 다른 요청을 하게 된다면 추가 요청을 하는 대신 RTKQuery는 존재하는 캐시 데이터를 제공하게 된다.

`When a subscription is started, the parameters used with the endpoint are serialized and stored internally as a queryCacheKey
 for the request. Any future request that produces the same queryCacheKey
 (i.e. called with the same parameters, factoring serialization) will be de-duped against the original, and will share the same data and updates. i.e. two separate components performing the same request will use the same cached data.`

- 구독이 시작되면 엔드포인트에 사용된 parameter는 serialized되며, 내부적으로 요청 캐시키로 저장된다.
- 앞으로 같은 캐시키를 생성한다면 원래의 것과 비교해 중복되지 않게 하며, 같은 데이터와 업데이트를 공유하게 된다. (두개의 다른 컴포넌트에서 같은 요청을 하게 된다면 같은 캐시된 데이터를 사용하게 된다.)

`When a request is attempted, if the data already exists in the cache, then that data is served and no new request is sent to the server. Otherwise, if the data does not exist in the cache, then a new request is sent, and the returned response is stored in the cache.`

- 요청이 시작되고 데이터가 이미 존재한다면 서버로 요청을 하지 않고 데이터를 주게 되고 데이터가 캐시에 존재하지 않는다면 새로운 요청을 하게 되면 해당 response는 캐시로 저장하게 된다.

`Subscriptions are reference-counted. Additional subscriptions that ask for the same endpoint+params increment the reference count. As long as there is an active 'subscription' to the data (e.g. if a component is mounted that calls a useQuery
 hook for the endpoint), then the data will remain in the cache. Once the subscription is removed (e.g. when last component subscribed to the data unmounts), after an amount of time (default 60 seconds), the data will be removed from the cache. The expiration time can be configured with the keepUnusedDataFor property for the [API definition as a whole](https://redux-toolkit.js.org/rtk-query/api/createApi#keepunuseddatafor), as well as on a [per-endpoint](https://redux-toolkit.js.org/rtk-query/api/createApi#keepunuseddatafor-1) basis.`

- 구독은 [reference-counted](https://ko.wikipedia.org/wiki/%EC%B0%B8%EC%A1%B0_%ED%9A%9F%EC%88%98_%EA%B3%84%EC%82%B0_%EB%B0%A9%EC%8B%9D)(참조횟수 계산 방식)이다. 같은 엔드포인트+param 에 대한 요청은 reference 횟수를 늘린다.
- 데이터에 대한 활성화된 구독이 있다면 (컴포넌트가 mount되면 엔드포인트에 대한 useQuery hook을 호출하게 된다.)데이터는 캐시로 남는다.
- 구독이 취소되면(데이터를 구독하는 마지막 컴포넌트가 unmount되면) 특정 시간( 기본 60초 ) 후에 데이터는 캐시에서 제거된다.
- 기한 시간은 keepUnusedDataFor property를 이용하여 설정할 수 있다.

### cache lifetime & subscription

```jsx
import { useGetUserQuery } from "./api.ts";

function ComponentOne() {
  // component subscribes to the data
  const { data } = useGetUserQuery(1);

  return <div>...</div>;
}

function ComponentTwo() {
  // component subscribes to the data
  const { data } = useGetUserQuery(2);

  return <div>...</div>;
}

function ComponentThree() {
  // component subscribes to the data
  const { data } = useGetUserQuery(3);

  return <div>...</div>;
}

function ComponentFour() {
  // component subscribes to the *same* data as ComponentThree,
  // as it has the same query parameters
  const { data } = useGetUserQuery(3);

  return <div>...</div>;
}
```

### **Manipulating Cache Behavior**

- **Reducing subscription time with `keepUnusedDataFor` (as a number in seconds)**

  ```jsx
  import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
  import type { Post } from './types'

  export const api = createApi({
    baseQuery: fetchBaseQuery({ baseUrl: '/' }),
    // global configuration for the api
    keepUnusedDataFor: 30,
    endpoints: (builder) => ({
      getPosts: builder.query<Post[], number>({
        query: () => `posts`,
        // configuration for an individual endpoint, overriding the api setting
        keepUnusedDataFor: 5,
      }),
    }),
  })
  ```

- **Re-fetching on demand with `refetch`/`initiate`**
- **Encouraging re-fetching with `refetchOnMountOrArgChange` (accepts either a boolean value, or a number as time in seconds)**

  - false를 넘기면 default behavior
  - true를 넘기면 엔드포인트 + arg combination으로 캐시데이터가 있더라도 mount되거나 argument가 바뀌게 되면 refetch 를 실행하게 된다.
  - 숫자를 넘기게 되면
    - 쿼리 구독이 시작되었을 때
      - 캐시 데이터가 있다면 현재 시간과 마지막 fulfilled 타임스탬프를 비교하고 주어진 시간보다 경과되었다면 refetch한다.
    - 쿼리가 없다면 데이터를 fetch 한다.
    - 쿼리가 있고 주어진 시간보다 경과되지 않았다면 기존에 있는 캐시 데이터를 전달하게 된다.

  ```jsx
  import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
  import type { Post } from './types'

  export const api = createApi({
    baseQuery: fetchBaseQuery({ baseUrl: '/' }),
    // global configuration for the api
    refetchOnMountOrArgChange: 30,
    endpoints: (builder) => ({
      getPosts: builder.query<Post[], number>({
        query: () => `posts`,
      }),
    }),
  })
  ```

  ```jsx
  import { useGetPostsQuery } from "./api";

  const Component = () => {
    const { data } = useGetPostsQuery(
      { count: 5 },
      // this overrules the api definition setting,
      // forcing the query to always fetch when this component is mounted
      { refetchOnMountOrArgChange: true }
    );

    return <div>...</div>;
  };
  ```

- **Re-fetching on window focus with `refetchOnFocus`**
- **Re-fetching on network reconnection with `refetchOnReconnect`**
- **Re-fetching after mutations by invalidating cache tags**

## Automated refetching

[**Tags**](https://redux-toolkit.js.org/rtk-query/api/createApi#tagtypes)

- cache control, Refetching 목적의 invalidation을 위한 naming
- mutation에 의해서 data가 영향을 받아야하는지 판단하기 위해 읽어지는 라벨
- tag는 tagTypes argument로 정의할 수 있다.

[**Providing tags**](https://redux-toolkit.js.org/rtk-query/api/createApi#providestags)

- _(optional, only for query endpoints)_
- 쿼리는 캐시 데이터의 provide tags 를 가질 수 있다.
- 쿼리에 의해 return 된 데이터에 tag를 붙인다.
- string으로 된 array나 array를 return 하는 callback가 argument가 된다.
- 함수는 결과의 첫번째 argument로 전달되며 두변째로는 response error 세번째로는 query에 전달된 arument가 된다.
- 쿼리가 성공했느냐에 따라 result 나 error는 undefined가 될 수 있다.

[**Invalidating tags**](https://redux-toolkit.js.org/rtk-query/api/createApi#invalidatestags)

- _(optional, only for mutation endpoints)_
- mutation은 tag를 기반으로 특정한 캐시 데이터를 invalidate 할 수 있다.
- 어떤 캐시 데이터가 refetch 혹은 제거 되어야할지 결정한다.
- string으로 된 array나 array를 return 하는 callback가 argument가 된다.
- 함수는 결과의 첫번째 argument로 전달되며 두변째로는 response error 세번째로는 query에 전달된 arument가 된다.
- 쿼리가 성공했느냐에 따라 result 나 error는 undefined가 될 수 있다.

### Cache Tags

- 한 엔드포인트를 위한 mutation이 다른 엔드포인트 쿼리가 제공한 데이터를 invalidate 할지 결정하는 역할을 한다.
- 캐시 데이터가 invalidate 될 때 쿼리를 refetch 하거나(컴포넌트가 아직 데이터를 이용하고 있다면), 캐시에서 데이터를 지우게 된다.
- createAPI에서 쿼리들이 제공할 수 있는 가능한 tag name option의 배열을 받을 수 있다.

```jsx
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query'
import type { Post, User } from './types'

const api = createApi({
  baseQuery: fetchBaseQuery({
    baseUrl: '/',
  }),
  tagTypes: ['Post', 'User'],
  endpoints: (build) => ({
    getPosts: build.query<Post[], void>({
      query: () => '/posts',
    }),
    getUsers: build.query<User[], void>({
      query: () => '/users',
    }),
    addPost: build.mutation<Post, Omit<Post, 'id'>>({
      query: (body) => ({
        url: 'post',
        method: 'POST',
        body,
      }),
    }),
    editPost: build.mutation<Post, Partial<Post> & Pick<Post, 'id'>>({
      query: (body) => ({
        url: `post/${body.id}`,
        method: 'POST',
        body,
      }),
    }),
  }),
})
```

- By declaring these tags as what can possibly be provided to the cache, it enables control for individual mutation endpoints to claim whether they affect specific portions of the cache or not, in conjunction with `providesTags` and `invalidatesTags` on individual endpoints.

### providing cache data

- Provided tags have no inherent relationship across separate `query`endpoints. Provided tags are used to determine whether cached data returned by an endpoint should be `invalidated`and either be refetched or removed from the cache. If two separate endpoints provide the same tags, they will still contribute their own distinct cached data, which could later both be invalidated by a single tag declared from a mutation.
- In order to provide stronger control over invalidating the appropriate data, you can use an arbitrary ID such a `'LIST'` for a given tag. See [Advanced Invalidation with abstract tag IDs](https://redux-toolkit.js.org/rtk-query/usage/automated-refetching#advanced-invalidation-with-abstract-tag-ids)
   for additional details.

```jsx
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
import type { Post, User } from './types'

export const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/' }),
  tagTypes: ['Posts'],
  endpoints: (build) => ({
    getPosts: build.query<Post[], void>({
      query: () => 'posts',
      providesTags: (result) =>
        result ? result.map(({ id }) => ({ type: 'Posts', id })) : ['Posts'],
    }),
    addPost: build.mutation<Post, Partial<Post>>({
      query: (body) => ({
        url: `post`,
        method: 'POST',
        body,
      }),
      invalidatesTags: ['Posts'],
    }),
    getPost: build.query<Post, number>({
      query: (id) => `post/${id}`,
      providesTags: (result, error, id) => [{ type: 'Posts', id }],
    }),
  }),
})

export const { useGetPostsQuery, useGetPostQuery, useAddPostMutation } = api
```

- 위처럼 사용하면 getPost에(모든 post detail) 해당하는 모든 쿼리를 다시 실행하게 된다.

```jsx
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
import type { Post, User } from './types'

export const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/' }),
  tagTypes: ['Posts'],
  endpoints: (build) => ({
    getPosts: build.query<Post[], void>({
      query: () => 'posts',
      providesTags: (result) =>
        result
          ? [
              ...result.map(({ id }) => ({ type: 'Posts' as const, id })),
              { type: 'Posts', id: 'LIST' },
            ]
          : [{ type: 'Posts', id: 'LIST' }],
    }),
    addPost: build.mutation<Post, Partial<Post>>({
      query(body) {
        return {
          url: `post`,
          method: 'POST',
          body,
        }
      },
      invalidatesTags: [{ type: 'Posts', id: 'LIST' }],
    }),
    getPost: build.query<Post, number>({
      query: (id) => `post/${id}`,
      providesTags: (result, error, id) => [{ type: 'Posts', id }],
    }),
  }),
})

export const { useGetPostsQuery, useAddPostMutation, useGetPostQuery } = api
```

- 위와 같이 사용하면 getPost는 invalidate 되지 않고 getPosts만 invalidate 된다.

### invalidate tag

```jsx
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query'
import type { Post, User } from './types'

const api = createApi({
  baseQuery: fetchBaseQuery({
    baseUrl: '/',
  }),
  tagTypes: ['Post', 'User'],
  endpoints: (build) => ({
    getPosts: build.query<Post[], void>({
      query: () => '/posts',
      providesTags: (result, error, arg) =>
        result
          ? [...result.map(({ id }) => ({ type: 'Post' as const, id })), 'Post']
          : ['Post'],
    }),
    getUsers: build.query<User[], void>({
      query: () => '/users',
      providesTags: ['User'],
    }),
    addPost: build.mutation<Post, Omit<Post, 'id'>>({
      query: (body) => ({
        url: 'post',
        method: 'POST',
        body,
      }),
      invalidatesTags: ['Post'],
    }),
    editPost: build.mutation<Post, Partial<Post> & Pick<Post, 'id'>>({
      query: (body) => ({
        url: `post/${body.id}`,
        method: 'POST',
        body,
      }),
      invalidatesTags: ['Post'],
    }),
  }),
```

- 위 예시는 addPost 혹은 editPost mutation이 호출되고끝났을 때 Post tag를 갖고 있는 어떠한 캐시 데이터도 valid 하지 않다. 컴포넌트가 해당 post tag를 갖고 있는 캐시 데이터를 구독하고 있다면 자동적으로 mutation 끝난 후에 서버로부터 업데이트 받기 위해 refetch 하게 된다.

### [Manual cache update](https://redux-toolkit.js.org/rtk-query/usage/manual-cache-updates)

- manual cache update 보다는 automated refetching을 선호함
- 이미 존재하는 쿼리의 캐시 데이터를 수동으로 업데이트 하기위해서 updateQueryData를 이용할 수 있다.
- dispatch에 접근할 수 어떠한 곳에서도 사용해 업데이트할 수 있다.
- optimistic update

  - 쿼리가 시작할 때 바로 업데이트
  - onQueryStarted 에서 updateQueryData이용
  - queryFulfilled가 reject 되면 .undo로 rollback 가능

  ```jsx
  updatePost: build.mutation<void, Pick<Post, 'id'> & Partial<Post>>({
        query: ({ id, ...patch }) => ({
          url: `post/${id}`,
          method: 'PATCH',
          body: patch,
        }),
        async onQueryStarted({ id, ...patch }, { dispatch, queryFulfilled }) {
          const patchResult = dispatch(
            api.util.updateQueryData('getPost', id, (draft) => {
              Object.assign(draft, patch)
            })
          )
          try {
            await queryFulfilled
          } catch {
            patchResult.undo()

            /**
             * Alternatively, on failure you can invalidate the corresponding cache tags
             * to trigger a re-fetch:
             * dispatch(api.util.invalidateTags(['Post']))
             */
          }
        },
      }),
  ```

- pessimistic update
  - 쿼리가 서버로부터 결과를 받아오고 나서 업데이트
  - onQueryStarted 에서 updateQueryData이용, 단 fulfilled 될때까지 wait
  ```jsx
  updatePost: build.mutation<Post, Pick<Post, 'id'> & Partial<Post>>({
        query: ({ id, ...patch }) => ({
          url: `post/${id}`,
          method: 'PATCH',
          body: patch,
        }),
        async onQueryStarted({ id, ...patch }, { dispatch, queryFulfilled }) {
          try {
            const { data: updatedPost } = await queryFulfilled
            const patchResult = dispatch(
              api.util.updateQueryData('getPost', id, (draft) => {
                Object.assign(draft, updatedPost)
              })
            )
          } catch {}
        },
      }),
  ```
- general update

  - useDispatch 및 store.dispatch에 접근이 가능하다면 어디에서도 업데이트가 가능하다.
  - You should generally avoid manually updating the cache outside of the `onQueryStarted`callback for a mutation without a good reason, as RTK Query is intended to be used by considering your cached data as a reflection of the server-side state.

  ```jsx
  import { api } from "./api";
  import { useAppDispatch } from "./store/hooks";

  function App() {
    const dispatch = useAppDispatch();

    function handleClick() {
      /**
       * This will update the cache data for the query corresponding to the `getPosts` endpoint,
       * when that endpoint is used with no argument (undefined).
       */
      const patchCollection = dispatch(
        api.util.updateQueryData("getPosts", undefined, (draftPosts) => {
          draftPosts.push({ id: 1, name: "Teddy" });
        })
      );
    }

    return <button onClick={handleClick}>Add post to cache</button>;
  }
  ```


## RTKQuery에서 Endpoint 관리

Typically, you should only have one API slice per base URL that your application needs to communicate with. For example, if your site fetches data from both `/api/posts` and `/api/users`, you would have a single API slice with `/api/` as the base URL, and separate endpoint definitions for `posts` and `users`. This allows you to effectively take advantage of [automated re-fetching](https://redux-toolkit.js.org/rtk-query/usage/automated-refetching) by defining [tag](https://redux-toolkit.js.org/rtk-query/usage/automated-refetching#tags) relationships across endpoints.

For maintainability purposes, you may wish to split up endpoint definitions across multiple files, while still maintaining a single API slice which includes all of these endpoints. See [code splitting](https://redux-toolkit.js.org/rtk-query/usage/code-splitting) for how you can use the `injectEndpoints` property to inject API endpoints from other files into a single API slice definition.

- 같은 baseURL을 사용한다면 한 파일 내에서 하나의 createApi로 관리하는게 좋음
- 유지보수를 위하여 여러개의 파일로 엔드포인트 정의를 나누고 싶다면 injectEndpoints를 이용해보자.

```jsx
// Or from '@reduxjs/toolkit/query' if not using the auto-generated hooks
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'

// initialize an empty api service that we'll inject endpoints into later as needed
export const emptySplitApi = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/' }),
  endpoints: () => ({}),
})
```

```jsx
import { emptySplitApi } from './emptySplitApi'

const extendedApi = emptySplitApi.injectEndpoints({
  endpoints: (build) => ({
    example: build.query({
      query: () => 'test',
    }),
  }),
  overrideExisting: false,
})

export const { useExampleQuery } = extendedApi
```

### `reducerPath`[](https://redux-toolkit.js.org/rtk-query/api/createApi#reducerpath)

The `reducerPath` is a *unique* key that your service will be mounted to in your store. If you call `createApi` more than once in your application, you will need to provide a unique value each time. Defaults to `'api'`.

```jsx
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query'

const apiOne = createApi({
  reducerPath: 'apiOne',
  baseQuery: fetchBaseQuery({ baseUrl: '/' }),
  endpoints: (builder) => ({
    // ...endpoints
  }),
})

const apiTwo = createApi({
  reducerPath: 'apiTwo',
  baseQuery: fetchBaseQuery({ baseUrl: '/' }),
  endpoints: (builder) => ({
    // ...endpoints
  }),
})
```

### reference

[Cache Behavior | Redux Toolkit](https://redux-toolkit.js.org/rtk-query/usage/cache-behavior#default-cache-behavior)
[Code Splitting | Redux Toolkit](https://redux-toolkit.js.org/rtk-query/usage/code-splitting)
[createApi | Redux Toolkit](https://redux-toolkit.js.org/rtk-query/api/createApi#reducerpath)
[API Slices: Code Splitting and Generation | Redux Toolkit](https://redux-toolkit.js.org/rtk-query/api/created-api/code-splitting)