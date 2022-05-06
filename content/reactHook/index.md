---
emoji: ''
title: 'React:Hook'
date: '2021-04-12 22:00:00'
author: seoyul
tags: frontEnd
categories: frontEnd
---

## Hook
- 함수 컴포넌트에서 React state와 생명주기 기능을 연동할 수 있게 해주는 함수이다.
- class 안에서는 동작하지 않으며 class 없이 react를 사용할 수 있게 해준다.

### 사용 규칙
1. 최상위 에서만 Hook을 호출하며 반복문, 조건문, 중첩된 함수 내에서 hook을 실행하지 않는다.
- early return이 실행되기 전 react 함수의 최상위에서 hook을 호출하며 이 규칙을 따르면 컴포넌트가 렌더링 될 때마다 항상 동일한 순서로 hook이 호출되는 것을 보장하며 useState와 useEffect가 여러번 호출되는 중에도 hook의 상태를 올바르게 유지할 수 있도록 새준다.

2. 일반 javascript 함수에서는 hook을 호출하지 않는다. 
- React 함수 컴포넌트 내에서만 호출하거나 직접 작성한 custom hook 내에서만 hook을 호출하면 컴포넌트의 모든 상태 관련 로직을 소스코드에서 명확하게 보이도록 할 수 있다.

> eslint-plugin-react-hooks 플러그인을 통해 위 두가지 규칙을 강제할 수 있다.

### Hook을 사용했을 때 얻을 수 있는 이점
#### 컴포넌트 사이에서 상태와 관련된 로직을 재사용
React는 컴포넌트에 재사용 가능한 행동을 붙이기 위하여 render props, hoc와 같은 패턴을 사용한다. 하지만 이러한 패턴을 사용하기 위해서는 컴포넌트를 재구성해야하며 providers, comsumers, hoc, render props 그리고 다른 추상화에 대한 레이어로 둘러싸인 wrapper hell을 볼 가능성이 높다.

Hook을 사용하면 컴포넌트로부터 상태 관련 로직을 추상화할 수 있고 독립적인 테스트와 재사용이 가능하다. 계층 변화 없이 상태 관련 로직을 재사용할 수 있도록 도와주며 많은 컴포넌트 사이에서 hook을 공유하기 쉬워진다.

#### 로직에 기반을 둔 작은 함수로 컴포넌트를 나눌 수 있다.
각 생명주기 메서드는 자주 관련없는 로직이 섞여있다. componentDidMount 혹은 componentDidUpdate로 데이터를 가져오는것을 수행할 수도 있고 componentDidUpdate에 이벤트 리스너를 설정하거나 componentWillUnmount에서 cleanup을 수행하는 등 관계 없는 일부 로직이 포함될 수 있다. 함께 변경되는 상호 관련 코드는 분리되지만 이와 연관 없는 코드들은 단일 메서드로 결합하며 이로 인해 버그가 쉽게 발생하고 무결성을 해칠 수 있다.

이러한 것들을 해결하기 위해 hook을 통해 구독 설정 및 데이터를 불러오는 것과 같은 로직 같이 로직에 기반을 둔 작은 함수로 컴포넌트를 나눌 수 있으며 조금 더 예측할 수 있도록 하기 위해서 리듀서를 활용해 컴포넌트의 지역 상태값을 관리하도록 할 수 있다.

*관심사를 구분하려고 한다면 Multiple Effect를 사용한다.
Hook 이 만들어진 동기가 된 문제중의 하나가 생명주기 class 메서드가 관련이 없는 로직들은 모아놓고 관련이 있는 로직들은 여러개의 메서드에 나누어 놓는 경우가 자주 있기 때문이다. 

```
class FriendStatusWithCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0, isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }
  // ...
```

document.title을 설정하는 로직이 componentDidMount와 componentDidUpdtae에 나누어져 있고 subscription로직 또한 componentDidMount와 componentWillUnmount에 나누어져 있다.

이를 effect를 여러번 사용하여 서로 관련이 없는 로직들을 갈라놓을 수 있다.

```
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
  // ...
}
```
hook을 사용하면 생명주기 메서드에 따라서가 아니라 코드가 무엇을 하는지에 따라 나눌 수있다. 리액트는 컴포넌트에 사용된 모든 effect를 지정된 순서에 맞춰 적용한다.

#### React 를 배우는데 큰 진입 장벽인 Class를 이용하지 않을 수 있다.
Class가 코드의 재사용성과 코드 구성을 어렵게 만들며 javascript에서 this가 어떻게 작동하는지 알아야하고 이벤트 핸들러가 등록되는 방법 등을 기억해야한다.

Class는 잘 축소되지 않고 hot reloading을 깨지기 쉽고 신뢰할 수 없게 만들며 최적화를 더 드린 경로로 되돌리는 의도하지 않은 패턴을 장려할 수 있다.

개념적으로 React 컴포넌트는 항상 함수에 더 가까우며 hook이 그 개념 그대로 사용 가능하게 해준다.

```


```toc

```
