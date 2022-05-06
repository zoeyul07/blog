---
emoji: ''
title: 'Reack: Hooks (useState, useRef, useEffect)'
date: '2020-09-01 22:00:00'
author: seoyul
tags: frontEnd
categories: FrontEnd
---

## Hooks
- 함수형 컴포넌트에서도 상태를 관리하기 위해서 hooks를 사용한다.

### useState
- 컴포넌트에서 동적인 값을 상태(state)라고 하는데, 이것을 사용하면 컴포넌트에서 상태를 관리할 수 있다.

```tsx
 import React, { useState } from 'react';

 const [stageIdx, setStageIdx] = useState<number>(0);
 const [questionIdx, setQuestionIdx] = useState<number>(1);

 setQuestionIdx(questionIdx + 1);
 setStageIdx(questionIdx);
```

- useState를 사용할 때는 상태의 기본값을 파라미터로 넣어서 호출해준다. 이 함수를 호출해주면 배열이 반환되는데, 첫번째 원소는 현재 상태, 두번째 원소는 Setter함수이다.

- 배열 비구조화 할당을 통하여 다음과 같이 사용하지 않고 각 원소를 추출해서 사용한다.
```js
const numberState = useState(0);
const number = numberState[0];
const setNumber = numberState[1];
```

- Setter 함수는 파라미터로 전달 받은 값을 최신 상태로 설정해준다.

#### 함수형 업데이트
- setter 함수를 사용할 때 업데이트 하고 싶은 새로운 값을 파라미터로 넣어줄 수도 있지만 그 대신에 기존 값을 어떻게 업데이트 할지에 대한 함수를 등록하는 방식으로도 값을 업데이트할 수 있다.

```js
const [number, setNumber] = useState(0);

  const onIncrease = () => {
    setNumber(prevNumber => prevNumber + 1);
  }

  const onDecrease = () => {
    setNumber(prevNumber => prevNumber - 1);
  }
```

- 이렇게 그 다음 상태를 값을 업데이트 하는 함수로 파라미터에 넣어주며 함수형 업데이트는 주로 컴포넌트를 최적화 하게될 때 사용된다.

### useRef
- Javascript를 사용할 때 특정 DOM 을 선택해야 하는 상황에 getElementById. querySelector 같은 DOM Selector 함수를 사용해서 DOM을 선택하는데, 리액트를 사용하는 프로젝트에서도 가끔 DOM을 직접 선택해야 하는 상황이 발생하면 ref 라는 것을 사용한다.

- 함수형 컴포넌트에서 ref를 사용할 때는 useRef라는 Hook 함수를 사용한다. (클래스형 컴포넌트에서는 콜백 함수를 사용하거나, React.createRef 라는 함수를 사용한다.)

```tsx
const Result: React.FC<RouteComponentProps> = (props) => {
  const element = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (element.current) {
      const { current } = element;
      current.style.transition = `transform 1.2s ease-in`;
      current.style.transform = `translateX(1800px)`;
      setTimeout(() => props.history.push("/result/bali"), 1500);
    }
  }, [props.history]);

  return (
    <MainWrapper>
      <AnimationWrapper ref={element}>
        <LoadingImage                                     		src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEQ...."/>
      </AnimationWrapper>
    </MainWrapper>
  );
};
```
- useRef()를 사용하여 Ref 객체를 만들고, 이 객체를 선택하고 싶은 DOM에 ref 값으로 설정해준다. 그러면 Ref 객체의 .current 값은 우리가 원하는 DOM 을 가르키게 된다.

- useRef Hook은 DOM을 선택하는 용도 외에도 다른 용도가 한가지 더 있는데, 컴포넌트 안에서 조회 및 수정할 수 있는 변수를 관리하는 것이다.

- useRef로 관리하는 변수는 값이 바뀐다고 해서 컴포넌트가 리렌더링 되지 않는다. 리액트 컴포넌트에서의 상태는 상태를 바꾸는 함수를 호출하고 나서 그 다음 렌더링 이후로 업데이트 된 상태를 조회할 수 있는 반면, useRef로 관리하고 있는 변수는 설정 후 바로 조회할 수 있다.

- 이 변수를 이용하여 setTimeout, setInterval 을 통해서 만들어진 id,
외부 라이브러리를 사용하여 생성된 인스턴스, scroll 위치 등의 값을 관리할 수 있다.

```js
import React, { useRef } from 'react';
import UserList from './UserList';

function App() {
  const users = [
    {
      id: 1,
      username: 'test01',
      email: 'test01@gmail.com'
    },
    {
      id: 2,
      username: 'test02',
      email: 'test02@gmail.com'
    },
    {
      id: 3,
      username: 'test03',
      email: 'test03@gmail.com'
    }
  ];

  const nextId = useRef(4);
  const onCreate = () => {
    const user = {
      id: nextId.current,
      username,
      email
    };
    setUsers(users.concat(user));

    setInputs({
      username: '',
      email: ''
    });

    nextId.current += 1;
  };
  return <UserList users={users} />;
}

export default App;
```

- useRef() 를 사용 할 때 파라미터를 넣어주면, 이 값이 .current 값의 기본값이 되며, 이 값을 수정 할때에는 .current 값을 수정하고 조회 할 때에는 .current 를 조회한다.

### useEffect
- useEffect라는 hook은 컴포넌트가 마운트 되었을 때(처음 나타났을 때), 언마운트 되었을 때(사라질 때), 그리고 업데이트 될 때(특정 props가 바뀔 때) 특정 작업을 처리하여 사용할 수 있다.
```
useEffect(() => {
    console.log('컴포넌트가 화면에 나타남');
    return () => {
      console.log('컴포넌트가 화면에서 사라짐');
    };
  }, []);
```

- useEffect를 사용할 때에는 첫번쨰 파라미터에는 함수, 두번째 파라미터에는 의존값이 들어있는 배열(dependency)을 넣는다. 만약 배열을 비우게 되면, 컴포넌트가 처음 나타날때에만 useEffect에 등록한 함수가 호출괸다.

- 그리고 useEffect에서 함수를 반환할 수 있는데, 이를 cleanup 함수라고 부른다. cleanup 함수는 useEffect에 대한 뒷정리를 해주는데, 배열이 비어있는 경우에는 컴포넌트가 사라질 때 cleanup 함수가 호출된다.

- 주로 마운트시에는 props 로 받은 값을 컴포넌트의 로컬 상태로 설정, 외부 API 요청청, setInterval 을 통한 반복작업 혹은 setTimeout 을 통한 작업 예약과 같은 작업을 하고, 언마운트시에는 setInterval, setTimeout 을 사용하여 등록한 작업들 clear 하기 (clearInterval, clearTimeout), 라이브러리 인스턴스 제거를 한다.

- dependency에 특정 값을 넣으면 컴포넌트가 처음 마운트 될 때에도 호출이 되고, 지정한 값이 바뀔때에도 호출이 된다. 그리고 배열 안에 특정값이 있다면 언마운트시에도 호출이 되고 값이 바뀌기 직전에도 호출이 된다.

- useEffect안에서 사용하는 상태나 props가 있다면 배열 안에 넣어주어야 한다. 만약 넣지 않는다면 useEffect에 등록한 함수가 실행될 때 최신 props와 상태를 가르키지 않게 된다.

- 배열을 아예 생략하게 된다면 컴포넌트가 리렌더링 될때마다 호출이 된다.

- 리액트 컴포넌트는 기본적으로 부모 컴포넌트가 리렌더링 되면 바뀐 내용이 없어도 자식 컴포넌트도 리렌더링된다.(실제 DOM 에 변화가 반영되는 것은 바뀐 내용이 있는 컴포넌트에만 해당하지만, Virtual DOM 에는 모든걸 다 렌더링하고 있다.)




***
[벨로퍼트와 함께하는 모던 리액트](https://react.vlpt.us/basic/07-useState.html)
[리액트의 Hooks 완벽 정복하기](https://velog.io/@velopert/react-hooks#9-다른-hooks)
[About React Hook](https://medium.com/vingle-tech-blog/react-hook-ec3f25c2d8fa)
[When to useMemo and useCallback 영어 문서](https://kentcdodds.com/blog/usememo-and-usecallback)
[때늦은 React Hooks 시리즈 4탄 - useCallback/useRef](https://gist.github.com/ninanung/767ca722befa8b0affe51ffa0064296b)
[When to useMemo and useCallback 한글 문서](https://ideveloper2.dev/blog/2019-06-14--when-to-use-memo-and-use-callback/)
```


```toc

```
