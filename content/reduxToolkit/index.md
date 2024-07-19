---
emoji: ''
title: 'redux-toolkit'
date: '2022-09-23 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

# redux-toolkit

프로그램 개발을 하다보면, 전역에서 자주 사용되는 api를 호출하거나, api 호출한 결과를 여러 군데에서 사용해야 할 상황이 생기는데, 이와 같은 비동기 처리를 redux store에서는 자체적으로 하지 못하는 단점이 있다.

따라서 Redux를 사용할 때는 redux-thunk, redux-saga와 같은 미들웨어를 사용해서 비동기 처리를 진행한다.

Redux Toolkit을 이용한다면 createAsyncThunk를 사용해 비동기 처리를 진행할 수 있다.

### [**createAsyncThunk()**](https://narup.tistory.com/257#--%--createAsyncThunk--)

- 액션을 정의

```jsx
createAsyncThunk(typePrefix: string, payloadCreator: AsyncThunkPayloadCreator,
options?: AsyncThunkOptions): AsyncThunk

export const fetchUser = createAsyncThunk(
  'auth/profile',
  // 만약, api를 호출할때 object를 추가하고 싶으면 async (object, thunkAPI)로 호출합니다.
  async (_, thunkAPI) => {
    try {
      const response = await axios.get<{
        name: string
        email: string
        type: string
      }>('api/users/me')
      return response.data
    } catch (error) {
      return thunkAPI.rejectWithValue(error)
    }
  }
)

a. 첫번째 인자 : 액션타입
컴포넌트에서 dispatch로 해당 createAsyncThunk가 호출됨으로 만들어지는 객체를 가지고 그 내부에 있는 pending 메서드를 호출해 해당 액션을 디스패치한다.
그리고나서, 두번째 인자로 전달된 비동기 함수를 호출하여 Promise가 settled가 될 때까지 기다린 후, Promise 객체의 Result 슬롯의 결과에 따라서 fulfilled 메서드나 rejected 메서드를 호출해 액션을 디스패치한다.
그러면 뒤에서 설명할 extraReducre에 등록된 함수를 통해 저장되어 있던 케이스들을 확인한 뒤에 해당 케이스별로 콜백함수를 실행하여 작업을 처리한다.

b. 두번째 인자 : 페이로드 생성자
두번째 인자는 Promise 객체를 리턴하는 함수가 들어간다
이때, 해당 함수의 인자로 들어가는 값은 다음과 같다.
arg : 컴포넌트상에서 처음으로 dispatch로 비동기 작업 액션을 생성할 때에 보내지는 인자이다. 오로지 하나의 값만 전달할 수 있기 때문에, 값이 여러개라면 객체로 보내라고 하고 있다.
thunkAPI : 기존 redux-thunk에서 전달했던 값들에서 조금 더 추가가 된 것들이 있다.
--dispatch : 함수 몸체에서 무언가 작업을 한 후, 특정 액션을 또다시 dispatch할 필요성이 있을 경우 사용하면 된다.
--getState : 함수 몸체에서 특정 작업을 할 때에, 현재 store의 상태를 조회해서 진행해야 할 필요성이 있을 경우 사용하면 된다.
--extra : 이건 사실 사용할 일이 잘 없긴한데, 초기에 redux store을 만들면서 middleware로 해당 값을 붙여놨으면 자동으로 비동기 작업을 할 때마다 해당 데이터가 이 extra 프로퍼티의 값으로 할당되어 들어간다.
```

- 위와 같이 첫번째 파라미터로 typePrefix로 액션 타입을 생성하고, 두번째 파라미터로 promise 기반으로 결과를 반환하는 action을 작성한다.
- createAsyncThunk는 Promise의 3가지 상태와 같이 **pending, fulfilled, rejected의 상태**를 갖는다.

### [**extraReducer**](https://narup.tistory.com/257#--%--extraReducer)

- 리듀서에 연결

```jsx
export const fetchSlice = createSlice({
  name: 'posts',
  initialState,
  reducers: {},
  extraReducers: (builder) => {
    builder.addCase(fetchUser.pending, (state) => {
      state.loading = 'pending';
    });
    builder.addCase(fetchUser.fulfilled, (state, action) => {
      state.loading = 'succeeded';
      state.user = action.payload;
    });
    builder.addCase(fetchUser.rejected, (state) => {
      state.loading = 'failed';
    });
  },
});
```

### 사용

```jsx
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit'

const fetchUserById = createAsyncThunk(
  'users/fetchByIdStatus',
  async (userId: number, thunkAPI) => {
    const response = await userAPI.fetchById(userId)
    return response.data
  }
)
.
.
.
const usersSlice = createSlice({
  name: 'users',
  initialState,
  reducers: {
    // standard reducer logic, with auto-generated action types per reducer
  },
  extraReducers: (builder) => {
    builder.addCase(fetchUserById.fulfilled, (state, action) => {
      // Add user to the state array
      state.entities.push(action.payload)
    })
  },
})

// inside component
dispatch(fetchUserById(123))
```

### redux toolkit vs RTKQuery

```jsx
redux toolkit 에는 createAsyncThunk 를 사용해서,
thunk 를 만들어서 async 요청을 관리할 수 있도록 하는 기능이 있다.
그런데 createAsyncThunk 의 경우 async 요청의 응답을 따로 createSlice 를 활용해서 관리해줘야 해서, 코드가 늘어날 수 밖에 없다.

RTK Query는 Redux Toolkit 의 createSlice 와 createAsyncThunk 의 기능을 동시에 사용하면서,
react-query 와 유사하게 백엔드에서 넘어온 데이터를 관리할 수 있는 기능이 만들어져있다.
또한 캐싱 관련 기능도 지원하기 때문에, 불필요한 요청을 줄일 수도 있다.
그리고 post 하자마자 자동으로 get 하는 기능까지도 구현할 수 있다.
뿐만 아니라, 프론트 데이터와 api 데이터 관리하는 로직 자체를 분리시킬 수도 있습니다.

백엔드와의 요청을 통해서 받아오는 데이터 → RTK Query 로 관리

프론트엔드만의 데이터 → createSlice 로 관리
```

### reference
[RTK QUERY](https://velog.io/@dlstjr1106/RTK-QUERY)

[RTK Query) 시작하기](https://velog.io/@jungsangu/RTK-Query-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)

[RTK Query Overview | Redux Toolkit](https://redux-toolkit.js.org/rtk-query/overview)

- jwt authentication

[RTK Query Examples | Redux Toolkit](https://redux-toolkit.js.org/rtk-query/usage/examples)

[React + Redux Toolkit: JWT Authentication and Authorization 2022](https://codevoweb.com/react-redux-toolkit-jwt-authentication-and-authorization/)

- migrating to rtk query

[Migrating to RTK Query | Redux Toolkit](https://redux-toolkit.js.org/rtk-query/usage/migrating-to-rtk-query)

[[React] Redux Toolkit 과 RTK Query 정리](https://velog.io/@nowod_it/React-Redux-Toolkit-%EC%A0%95%EB%A6%AC)

[[Redux] 비동기 상태처리를 위한 redux-toolkit 분석 및 탐구](https://velog.io/@chltjdrhd777/Redux-%EB%B9%84%EB%8F%99%EA%B8%B0-%EC%83%81%ED%83%9C%EC%B2%98%EB%A6%AC%EB%A5%BC-%EC%9C%84%ED%95%9C-redux-toolkit-%EB%B6%84%EC%84%9D-%EB%B0%8F-%ED%83%90%EA%B5%AC)

[[React] Redux Toolkit 에서의 비동기 처리(createAsyncThunk, extraReducers)](https://narup.tistory.com/257)