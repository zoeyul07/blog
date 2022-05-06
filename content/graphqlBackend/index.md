---
emoji: ''
title: 'Graphql: backend'
date: '2020-09-21 22:00:00'
author: seoyul
tags: backend
categories: BackEnd
---

## GraphQL

- 클라이언트가 필요한 데이터를 정확하게 특정하여 API에 요청하는 선언적인 데이터 불러오기를 가능하게 만드는 것이다. 

- 고정된 형태의 데이터 구조를 반환하는 엔드포인트를 여러개 제공하는 것이 아니라 단 하나의 엔드포인트만을 노출시키고, 클라이언트가 요청한 데이터들만을 정확하게 반환한다. (필요한 데이터가 무엇인지 서버에 알려주기 위해서 클라이언트가 보다 많은 정보를 보내야 한다.)

- GraphQL은 데이터베이스 관련 기술이 아닌 API를 위한 쿼리 언어이다. 데이터베이스의 종류와 상관 없이 작동하며, 효율적으로 사용될 수 있다.

- 모바일 사용의 증가로 인한 효율적인 데이터 로딩의 필요성을 위해서 네트워크 상에서 전송되어야 하는 데이터의 양을 최소화하고 어플리케이션의 퍼포먼스를 대체적으로 향상시켜준다.

- UI를 조금이라도 바꾸면 필요한 데이터가 변할수도 있기 때문에 그럴 경우 백엔드에서 대응해야 하는데, graphql을 사용하면 서버에서 추가로 작업하지 않더라도 클라이언트를 수정할 수 있다.

#### Client

- 주로 사용되는 graphql 클라이언트로는 apllo, relay가 있다. apollo는 graphql 클라이언트를 모든 주요 개발 플랫폼에서 사용할 수 있도록 하겠다는 방향성 아래 형성된 커뮤니티에 의해서 개발이 이루어지고 있고, relay는 페이스북이 개발한 graphql 클라이언트로 성능 최적화가 이루어져 있으며, 오직 웹에서만 사용 가능하다.

### GraphQL로 해결할 수 있는 문제
- 개발자가 어떤 정보를 원하는지에 대해 통제할 수 있다.

#### over-fetching
- 요청한 영역의 정보보다 많은 정보를 서버에서 전달받는다. (응답 데이터에는 실제로 필요없는 정보가 포함되어 있을 수 있다.)

#### under-fetching
- 어떤 하나를 완성하기 위해 다른 요청을 해야할 경우 (REST에서 하나를 완성하기 위해 많은 소스를 요청한다.)

- 특정 엔드포인트가 필요한 정보를 충분히 제공하지 못하는 경우를 말하며 필요한 정보를 모두 확보하기 위해서 추가적인 요청을 보내야 한다.

## Schema
- 사용자에게 보내거나 사용자로부터 받을 데이터에 대한 설명으로 schema.graphql 파일에서 사용자가 뭘 할지에 대해서 정의한다. (API가 할 수 있는 행위를 정해주고, 클라이언트가 데이터를 요청하는 방법을 정의한다.)

- query는 단지 정보를 받을때만 쓰인다.

- mutation(변형)은 정보를 변형할 때, 서버에서 혹은 데이터베이스에서, 메모리에서 정보를 바꾸는 작업을 할 때를 사용된다.(백엔드에 저장된 데이터를 수정하느느 것으로 create, update, delete를 수행한다.)

```graphql
//graphql/schema.graphql
//어떤 사용자가 query에 name을 보내면 string을 보낸다는 설명

type Seoyul {
  name: String!
  age: Int!
  gender: String!
}

type Query {
  person: Seoyul!
}


```
## Resolver
- query를 resolve 한다. 쿼리는 데이터베이스에게는 문제와 같으며 이 쿼리를 어떤 방식으로 resolve 해야한다.

- 그래프큐엘 서버를 구현할 때 각각의 필드들은 resolver라고 불리는 함수에 대응하게 되는데, 이 함수의 목적은 해당 필드를 위한 데이터를 불러오는 것이다.

- 서버가 쿼리를 받았을 때 쿼리 페이로드 상에 명시된 각 필드들에 대한 함수들을 모두 호출하게 되는데, 이를 통해 쿼리를 resolve하고 각 필드에 대하여 올바른 데이터를 반환할 수 있게 된다. 

- 모든 resolver들이 값을 반환하면 서버는 쿼리 상에 서술된 형태로 데이터들을 포장한 뒤 클라이언트에 반환해준다.

```js
//graphql/resolvers.js
//사용자가 name query를 보내면 seoyul을 반환하는 함수로 답한다.

const seoyul = {
  name: "Seoyul",
  age: 27,
  gender: "female"
}

const resolvers = {
  Query: {
    person: () => seoyul
  }
}

export default resolvers
```

> GraphQL Playground 
- Postman과 비슷하게 데이터베이스를 테스트할 수 있으며 localhost:4000에서 확인 가능하다.

## index.js
```js
import {GraphQLServer} from "graphql-yoga";
import resolvers from "./graphql/resolvers"

const server = new GraphQLServer({
  typeDefs: "graphql/schema.graphql",
  resolvers
})

server.start(() => console.log("graphql sever running")) 
```
***
## 특정 아이디 값을 갖고 있는 데이터 불러오기
- graphql resolvers는 graphql 서버에서 요청을 받으며 graphql 서버가 query나 mutation의 정의를 발견하면 resolver를 찾고 해당 함수를 실행한다.

```js
//db.js

export const people = [
  {
    id: 0,
    name: "Seoyul",
    age: 27,
    gender: "female"
  },
  {
    id: 1,
    name: "hello",
    age: 27,
    gender: "female"
  },
  {
    id: 2,
    name: "my",
    age: 27,
    gender: "female"
  },
  {
    id: 3,
    name: "name",
    age: 27,
    gender: "female"
  },
  {
    id: 4,
    name: "is",
    age: 27,
    gender: "female"
  },
  {
    id: 5,
    name: "Seoyul",
    age: 27,
    gender: "female"
  }
]

export const getById = id => {
  const filterPeople = people.filter(person => person.id === id)
  return filterPeople[0]
}

```

```graphql
//schema.graphql
type Person {
  id: Int!
  name: String!
  age: Int!
  gender: String!
}

type Query {
  people: [Person]!
  person(id: Int!): Person
}

```

```js
//resolvers.js
import {getById, people} from "./db"

const resolvers = {
  Query: {
    people: () => people,
    person: (_, {id})=> getById(id)
  }
}
 
export default resolvers
```
- 위와 같이 파일들을 생성하고 서버를 돌려 localhost:4000에서 다음과 같이 쿼리를 날린다.
```graphql
{
  person(id:3){
    name
  }
}
```

- 그러면 다음과 같은 결과가 나온다.
```graphql
{
  "data": {
    "person": {
      "name": "name"
    }
  }
}
```
## Mutation
- 데이터베이스 상태가 변할 때 사용된다.
```
//shema.graphql

type Movie {
  id: Int!
  name: String!
  score: Int!
}

type Query {
  movies: [Movie]!
  movie(id: Int!): Movie
}

type Mutation {
  addMovie(name: String!, score: Int!): Movie!
  deleteMovie(id: Int!): Boolean!
}

```

```
//resolvers.js

import {getMovies, getById, addMovie, deleteMovie } from "./db"

const resolvers = {
  Query: {
    movies: () => getMovies(),
    movie: (_, {id})=> getById(id)
  },
  Mutation:{
    addMovie:(_, {name, score})=> addMovie(name, score),
    deleteMovie:(_, {id}) => deleteMovie(id)
  }
};
 
export default resolvers
```

```
//db.js

let movies = [
    {
      id: 0,
      name: "Star Wars - The new one",
      score: 1
    },
    {
      id: 1,
      name: "Avengers - The new one",
      score: 8
    },
    {
      id: 2,
      name: "The Godfather I",
      score: 99
    },
    {
      id: 3,
      name: "Logan",
      score: 2
    }
  ];
  
  export const getMovies = () => movies;
  
  export const getById = id => {
    const filteredMovies = movies.filter(movie => movie.id === id);
    return filteredMovies[0];
  };
  
  export const deleteMovie = id => {
    const cleanedMovies = movies.filter(movie => movie.id !== id);
    if (movies.length > cleanedMovies.length) {
      movies = cleanedMovies;
      return true;
    } else {
      return false;
    }
  };
  
  export const addMovie = (name, score) => {
    const newMovie = {
      id: `${movies.length}`,
      name,
      score
    };
    movies.push(newMovie);
    return newMovie;
  };

```

## subscription
- subscription은 주로 실시간(real-time) 애플리케이션을 구현하기 위해서 사용된다.

- query와 mutation은 서버/클라이언트(server/client) 모델을 따르는 반면에, subscription은 발행/구독(publish/subscribe) 모델을 따른다. (서버와의 실시간 통신을 통한 업데이트를 위햐여 구독이라는 개념을 제공한다.)

- 클라이언트가 어떤 이벤트를 구독하면 서버와 지속적인 연결을 형성하고 유지하게되며 특정 이벤트가 발생했을 시 서버는 대응하는 데이터를 클라이언트에 푸시해준다.

- 쿼리문, 뮤테이션과 동일한 문법을 사용하여 작성되며, 새로운 뮤테이션이 이루어질때마다 서버는 새로운 정보를 클라이언트에 전송하게 된다.

***
[graphql-yoga github](https://github.com/prisma-labs/graphql-yoga)

```toc

```
