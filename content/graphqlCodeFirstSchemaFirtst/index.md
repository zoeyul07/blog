---
emoji: ''
title: 'GraphQL: code-first vs schema-first'
date: '2021-03-10 23:00:00'
author: seoyul
tags: backend
categories: BackEnd
---


## Graphql Schema
graphql을 사용하면서 schema를 바탕으로 code가 매치되게 할 수 있고, 코드를 바탕으로 schema가 생성되게 할 수 있다.

두 케이스 모두 graphql 서비스의 작동을 하게 되지면 어떤 접근법을 사용하느냐에 따라 더 많은 기능을 사용할수도 사용하지 못하게 될수도 있고, 더 쉽거나 어렵게 작업하게 될 수도 있다. 

### Schema First
schema first는 graphql의 schema를 먼저 정의하고 그 schema 정의에 맞게 코드를 작성하는 방법을 말한다. schema를 작성하기 위해서는 그래프큐엘의 data model을 나타내기 위해 만들어진 SDL(Schema Definition Language)를 사용하며 그렇기 때문에 schema first라고 불린다.

```graphql

#title은 string으로 된 value를 갖게 되며, !로 항상 반드시 가져야하는 value라는 의미를 나타낼 수 있다.

type Film {
  id: Int
  title: String!
}
```

graphql이 그래프 형태의 관계들을 쿼리하여 사용할 수 있는 만큼 다음과 같이 타입으로 관계를 지정할 수 있다.

```graphql
type Film {
  id: Int
  title: String!
  director: Person!
  actors(limit: Int = 10): [Person]
}

type Person {
  id: Int
  name: String!
}
```

film이 많은 actor들을 가직 수 있는 만큼 리스트의 형태로 표현되며 limit의 default value를 통해 몇개의 요소들을 return 할 것인지 제한할 수 있다.

위의 예제를 통해 알 수 있듯 SDL의 가장 큰 장점은 간단하고 명확하며, 익숙하거나 익숙하지 않는 사람 모두에게 이해하기 쉬우며 data model을 생성하는데 있어서 팀 가운데 커뮤니케이션 툴처럼 사용될 수 있다.

하지만 위에서 보여주는 SDL의 단점은 필드의 값들을 계산할 실제 코드를 나타내는 resolver를 포함하지 않는다는 것이다. SDL은 그 자체만으로 전부가 될 수 없으며 스키마 정의와 resolution을 나누어 사용하게 된다.

두번째 단점으로는 resolver 코드가 SDL의 정의와 정확하게 일치해야한다는 점이다.(String!으로 정의했다면 String이나 Int!가 아닌 정확히 String!이여야한다.)

#### 장점
- ASynchronous한 개발 가능
- 확실한 의사소통의 수단
- SDL을 제외한 document가 필요 없음
- 빠른 mocking으로 인해 개발 속도가 빨라짐

#### 단점
- SDL과 Resolver간의 일치성을 보장할 수 없음
- run time에서만 알 수 있는 단점
- schema가 늘어갈 수록 실수의 가능성이 높아짐
- 다양한 도구들로 문제를 해결하려 하지만 근본적인 해결책이 되지 않는다.

### Code First
code first 접근 방식에서는 resolver를 먼저 작업함으로서 시작하게 된다. 그 후 그 코드로 부터 schema를 generate하게 된다. 그렇기 때문에 schema를 여전히 갖게 되지만 수동적으로 생성하지 않고 script를 통해 생성하게 되지만 코드만으로 schema를 이해하기란 SDL의 정의를 보는것 만큼 이해하기 쉽지 않다. 

#### 장점
- schema와 resolver간의 type safety를 보장
- 자신이 편한 언어로 개발 가능
	- 자신이 사용하는 언어의 다양한 기능을 사용 가능
    - 낮은 learning curve
- 코드 중복의 최소화

### schema first? code first?
- 두가지 방법으로 모두 graphql을 사용할 수 있으나 어떤 방법을 사용할지는 초기에 결정하는 것이 좋으며 사용하는 언어나 프레임워크에 따라 두가지의 옵션이 모두 제공될수도 한가지만 사용 가능할 수도 있다.

#### schema firtst가 나은 이유
- TDD(test-driven-development)와 비슷하다. 개발자는 처음 작업을 시작할 때 반드시 여러 use case와 사용자 편의성을 고려해야하며 마지막 결과는 더 모듈화되고 유지보수 가능한 디자인을 갖을 수 있다.

- DIP(Dependency Inversion Principle), 의존성 역전 원칙을 따른다.
> 👉🏻 DIP란?
상위 모듈은 하위 모듈의 구현에 의존해서는 안되며 하위 모듈이 상위 모듈이 정의한 추상에 의존해야 한다.
DIP에서 역전이란 실제 호출 및 사용관계의 역전이 아닌 절차적 흐름의 구조적 디자인에서 발생하던 하위 레벨의 변경이 상위 레벨에 미치는 변경을 끊자는 의미이다. 상위 모듈에서 하위 모듈의 구현을 직접 의존하는 것이 아닌 하위 모듈이 상위 모듈이 정의한 추상을 구현하게 되므로 역전이라고 한다.

- schema는 api의 mock data를 사용할수 있게 하므로 백엔드의 api를 기다리지 않고 동시에 client가 작업에 들어갈 수 있다.

#### schema first가 나은 이유
- code first는 schema first에 지원하는 모든 기능들을 예외 없이 지원하며 여러 tool(1. schema definition과 resolver의 sync가 항상 일치할 수 있도록 해야한다. 2. graphql type definition을 여러 파일로 나누어 정리해야한다.)에 의존할 필요 없이 노력을 덜 들이고 사용할 수 있다.

```toc

```
