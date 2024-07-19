---
emoji: ''
title: 'redux'
date: '2022-08-11 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

## Redux

### action

- useReducer를 이용하면 컴포넌트에서 다양한 상황에 따라 다양한 상태를 다른 값으로 업데이트 해줄 수 있다.
- 액션은 객체로 type 필드를 작성해 다양한 상황을 만들어 준다.

```jsx
//액션 함수
const change = (changeData) => ({type : actions.CHANGE, info : changeData});
```

### store

- 리덕스에서는 한 어플리케이션당 하나의 스토어를 만들게 되며 그 안에는 state, reducer 및 내장함수들이 있다.

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';
import {createStore} from 'redux';
import { composeWithDevTools } from 'redux-devtools-extension';
import { Provider } from 'react-redux';
import rootReducers from './ex/modules/combineStore';
const store = createStore(rootReducers, composeWithDevTools());
ReactDOM.render(
  <React.StrictMode>
    <Provider store = {store}>
      <App />
    </Provider>
  </React.StrictMode>,
  document.getElementById('root')
);
serviceWorker.unregister();
```

### Dispatch

- 스토어의 내장함수 중 하나로 액션을 인자로 넘겨 Reducer() 함수를 실행하여 액션 ㄱ밧을 받아 상태를 변경한다.

```jsx
dispatch({type:"ADD"})
```

### Reducer

- 2개의 인자값을 받아와 state 값에 변화를 일으키는 함수로 현재의 state와 전달받은 action을 참고하여 새로운 state를 return 해준다.

```jsx
//state 는 기존 state 값, action은 dispatch 인자값
function exampleReducer (state, action) {
	switch(action.type) {
		case "ADD" :
			return { count: state.count + 1 } 
		case "MINUS":
			return { count: state.count -1 }
		default:
			return state
	}
}

//reducer 사용시 여러개의 reducer들을 하나로 함쳐서 사용한다.

import {combineReducers} from 'redux';
import counter from './counter';
import todoListStore from './todoListStore';
const rootReducer = combineReducers({
    counter,
    todoListStore
});
export default rootReducer;
```

### subscribe

- 함수 형태의 값을 파라미터로 받아와 전달하면 액션이 디스패지 될 때마다 전달해준 함수가 호출된다.
- dispatch 되어 state 가 변경될 때 리렌더링 해준다.

### Connect()

```jsx
//class를 redux에 연결하기 위해 connect()라는 함수를 이용한다.
connect(mapReduxStateToReactProps, mapDeduxDispatchtoReactProps)(전달받을 컴포넌트)

/*mapReduxStateToReactProps는 redux store가 변경될 때마다 호출이 되며, 
redux state값을 react props로 mapping 해준다*/

/*인자값으로 받은 state는 redux store의 state값을 공급해주고. 
return값으로 react props가 되어 하위컴포넌트에서 사용하게 될 props.state가 된다.*/
function mapReduxStateToReactProps(state){
return { id : state.id};
}

/*mapDeduxDispatchtoReactProps는 하위 컴포넌트에서 사용할 이벤트이다.
인자값으로 store.dispatch() api를 받아서 사용할 수 있게 해주며
return 값은 하위컴포넌트에서 사용하게 될 props.onClick이 된다.*/
function mapDispatchToProps(dispatch) {
	return onClick: function (size) {
		dispatch({ type: "INCREMENT", size: size });
	}
}
```

### useSelector

- connect()함수에서 mapReduxStateToReactProps와 유사한 역할을 하며 store의 state 를 할당할 수 있도록 한다.
- 연결된 action이 dispatch 될때마다 useSelector에 접근되어 값을 반환하게 되며, 리렌더링 된다

```jsx
const state = useSelector(state => state.todoListStore);
```

### useDispatch

- 리덕스 스토어에 설정된 action에 대한 dispatch를 연결하는 hook으로 store 파일 안에 선언된 action을 연결할 수 있도록 해준다.

```jsx
const selectChange = (changeData) => {dispatch(T.change(changeData))};
```

## Redux-Thunk

- 리덕스에서 비동기 작업을 처리할 때 가장 많이 사용하는 미들웨어로 액션 객체가 아닌 함수를 디스패치할 수 있다.
- dispatch()가 일어나면 액션 생성 함수로 가는데, 이때 받을 인자를 객체가 아닌 함수로 받을 수 있다.

```jsx
//액션함수
const increase = (ten) => ({ type: INCREASE, number : ten});

//thunk
/*store를 사용하여 dispatch, getState를 접근 할 수 있고, 
reducer로 가는 걸 next로 통제하거나, 
action으로 reducer를 진행하게 만든다.*/
//reducer로 가기 전에 끼어들어서 값이나 원래 dispatch애를 바꿀 수 있다.
const increase = (ten) => (dispatch, getState) => { 
	//로직 구현;
};
```

### Redux-Saga

- 액션을 모니터링하고 있다가 특정 액션이 발생하면 이에따라 특정 작업(자바스크립트 실행, 다른 액션 디스패치, 현재상태 불러오기 등등)을 실행한다.
- redux-saga 는 redux-thunk에 비해 다음과 같은 기능들을 사용 가능하다.
    - 비동기 작업시 기존 요청취소
    - 특정 액션 발생시 다른 액션 디스패치 및 자바스크립트 코드 실행
    - 웹소켓 사용시 channel 기능 사용
    - api 요청 실패시 재 요청
- es6 generator 문법을 사용한다.

```jsx
//일반적으로 함수에서는 값을 여러번 반환하는것은 불가능하다.
function weirdFunction() {
  return 1;
  return 2;
}

/*generator는 함수를 작성할 때 툭정 구간에 멈춰놓을 수 있고,
원할 때 따시 돌아가게할 수 있으며,
결과값을 여러번 반환할 수 있다.*/

/*generator 함수를 호출한다고해서 해당 함수 안의 코드가 바로 시작되지는 않는다. 
generator.next()를 호출해야만 코드가 실행되며, yield를 한 값을 반환하고 코드의 흐름을 멈춘다.
추후 다시 generator.next()를 호출하면 이어서 다시 시작된다.*/

function* generatorFunction() {
    console.log('안녕하세요?');
    yield 1;
    console.log('제너레이터 함수');
    yield 2;
    console.log('function*');
    yield 3;
    return 4;
}
```

```jsx
function* watchGenerator() {
    console.log('모니터링 시작!');
    while(true) {
        const action = yield;
        if (action.type === 'HELLO') {
            console.log('안녕하세요?');
        }
        if (action.type === 'BYE') {
            console.log('안녕히가세요.');
        }
    }
}

const watch = watchGenerator();

> watch.next()
> 모니터링 시작!
> { value: undefined, done: false }

> watch.next({ type: "HELLO"})
> 안녕하세요?
> { value: undefined, done: false }

```

```jsx
import { delay, put, takeEvery, takeLatest } from 'redux-saga/effects';const INCREASE = 'INCREASE';

const INCREASE = 'INCREASE';
const DECREASE = 'DECREASE';
const INCREASE_ASYNC = 'INCREASE_ASYNC';
const DECREASE_ASYNC = 'DECREASE_ASYNC';

export const increase = () => ({ type: INCREASE });
export const decrease = () => ({ type: DECREASE });
export const increaseAsync = () => ({ type: INCREASE_ASYNC });
export const decreaseAsync = () => ({ type: DECREASE_ASYNC });

function* increaseSaga() {
  yield delay(1000); // 1초를 기다립니다.
  yield put(increase()); // put은 특정 액션을 디스패치 해줍니다.
}

function* decreaseSaga() {
  yield delay(1000); // 1초를 기다립니다.
  yield put(decrease()); // put은 특정 액션을 디스패치 해줍니다.
}

export function* counterSaga() {
	//버튼을 여러번 클릭하면 클릭한만큼 increase 된다.
  yield takeEvery(INCREASE_ASYNC, increaseSaga); // 모든 INCREASE_ASYNC 액션을 처리
	//버튼을 여러번 클릭해도 한번만 decrease 된다.  
	yield takeLatest(DECREASE_ASYNC, decreaseSaga); // 가장 마지막으로 디스패치된 DECREASE_ASYNC 액션만을 처리
}
```

### reference

[[React] 17. Redux-Thunk](https://velog.io/@vvvvwvvvv/React-17.-Redux-Thunk-%EC%98%88%EC%A0%9C)

[10. redux-saga](https://react.vlpt.us/redux-middleware/10-redux-saga.html)

[최고 리덕스야 고맙다! Redux & Redux Toolkit 알아보기](https://velog.io/@sweet_pumpkin/%EB%AC%B4%EC%9E%91%EC%A0%95%EB%94%B0%EB%9D%BC%ED%95%98%EA%B8%B0-%EC%B5%9C%EA%B3%A0-%EB%A6%AC%EB%8D%95%EC%8A%A4%EC%95%BC-%EA%B3%A0%EB%A7%99%EB%8B%A4-Redux-Redux-Toolkit-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)