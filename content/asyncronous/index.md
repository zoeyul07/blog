---
emoji: ''
title: 'About Javascript: 비동기'
date: '2022-01-08 23:00:00'
author: seoyul
tags: frontEnd javascript
categories: FrontEnd Javascript
---

## Javascript: 비동기

자바스크립트에서 비동기를 사용하려면 1. callback, 2. promise, 3. async function 을 사용할 수 있다.

### callback 의 문제점

promise 이전 콜백만을 사용해 비동기 코드를 작성해왔지만 이 방식은 callback 지옥이라는 문제가 발생할 수 있다. callback 의 무한 중첩으로 코드의 맥락과 흐름 파악에 어려움을 느낄 수 있고 콜백 뎁스에 추가적인 로직이 들어간다면 코드는 더 복잡해지게 된다.(콜백의 경우 코드 진행이 순차적이 지 못해 코드의 컨텍스트 및 흐름 파악에 어려움이 있을 수 있다.)

어떠한 함수에 콜백을 넘기면 콜백에 대한 제어권은 호출부를 떠나 그 함수에 넘어가게 되며 이를 제어의 역전 (IoC, Inversion Of Control)라고 한다. 콜백 체계에는 코드의 순차성, 빋음성이 결여되는 결함이 생기며 콜백 지옥이 펼쳐질 수 기때문에 그 문제점을 해결하기 위새 ES6부터 promise 가 도입되었다. promise는 콜백에서 발생할 수 있는 순차성과 믿음성 결여를 해결한다.

### promise

#### 순차성 해결

프로미스를 사용하면 호출부로 프로미스 인스턴스가 반환되머 인스턴스에 체이닝 되어있는 then은 이를 수신하고 로직을 비동기적으로 실행할 수 있게 되며 제어권을 호출부로 되찾아오게 된다.

```javascript
new Promise((resolve, rejecct) => {
  resolve()
}).then(() => {
    foo()
  },
   (error) => {
    fooError()
  }
).then...
```

#### 믿음성 해결

같은 작업인데도 콜백이 어떨때는 동기적으로 어떨때는 비동기적으로 호출되고 종결되어 경합조건(race condition)에 이르는 경우가 있다. 프로미스는 복잡한 관용 코드 없이 로직을 비동기적으로 가져갈 수 있다.

#### Promise.race

로직이 잘못되거나 네트워크의 상태가 좋지 않는 등의 이유로 비동기 로직이 종결되지 않아 프로미스가 귀결되지 않는 경우가 있는데. 이 경우 then 에 걸려있는 콜백이 호출되지 않는다. 이러할 경우 경합을 추상화한 Promise.race를 이용한 프로미스 타임아웃 패텬을 이용해 해결할 수 있다.

```javascript
Promise.race([bar(), timeoutPromise(3000)]).ten(
function(){
  //bar() 프로미스가 제 시간에 이행되어 귀결
}
function(err){
  //bar() 프로미스가 거부로 귀결 혹은 시간 안에 귀결되지 못해 타임 아웃 프로미스가 먼저 귀결
}
)
```

#### 콜백은 단 한번만 호출된다.

프로미스는 이행이던 거부이던 최초 한번만 귀결되기 때문에 then에 등록된 콜백 또한 한번씩만 호출된다.

```javascript
const promise = new Promise(fucntion(resolve, reject){
                              let value = 0

                              setInterval(function() {
                              value+=1
                              resolve(value, "in promise")
                              }, 1000)
                            })

 setInterval(function(){
   promise.then(function(value{
                          console.log(value, "in callback")
                          },1000))
                         })

// 1 in promise
//1 in callback
//2 in promise
//1 in callback
//3 in promise
//1 in callback
.....

```

```toc

```
