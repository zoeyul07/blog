---
emoji: ''
title: 'About Javascript: 동작 원리'
date: '2022-01-07 23:00:00'
author: seoyul
tags: frontEnd javascript
categories: FrontEnd Javascript
---

## javascript

자바스크립트는 스크립트를 작성해 실행하는데 컴파일이 필요하지 않은 Interpreter 언어이다. 하지만 자바스크립트 엔진 내부에서 실행중 컴파일이 필요한 경우 내부에서 컴파일한다.

### compile 언어

compile 언어와 interpreter 언어의 가장 큰 차이점은 pre-processting, 즉 compile 유무이다. compile이란 A를 B 언어로 변환하는 과정으로 고금 언어로 작성된 소스코드를 기계어로 변환하는 것을 의미한다. 변환된 기계어는 고급 언어를 바로 인터프리팅 방식으로 실행한 것보다 빠르다.

### interpreter 언어

interpret은 코드를 한줄씩 순차적으로 bytecode(abstraction of machine code)로 변환하는 방식이다. 이 bytecode를 가지고 컴퓨터가 이해할 수 없기 때문에 그것을 받아서 실행해주는 프로세스 가상 머신이 필요한데 이 가상 머신이 런타임에 호출되는 부분을 골라서 실행한다.

컴퓨터가 바로 이해할 수 있는 언어가 아닌 중간 언어를 만들고 그것을 실행하는 머신에 전달해줘야 하기 때문에 compile에 비해 느린 속도를 보이지만 런타임에 호출되는 부분만 실행하기 때문에 동작 중에도 수정 및 디버깅이 가능하며 프로세스 가상 머신만 있다면 ㄴ어디서든 같은 코드를 실행할 수 있기 때문에 특정 환경에 종속적이지 않다.

### V8 엔진

자바스크립트 엔진은 자바스크립트 코드를 실행하는 프로그램 혹은 인터프리터를 말하며 그 중 구글의 V8엔진은 크롬과 nodejs에서 사용되며 크게 두부분으로 나뉜다.

1. Memory Heap: 메모리 할당이 이루어지는 곳
2. Call Stack : 코드가 실행되면서 스택 프레임이 쌓이는 곳

javascript 의 성능이 향상될 수 있었던 이유는 엔진 내부에서 컴파일 과정을 거치기 때문이다. 먼저 엔진이 실행할 JS 파일을 받게 되고, parser가 js 파일을 위에서부터 순차적으로 파싱하고, AST(Abstract Syntax Tree)를 구축하는 과정을 거친다. [참고](https://blog.sessionstack.com/how-javascript-works-parsing-abstract-syntax-trees-asts-5-tips-on-how-to-minimize-parse-time-abfcf7e8a0c8)

다음으로 interpreter가 AST를 받아 bytecode로 변환하고 실시간으로 bytecode가 실행된다. Byte code는 실행되며 최적화를 위해 profiling data와 함께 compiler에서 보내진다. compiler가 profiler에게 받은 부분을 최적화하고 최적화된 부분이 실행될 차례가 되면 bytecode 대신 최적화된 코드를 실행한다. 만약 최적화에 실패하면 원래 bytecode를 돌려준다.(코드를 수행하는 과정에서 프로파일러가 지켜보며 최적화할 수 있는 코드를 컴파일러에게 전달해주며 주로 반복하며 실행하는 코드를 컴파일(최적화) 하고 원래 있는 코드와 최적화된 코드를 바꿔준다.) 코드를 우선 인터프리터 방식으로 실행하고 필요할 때 컴파일 하는 방법을 Just-In-Time 컴파일러 라고 부른다.

#### 최적화된 코드는 어떻게 만들어낼까?

1. inline caching : 효율성을 위해 자주 사용하는 데이터를 빠른 속도로 접근하기 쉬운 곳에 저장해두고 사용한다.

2. hidden class: 같은 이름의 property들을 사용하는 여러개의 object가 있다면 JSE에는 property의 정보를 보고 hidden classf로 만들어서 저장해두고 데이터는 별도로 attributes만 저장해둔다. 그리고 a.name에 접근이 필요한 경우 property information에서 id property의 위치를 확인 후 aObj의 해당 위치의 값을 가져온다.

   ```javascript
   const a = {id: 1, name:"hello"}
   const b = {id: 2, name:"world"}
   const c = {id: 3, name: "foo"}
   const d = {id: 4, name: "bar"}
   ...

   property information 1 = {
   	첫번째 값(id) 위치
   }

   property information 2 = {
   	두번째 값(name) 위치
   }
   ...

   const aObj = {1, "hello"}
   const bObj = {2, "word"}
   ...
   ```

   여기서 property의 순서를 바꾸게 되면 수행 시간이 더 오래걸리게 되는데, 순서를 바꾸기 전에는 hidden calss를 두개만 만들어두고 계속 같은 위치의 값을 불러올 수 있었는데 순서가 바뀐 object가 존재하게 되면 hidden class를 더 많이 만들어야 해서 서로 다른 object를 찾을 때 다른 hidden class를 이용해야해서 위치를 찾아야하기 때문에 속도가 느려지게 된다.

### Runtime

브라우저에서 사용되는 setTimeout 과 같은 API는 엔진이 제공하는 것이 아니고 브라우저가 제공하는 web API를 이용해 사용하게 되는데 DOM, AJAX 등도 이를 통해 이용하게 된다.

![](https://images.velog.io/images/zoeyul/post/eadc7fa2-5aad-4c8d-991c-89e9c89f6a40/Screen%20Shot%202022-01-07%20at%202.30.01%20AM.png)

JSE는 js 파일을 위에서부터 순차적으로 읽어들이고 처리하고 setTimeout과 같은 WebAPI, 오래 걸리는 일을 만나게 되면 비동기 식으로 맡겨두고 처리가 끝난 후 callback queue에 추가된다.

call stack 에 쌓여있던 일들이 순차적으로 모두 실행되고 비게 되면(page load가 완전히 끝난다는 조건도 함께 충족해야한다.) 그때 event loop은 callback queue를 보고 있다가 call stack 에 추가해 일을 callback queue에 쌓여 있는 일을 순차적으로 수행하게 된다.

### Call Stack

자바스크립트는 싱글 쓰레드 프로그래밍 언어이기 때문에 콜스택이 하나이며 한번에 하나의 일만 할 수 있다.

콜스택은 우리가 프로그램의 어디에 있는지를 기록하는 자료구조이다. 우리가 함수 안으로 들어가는 순간 해당 함수를 이 스택의 제일 위에 놓게 되며 함수에서 돌아오면 스택의 가장 윗부분이 제거된다.

콜스택의 각각은 스택 프레임이라고 부르며 예외가 발생했을 때 스택 트레이스가 만들어지는 방식이다. 스택 트레이스란 예외가 발생했을 때의 콜스택 상태이다.

재귀 함수와 같이 별 다른 종료 조건 없이 자기 자신을 반복해서 호출하면 실행의 모든 단계에서 동일한 함수가 콜 스택에서 반복해서 추가된다. 그러다 어느순간 콜 스택의 수가 실재 콜 스택의 크기를 넘게 되면 maxium call stack size exceeded 와 같은 에러를 브라우저가 던지게 된다.

자바스크립트는 단일 쓰레드이기 때문에 콜스택이 하나이고 콜 스택에 수행할 함수가 있으면 아무것도 할 수 없는 블록킹 상태로 들어간다. 그렇게 때문에 위와 같은 이유로 오랜시간동안 코드 실행이 늦어지면 렌더링 혹은 다른 코드를 실행할 수 없기때문에 사용자 경험에 좋지 않은 영향을 끼칠 수 있다.

UI에 크게 영향을 끼치지 않고 무거운 코드를 수행하려면 비동기 콜백을 사용할 수 있다.

### Hoisting

#### lexical enviroment

lexical enviroment란 변수나 함수가 코드가 어디에서 실행되었는지(dynamic scope)가 아닌 어디 작성되어 있는지를 나타내는 것이다. 코드를 실행하여 call stack을 확인하면 맨 아래에 anonymous가 있는데 이것은 global execution context이고 그 위에 하나씩 함수가 실행될 때마다 쌓이게 되는 것을 function execution context 이다.

즉, global하게 작성된 함수나 변수의 lexical environment는 global execution content이고 function 내부에서 실행되는 함수의 lexical envirionment는 함수가 실행되며 생기는 function execution context이다.

#### Hosting

hoisting은 변수나 함수가 자신이 속한 lexical environment 맨 위에서 선언되는 것을 의미한다.

```javascript
hello = "hello1"
var hello;

//실행되며 변수를 선언하는 let hello; 를 만나게 되면 hoisting을 하고 hello의 lexical environment는 global이기 때문에 맨 위에 선언된다.
=> var hello = undefined;
	 hello = "hello1"

```

```javascript
hello = "hello1"
var hello;

const print = () => {
	console.log(hello)

	var hello2 = "hello2"
	console.log(hello)
}

//첫번째 로그인 hello는 function execution context 내부에 없는 변수이기 때문에 lexical environemnt가 다르다. 자신이 찾는 변수나 함수가 같은 environment에 없는 경우 global로 가서 찾기 때문에 오류 없이 결과가 출력된다.
=>  var hello = undefined;
	 hello = "hello1"

	const print = () => {
    var hello2 = undefined;

    console.log(hello)
    hello2 = "hello2"
    console.log(hello2)
  }

//hello1
//hello2
```

### var, let, const

#### var

var는 hoisting으로 인해 다른 블록 안에서 재선언을 하여도 오류가 발생하지 않는다 그로 인해 const, let이 추가되었다.

#### let, const

var의 경우 hoisting이 되면 자신의 lexical environment 맨 위에 undefined로 선언하지만 const 와 let의 경우 선언만 되고 undefined를 할당하지 않으므로 initialize 전에 사용할 수 없다는 에러를 마주치게 된다. 또한 let은 같은 lexical envirionment내에서는 한번만 선언이 가능하다.

즉, var의 경우 선언과 함께 할당(값이 없다면 undefined로)이 되고 같은 lexical environment내에서 재선언 및 재할당이 가능하지만 let의 경우 선언후 언제 어디서든지 재할당할 수 있지만 같은 lexical environment내에서는 한번만 선언이 가능하며, const 의 경우 재할당이 불가능하기 때문에 반드시 선언과 함께 할당해주어야한다.(let과 동일하게 같은 lexical environment내에서는 한번만 선언 가능)

### Scope Chain

scope chain 이란 자신의 조상 lexical environment에 접근 가능하다는 의미이다. function execution context 안에 찾는 변수가 없을 경우 global execution context에서 찾을 수 있던 이유가 이것 떄문이다. 대신 부모의 부모 environment에 있는 것들은 사용할 수 있지만 부모 및 조상의 다른 자식에게 있는 것들은 사용할 수 없다.

\*lexical environment가 접근할 수 있는 곳을 나뉘는 기준이 execution context(global execution context, function execution context)인것을 function scope라고 한다.

\*{}를 기준으로 나뉘는것을 block scope라고 하며 es6부터 생긴 const 와 let은 block scope를 따른다.

---

[How JavaScript works: an overview of the engine, the runtime, and the call stack](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)
[How JavaScript works: inside the V8 engine + 5 tips on how to write optimized code](https://engineering.huiseoul.com/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9E%91%EB%8F%99%ED%95%98%EB%8A%94%EA%B0%80-v8-%EC%97%94%EC%A7%84%EC%9D%98-%EB%82%B4%EB%B6%80-%EC%B5%9C%EC%A0%81%ED%99%94%EB%90%9C-%EC%BD%94%EB%93%9C%EB%A5%BC-%EC%9E%91%EC%84%B1%EC%9D%84-%EC%9C%84%ED%95%9C-%EB%8B%A4%EC%84%AF-%EA%B0%80%EC%A7%80-%ED%8C%81-6c6f9832c1d9)

```toc

```
