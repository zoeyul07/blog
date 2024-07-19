---
emoji: ''
title: 'react-query'
date: '2023-02-06 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

# React Query

- 기본적으로 데이터를 fetch 해온 후 캐싱하게 되며 캐싱한 데이터는 stale 하다고 판단한다.
    - stale time을 지정할 수 있다.
- 포커스가 들어왔을 경우(refetchOnWindowFocus), 새로 mount 되었을 때(refetchOnMount), 네트워크가 끊어졌다가 다시 연결된 경우(refetchOnReconnect),  데이터가 stale 하다고 판단될 수 있는 경우에 서버에서 데이터를 가져와 캐싱을 했어도 refetch하도록 지정할 수 있다.
- 위의 컨셉으로 인해 사용자는 항상 fresh한 데이터를 보게할 수 있다.

## 지원 기능 및 동작

- 렌더링 퍼포먼스 측면에서 쿼리가 업데이트 될때만 컴포넌트를 업데이트하며 여러 컴포넌트가 같은 쿼리를 사용할 시 한꺼번에 묶어 업데이트 해준다.
- 쿼리가 지정된 시간동안 사용되지 않을 경우 자동으로 garbage collection을 지원한다.
- 데이터 프리패칭이 가능하다.
- 프로미스에 cancel 함수를 적용하면 프로미스가 취소됨에 따라 쿼리도 함께 취소된다.(쿼리가 취소되면 해당 상태가 이전으로 돌아감)
- `react-query/devtools`를 통해 devtools를 제공한다
    - react native 는 [flipper](https://fbflipper.com/docs/getting-started/react-native/) plugin으로 사용 가능
    - [**https://github.com/bgaleotti/react-query-native-devtools**](https://github.com/bgaleotti/react-query-native-devtools)
- infinite scroll을 위한 useInfinityQuery도 제공한다.
- CUD 요청이 많다면 optimistic update도 이용 가능
- 여러개의 쿼리를 묶어서 처리할 수 있는 useQueries hook
- mutation 요청시 서버에 값이 변하므로 react query에 들고 있는 서버 데이터는 stale 하게 된다. 이때 invalidation 메소드를 사용하면 해당 key를 들고있는 쿼리들이 refetch 가 발생하게 된다.

## 기본 개념

### useQuery

```jsx
const { data, isLoading, error, isFetching } = useQuery(querykey, fetchFn, options)
```

- queryKey: 요청에 대한 응답 데이터를 캐시할 때 사용하는 unique key
    - refetch, cache, 여러 컴포넌트에서 쿼리 공유를 위해 사용됨
    - 배열이여야한다.
    - 쿼리키 배열내의 object에서 순서는 달라도 같은 키고 인식하지만 배열 내에서 순서가 달라진다면 다른 키로 인식한다.
    - 쿼리 function의 변화에 영향을 주는 변수라면 query key에 포함하도록하자.
- fetchFn: 요청을 수행하기 위한 promise를 Return 하는 함수
    - 데이터를 반환하거나 error를 던져야한다.
    - 쿼리 function 내에서 에러를 던져야 query가 에러라고 인식할 수 있다.
    - axios 나 graphql-request와는 다르게 fetch 이용시 자동으로 실패한 호출에 대해 error를 던져주지 않을 경우 직접 함수 내에서 추가해준다.
- options: useQuery에서 사용할 option 객체
- GET 요청 사용시 이용
- `useQueries`, `useInfiniteQuery` 도 지원
- inactive 한 쿼리는 5분 후에 garbage collected 된다.
    - cacheTime 으로 지정할 수 있다.
- 에러를 반환하기 전에 쿼리는 3번의 재시도를 한다.
    - retry, retryDelay로 지정할 수 있다.
- isLoading은 데이터를 처음 가져올 때, isFetching은 데이터를 다시 불러올 때 사용
- enabled 옵션으로 쿼리가 실행되지 않게 할 수 있다.

### useMutation

```jsx
const { mutate, isLoading, error, data } = useMutation(mutationFn, options)
```

- mutationFn: 요청을 수행하기 위한 promise를 return 하는 함수
- options: 사용할 option 객체
- POST, PUT, DELETE 이용시 사용

***IMPORTANT: The `mutate` function is an asynchronous function, which means you cannot use it directly in an event callback in React 16 and earlier. If you need to access the event in `onSubmit` you need to wrap `mutate` in another function. This is due to [React event pooling](https://reactjs.org/docs/legacy-event-pooling.html).***

```jsx
useMutation({
  mutationFn: addTodo,
  onMutate: (variables) => {
    // A mutation is about to happen!

    // Optionally return a context containing data to use when for example rolling back
    return { id: 1 }
  },
  onError: (error, variables, context) => {
    // An error happened!
    console.log(`rolling back optimistic update with id ${context.id}`)
  },
  onSuccess: (data, variables, context) => {
    // Boom baby!
  },
  onSettled: (data, error, variables, context) => {
    // Error or success... doesn't matter!
  },
})

//mutaion이 끝나기 전에 unmount되면 실행되지 않는다.
mutate(todo, {
  onSuccess: (data, variables, context) => {
  },
  onError: (error, variables, context) => {
  },
  onSettled: (data, error, variables, context) => {
  },
})
```

- onSuccess 에서 invalidateQueries, setQueryData를 통해서 reseponse 값으로 쿼리 데이터를 갱신할 수 있다.

```jsx
// When this mutation succeeds, invalidate any queries with the `todos` or `reminders` query key
const mutation = useMutation({
  mutationFn: addTodo,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['todos'] })
    queryClient.invalidateQueries({ queryKey: ['reminders'] })
  },
})
```

```jsx
//이미 mutaion을 통해 데이터를 가져왔기 때문에 해당 데이터를 통해 또 다시 네트워크 호출을 하지 않고 
//setQueryData를 통해서 데이터를 업데이트 해준다.
const queryClient = useQueryClient()

const mutation = useMutation({
  mutationFn: editTodo,
  onSuccess: data => {
    queryClient.setQueryData(['todo', { id: 5 }], data)
  }
})

mutation.mutate({
  id: 5,
  name: 'Do the laundry',
})

// The query below will be updated with the response from the
// successful mutation
const { status, data, error } = useQuery({
  queryKey: ['todo', { id: 5 }],
  queryFn: fetchTodoById,
})
```

Updates via **`setQueryData`**must be performed in an *immutable* way. **DO NOT** attempt to write directly to the cache by mutating data (that you retrieved via from the cache) in place. It might work at first but can lead to subtle bugs along the way

```jsx
queryClient.setQueryData(
  ['posts', { id }],
  (oldData) => {
    if (oldData) {
      // ❌ do not try this
      oldData.title = 'my new post title'
    }
    return oldData
  })

queryClient.setQueryData(
  ['posts', { id }],
  // ✅ this is the way
  (oldData) => oldData ? {
    ...oldData,
    title: 'my new post title'
  } : oldData
)
```

### pagination

***일반 페이지***

- 다음 페이지 요청시 이전에 있던 데이터가 유지되지 않을 수 있다. 그럴 경우 keepPreviousData 옵션을 사용하자.

***무한스크롤***

- **useInfiniteQuery 를 이용하자**
- 일반 페이지와 마찬가지로 keepPreviousData 옵션도 함께 사용한다.
- **fetchNextPage, fetchPreviousPage, getNextPageParam, getPreviousPageParam, isFetchingNextPage, isFetchingPreviousPage 이용 가능**
- 양방향 inifite list, 반대방향으로 페이지 보여주기, 수동업데이트 가능

***Note: It's very important you do not call `fetchNextPage` with arguments unless you want them to override the `pageParam` data returned from the `getNextPageParam` function. e.g. Do not do this: `<button onClick={fetchNextPage} />` as this would send the onClick event to the `fetchNextPage` function.***

### **Placeholder Query Data**

- 진짜 데이터를 받아오는 동안 데이터를 보여주고 싶을 때 사용할 수 있다.
- initialData로도 사용이 가능하지만 그와는 다르게 캐시로 지속되지 않는다.

### Invalidating queries

```jsx
// Invalidate every query in the cache
queryClient.invalidateQueries()
// Invalidate every query with a key that starts with `todos`
queryClient.invalidateQueries({ queryKey: ['todos'] })
```

- invalidateQueries를 통해 query 데이터를 stale 하게 설정이 가능하다. 해당 함수를 이용하면,
    - stale 로 mark 되고 stale time configuration 하게 된다면 override 된다.
    - 쿼리가 usequery hook을 통해 렌더되고 있다면 background에서 refetch 된다.

```jsx
import { useQuery, useQueryClient } from '@tanstack/react-query'

// Get QueryClient from the context
const queryClient = useQueryClient()

queryClient.invalidateQueries({ queryKey: ['todos'] })

// Both queries below will be invalidated
const todoListQuery = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodoList,
})
const todoListQuery = useQuery({
  queryKey: ['todos', { page: 1 }],
  queryFn: fetchTodoList,
})

queryClient.invalidateQueries({
  queryKey: ['todos', { type: 'done' }],
})

// The query below will be invalidated
const todoListQuery = useQuery({
  queryKey: ['todos', { type: 'done' }],
  queryFn: fetchTodoList,
})

// However, the following query below will NOT be invalidated
const todoListQuery = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodoList,
```

- variable이나 subkey가 없을 경우에만 return 하고 싶다면 exact 옵션을 사용하자

```jsx
queryClient.invalidateQueries({
  queryKey: ['todos'],
  exact: true,
})

// The query below will be invalidated
const todoListQuery = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodoList,
})

// However, the following query below will NOT be invalidated
const todoListQuery = useQuery({
  queryKey: ['todos', { type: 'done' }],
  queryFn: fetchTodoList,
})
```

- predicate 를 통해 invalidate 할지 true false 를 return 할 수 있다.

```jsx
queryClient.invalidateQueries({
  predicate: (query) =>
    query.queryKey[0] === 'todos' && query.queryKey[1]?.version >= 10,
})

// The query below will be invalidated
const todoListQuery = useQuery({
  queryKey: ['todos', { version: 20 }],
  queryFn: fetchTodoList,
})

// The query below will be invalidated
const todoListQuery = useQuery({
  queryKey: ['todos', { version: 10 }],
  queryFn: fetchTodoList,
})

// However, the following query below will NOT be invalidated
const todoListQuery = useQuery({
  queryKey: ['todos', { version: 5 }],
  queryFn: fetchTodoList,
}
```

### fetching 여부 한번에 확인하기

- useQuery를 병렬적으로 선언하여 사용하다 보면, 선언된 모든 쿼리들의 Fetching 여부를 확인하고 싶을 때가 있다.
    - 홈화면 mydata 및 여러 api를 다 fetch 했는지 로딩 status 확인 및 로그인기능 코드에 적용 가능한지 확인
    - [**Query Client를 활용한 커스텀 훅](https://tech.kakao.com/2022/06/13/react-query/) 참고**



### Reference

- React Query로 client 상태 관리

[**react-query로 클라이언트 상태 관리 하기**](https://blog.hyunmin.dev/23)

- flipper devtools

[React Native Flipper Debugger 설정](https://velog.io/@rjc1704/React-Native-Flipper-Debugger-%EC%84%A4%EC%A0%95)

- query client 옵션

[QueryClient | TanStack Query Docs](https://tanstack.com/query/latest/docs/react/reference/QueryClient#queryclientsetquerydata)

- react query

[Overview | TanStack Query Docs](https://tanstack.com/query/latest/docs/react/overview)

[React-Query 도입을 위한 고민 (feat. Recoil) - 오픈소스컨설팅 테크블로그 - 강동희](https://tech.osci.kr/2022/07/13/react-query/)

[Practical React Query](https://tkdodo.eu/blog/practical-react-query)

[React-Query(우아한 세미나 정리)](https://velog.io/@jinyoung234/React-Query%EC%9A%B0%EC%95%84%ED%95%9C-%EC%84%B8%EB%AF%B8%EB%82%98-%EC%A0%95%EB%A6%AC)

[https://www.youtube.com/watch?v=MArE6Hy371c&t=634s](https://www.youtube.com/watch?v=MArE6Hy371c&t=634s)

- reactQuery & typescript

[React Query and TypeScript](https://tkdodo.eu/blog/react-query-and-type-script)

- react query 구조

[[꼭꼭] React Query 계층 구조 도입하기](https://velog.io/@rat8397/%EA%BC%AD%EA%BC%AD-React-Query-%EA%B3%84%EC%B8%B5-%EA%B5%AC%EC%A1%B0-%EA%B5%AC%EB%B6%84%ED%95%98%EA%B8%B0)

- react cocurrentMode & reactQuery

[React Query와 함께 Concurrent UI Pattern을 도입하는 방법 | 카카오페이 기술 블로그](https://tech.kakaopay.com/post/react-query-2/)

- Redux to ReactQuery

[Store에서 비동기 통신 분리하기 (feat. React Query) | 우아한형제들 기술블로그](https://techblog.woowahan.com/6339/)

[My구독의 React Query 전환기](https://tech.kakao.com/2022/06/13/react-query/)

[카카오페이 프론트엔드 개발자들이 React Query를 선택한 이유 | 카카오페이 기술 블로그](https://tech.kakaopay.com/post/react-query-1/#user-content-fn-2)

- SWR vs ReactQuery

[React에서 서버 데이터를 최신으로 관리하기(React Query, SWR)](https://fe-developers.kakaoent.com/2022/220224-data-fetching-libs/)

[React Query vs SWR](https://tech.madup.com/react-query-vs-swr/)

[Redux 를 넘어 SWR 로(2)](https://min9nim.vercel.app/2020-10-05-swr-intro2/)