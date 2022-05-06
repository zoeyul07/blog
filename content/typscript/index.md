---
emoji: ''
title: 'Typescript'
date: '2020-08-28 22:00:00'
author: seoyul
tags: language
categories: language
---

## TypeScript
- 타입스크립트는 자바스크립트 처럼 생겼고 컴파일 했을 때 자바스크립트로 컴파일된다.

- 자바스크립트는 엄격한 규칙이 없고 사용하기 쉬우며 우리가 원하는 방향으로 수정하기 때문에 인기가 많은데 큰 프로젝트를 하거나 팀으로 일하거나 버그를 최소화하고 싶을 경우에는 단점이 될 수 있다.

- 타입스크립트는 이러한 단점을 보완하며 언어가 예측 가능하고 읽기 쉬운 코드 등 자바스크립트를 더 잘 사용할 수 있게 한다.

- 타입스크립트는 typed 언어로 어떤 종류의 변수와 데이터인지 설정해줘야 한다.

- 만약 해당 타입에 맞지 않게 설정하면 에디터 상에서 오류가 나타나게 된다.

```
const message: string = "hello world";
console.log(message);

let count = 0;
count += 1;
//다음은 에러가 난다
count = "문자열";

const message: string = "hello world";

const done: boolean = true;

const numbers: number[] = [1, 2, 3]; // 숫자 배열
const message: string[] = ["hello", "world"]; //문자 배열

message.push(1); //숫자는 안된다.

let mightBeUndefined: string | undefined = undefined; //string일수도 잇고 undefined 일수도 있다
let nullableNumber: number | null = null; // number일수도 null일수도 있다

let color: "red" | "orange" | "yellow" = "red"; //red, orange, yello중 하나이다.
color = "yello";
color = "green"; // 에러 발생
```

- 특정 변수 또는 상수의 타입을 지정할 수 있고 사전에 지정한 타입이 아닌 값이 설정될 때 바로 에러를 발생시키며 그때는 컴파일이 되지 않는다.

### 함수에서 타입 정의하기

```
function sum(x:number, y:number): numver{
  return x+y;  
}

sum(1,2);

```

- 타입 스크립트를 사용하면 코드를 작성하는 과정에서 함수의 파라미터로 어떠 타입을 넣어야 하는지 바로 알 수 있다.

- 가장 우측의 : number 는 해당 함수의 결과물이 숫자라는것을 명시하는 것이다. 이 상태에서 결과물이 number가 아닌 null을 반환한다면 오류가 뜨게된다.

```

function sumArray(numbers:number[]):number{
  return numbers.reduce((acc, current)=>acc_currend, 0)
}

const total=sumArray({1,2,3,4,5});
```
- 타입스크립트를 사용하면 배열의 내장함수를 사용할 때에도 타입 유추가 매우 잘 이루어진다.

- 함수에서 만약 아무것도 반환하지 않아야 한다면 반환 타입을 void 로 설정하면 된다.
```
function returnNothing(): void {
  console.log('I am just saying hello world');
}
```

### interface

- interface는 클래스 또는 객체를 위한 타입을 지정할 때 사용되는 문법이다.

```
interface Shape {
  getArea(): number; // Shape interface 에는 getArea 라는 함수가 꼭 있어야 하며 해당 함수의 반환값은 숫자이다.
}

class Circle implements Shape {
  //implements 키워드를 사용하여 해당 클래스가 Shape interface의 조건을 충족하겠다는 것을 명시한다

  radius: number; //멤버변수 radius 값을 설정한다.

  constructor(radius: number) {
    this.radius = radius;
  }

  getArea() {
    return this.radius * this.radius * Math.PI;
  }
}

class Rectangle implements Shape {
  width: number;
  height: number;
  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }

  getArea() {
    return this.width + this.height;
  }
}

const shapes: Shape[] = [new Circle(5), new Rectangle(10, 5)];

shapes.forEach((shape) => {
  console.log(shape.getArea());
});
```
- 멤버 변수를 선언한 다음에 constructor에서 해당 값들을 하나하나 설정해주었는데 타입스크립트에서는 constructor의 파라미터쪽에 public이나 private accessor를 사용하면 하나하나 설정 해주는 작업을 생략할 수 있다.

```
class Rectangle implements Shape {
  constructor(private width: number, private height: number) {
    this.width = width;
    this.height = height;
  }
  getArea() {
    return this.width * this.height;
  }
}
```

- public으로 선언된 값은 클래스 외부에서 조회할 수 있으며 private으로 선언된 값은 클래스 내부에서만 조회할 수 있다.

### 일반 객체를 interface로 타입 설정하기

```
interface Person {
  name: string;
  age?: number; //물음표는 설정해도 되고 안해도 되는 값이라는 의미이다.
}

interface Developer {
  name: string;
  age?: number;
  skills: string[];
}

const person: Person = {
  name: "김사람",
  age: 20,
};

const expert: Developer = {
  name: "김개발",
  skills: ["javascript", "react"],
};

const people: Person[] = [person, expert];

```

- Person과 Developer처럼 형태가 유사한 interface를 선언할 때는 다른 interface를 extends 키워드를 사용해서 상속받을 수 있다.

```
interface Person {
  name: string;
  age?: number; 
}

interface Developer extends Person {
  skills: string[];
}
```

### Type Alias
- type은 특정 타입에 별칭을 붙이는 용도로 사용하며 객체를 위한 타입을 설정할 수 도 있고, 배열 또는 그 어떤 타입이던 별칭을 지어줄 수 있다.

```
type Person={
  name: string;
  age?:number;
}

//& 은 Intersction으로 두개 이상의 타입을 합쳐준다.
https://www.typescriptlang.org/docs/handbook/advanced-types.html#intersection-types
type Developer=Person &{
  skills:string[];
}

const person: Person={
  name:"김사람"
}

const expert: Developer{
  name:"김개발",
  skills:["javascript", "react"]
};

type People=Person[];//// Person[] 를 이제 앞으로 People 이라는 타입으로 사용 할 수 있다.
const people:People=[person, expert];

type Color ="red" | "orange" | "yellow";
const color:Color="red";
const colors:Color[]=["red", "orange"]
```
- 클래스와 관련된 타입의 경우엔 interface를 사용하는 것이 좋고, 일반 객체의 타입의 경우엔 그냥 type을 사용해도 무방하다.

- 객체를 위한 타입을 정의할 때 무엇이든 써도 상관 없지만 일관성 있게 사용하도록 한다.

### Generics
- 함수, 클래스, interface, type alias를 사용하게 될 때 여러 종류의 타입에 대하여 호환을 맞춰야 하는 상황에서 사용하는 문법이다.

#### 함수에서 사용하기
- 객체 A와 객체 B를 합쳐주는 merge라는 함수를 만든다고 할 때 A 와 B가 어떤 타입이 올 지 모르기 때문에 이런 상황에서는 any라는 타입을 쓸 수도 있다.

```
function merge(a: any, b: any): any {
  return {
    ...a,
    ...b,
  };
}

const merged = merge({ foo: 1 }, { bar: 1 });
```

- 그렇지만 이렇게 하면 타입 유추가 모든 깨진것이니 이때는 Generic를 사용한다.

```

function merge<A, B>(a: A, b: B): A & B {
  return {
    ...a,
    ...b,
  };
}
```
```
function wrap<T>(param:T){
  return {
    param
  }
}

const wrapped=wrap(10)
```

- 함수에서 Generics를 사용하면 파라미터로 다양한 타입을 넣을 수도 있고 타입 지원을 지켜낼 수 있다.

#### interface에서 Generics 사용하기

```
interface Items<T>{
  list:T[;]
}

const items:Items<string>={
  list:["a", "b", "c"]
}
```

#### type에서 Generics 사용하기
```
type Items<T>{
  list:T[];
}

const items: Items<string>={
  list:["a", "b", "c"]
}
```

#### 클래스에서 Generics 사용하기
- Queue는 데이터를 등록할 수 있는 자료형이며 먼저 등록(enqueue)한 항목을 먼저 뽑아올 수 (dequeue)있는 성질을 가지고 있다.
```
class Queue<T> {
  list: T[] = [];
  get length() {
    return this.list.length;
  }
  enqueue(item: T) {
    this.list.push(item);
  }
  dequeue() {
    return this.list.shift();
  }
}

const queue = new Queue<number>()
```


***
[벨로퍼트와 함께하는 모던 리액트](https://react.vlpt.us/using-typescript/01-practice.html)
[Interface vs Type alias in TypeScript 2.7](https://medium.com/@martin_hotell/interface-vs-type-alias-in-typescript-2-7-2a8f1777af4c)
[TypeScript에서 Type을 기술하는 두 가지 방법, Interface와 Type Alias](https://joonsungum.github.io/post/2019-02-25-typescript-interface-and-type-alias/)
[React Typescript Tutorial](https://www.youtube.com/watch?v=Z5iWr6Srsj8)
[Creating a Todo List App in React using Typescript (Part 1)](https://www.youtube.com/watch?v=ODvirqIC09A&t=349s)
[React With Typescript Crash Course - How To Use React With Typescript](https://www.youtube.com/watch?v=e5c_RfYqIhU)
[Total React Tutorial - For Beginners - Using TypeScript](https://www.youtube.com/watch?v=nCoQg5qbLcY)
[Use TypeScript with React Components](https://www.youtube.com/watch?v=9lUN3sqAjQQ)
[React with TypeScript: Best Practices](https://www.sitepoint.com/react-with-typescript-best-practices/)
[10++ TypeScript Pro tips/patterns with (or without) React](https://medium.com/@martin_hotell/10-typescript-pro-tips-patterns-with-or-without-react-5799488d6680#78b9)
[Styled-component #3 with typescript](https://velog.io/@hwang-eunji/styled-component-typescript)
[[TypeScript] Advanced Types(고급 타입)](https://velog.io/@zeros0623/TypeScript-고급-타입)
[Working with React context providers in Typescript](https://dev.to/mitchkarajohn/working-with-react-context-providers-in-typescript-1fk0)
[How to use React useRef with TypeScript](https://linguinecode.com/post/how-to-use-react-useref-with-typescript)

```toc

```
