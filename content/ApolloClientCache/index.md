---
emoji: ''
title: 'Apollo Client:cache'
date: '2021-07-07 22:00:00'
author: seoyul
tags: frontEnd
categories: frontEnd
---

### InMemoryCache

- apollo client는 gql의 결과를 inMemoryCache에 저장하며 이를 통해 클라이언트는 불필요한 네트워크 요청 없이 동일한 데이터에 대해 요청할 수 있다.
    
    ![https://images.velog.io/images/zoeyul/post/4afc798e-3b20-46db-8b64-c8c887f1f17f/Screen%20Shot%202021-07-07%20at%201.45.39%20AM.png](https://images.velog.io/images/zoeyul/post/4afc798e-3b20-46db-8b64-c8c887f1f17f/Screen%20Shot%202021-07-07%20at%201.45.39%20AM.png)
    
- apollo cache는 먼저 캐시에 원하는 데이터가 있는지 확인하고 있으면 그대로 반환한다. 원하는 데이터가 없을 시 외부 서버로 해당 데이터를 요청한 후 재사용을 위해 반환되는 값을 받아오거나 직접 캐싱을 구현한다.

### 기본 캐시 설정

```tsx
import { InMemoryCache, ApolloClient } from '@apollo/client';

const client = new ApolloClient({
  // ...other arguments...
  cache: new InMemoryCache(options)
});

```

- 클라이언트를 생성할 떄 캐시 객체를 생성하여 넘겨주면 되고 options 인수로 TypePolicy의 keyFields, read, merge function을 다룰 수 있다.
- InMemoryCache는 쿼리 응답 개체를 내부 데이터 저장소에 저장하기 전에
    - 응답 받은 데이터에 포함된 모든 식별 가능한 개체에 대해 고유한 ID를 생성한다.
    - 개체를 ID별로 플랫 룩업 테이블에 저장한다.
    - 들어오는 객체가 기존 객체와 동일한 ID로 저장될 때마다 해당 객체의 필드가 병합된다.
- InMemoryCache는 기본적으로 __typename과 id 또는 _id를 결합시켜서 고유한 식별자를 만든다.(id가 아닌 다른 필드를 정의해서 설정할 수 있다.)

### TypePolicy

- TypePolicies는 서버 스키마에 정의된 모든 자료형 이름은 key값으로 설정할 수 있다.

```tsx
type TypePolicy = {
  keyFields?: KeySpecifier | KeyFieldsFunction | false;
  queryType?: true;
  mutationType?: true;
  subscriptionType?: true;
  fields?: {
      [fieldName: string]: FieldPolicy<any> | FieldReadFunction<any>;
  };
};

```

- keyFields : 해당 자료형을 분류할 때 기본키(분류 기준)로 설정할 필드
- fields : TypePolicy의 하위 필드로서 해당 필드의 읽기-쓰기 등 여러 정책을 세부적으로 설정할 수 있다. 필드 이름은 서버 스키마에 명시된 필드뿐만 아니라 클라이언트 측에서만 사용될 필드 이름을 정의할 수 있으며 클라이언트에서 보내는 쿼리의 해당 필드 옆에 @client 키워드를 넣어줘야 한다.

### FieldPolicy

```tsx
type FieldPolicy<TExisting = any, TIncoming = TExisting, TReadResult = TExisting> = {
  keyArgs?: KeySpecifier | KeyArgsFunction | false;
  read?: FieldReadFunction<TExisting, TReadResult>;
  merge?: FieldMergeFunction<TExisting, TIncoming> | boolean;
};

```

keyArgs : Query 자료형의 하위 필드에서 자주 쓰이는 항목으로 필드의 인수를 추가 분류 기준으로 설정할 수 있다. keyFields는 자료형을 구분하는 기준이고, keyArgs는 필드를 구분하는 기준으로 사용한다.

read(), merge() : 이 함수를 통해 필드를 읽거나 필드에 새로운 값을 쓰는 과정을 세부적으로 설정할 수 있다.

### 캐시 데이터 쓰기

- mutation 및 query 실행 후 cache를 업데이트하려면
    - fetchPolicy를 사용한다.
    - mutation 및 쿼리 실행 후 cache에 접근하여 테이터를 변경할 수 있다.

```tsx
   client = new ApolloClient({
      link: authLink.concat(httpLik),
      cache: new InMemoryCache({
        typePolicies: {
          HLBrand: {
            keyFields: ["ID"],
            merge: true,
            fields: {
              links: {
                keyArgs: ["input", ["filter", "sort"]],
                merge(
                  existing: Links_brand_links,
                  incoming: Links_brand_links,
                ) {
                  if (!existing) return incoming;

                  return {
                    ...incoming,
                    list: [...existing.list, ...incoming.list],
                  };
                },
              },
            },
          },
        },
      }),
    });

typePolicies: {
    Query: {
      fields: {
        feed: {
          keyArgs: ["type"],

          merge(existing, incoming, {
            args: { cursor },
            readField,
          }) {
            const merged = existing ? existing.slice(0) : [];
            let offset = offsetFromCursor(merged, cursor, readField);.
            if (offset < 0) offset = merged.length;
            for (let i = 0; i < incoming.length; ++i) {
              merged[offset + i] = incoming[i];
            }
            return merged;
          },
          read(existing, {
            args: { cursor, limit = existing.length },
            readField,
          }) {
            if (existing) {
              let offset = offsetFromCursor(existing, cursor, readField);
              if (offset < 0) offset = 0;
              return existing.slice(offset, offset + limit);
            }
          },
        },
      },
    },
  },
});

```

```tsx
  const [addBrandTagToLinkMutation] = useMutation<
    AddBrandTagToLinkVariables,
    AddBrandTagToLink
  >(addBrandTagToLinkMutationDoc, {
    update(cache, { data: result }) {
      if (result?.addBrandTagToLink) {
        cache.modify({
          id: cache.identify({ __typename: "HLLink", ID: linkID }),
          fields: {
            brandTags(existing: Links_brand_links_list_brandTags[]) {
              return [...existing, { ...result?.addBrandTagToLink }];
            },
          },
        });
      }
    },
  });

```

### 캐시 데이터 관리

- InMemoryCache는 쿼리 결과 객체에 UID를 부여해서 관리한다.
1.  쿼리 결과에 있는 객체에 고유한 ID(UID)를 생성해서 부여한다.
2. 동일한 UID를 가지는 객체는 서로 병합하며 기본적으로 기존 객체의 필드 값은 새로운 값으로 덮어쓰이며 병합하는 방식은 따로 설정할 수 있다.
3. 캐시된 데이터는 UID 를 기준으로  key-value 형식으로 분류한다. 여기서 key UID이고 value는 해당 UID를 가진 객체이다.

### 캐시 상태 확인

캐시에는 기본적으로 ROOT_QUERY 공간이 존재하는데 따로 설정하지 않을시 이곳에 쿼리 결과가 캐싱된다.

ROOT_QUERY 안에 __ref 항목처럼 캐시 레퍼런스로 저장된 데이터는 직접 읽을 수 없고 apollo client에서 제공하는 readField 함수를 사용해야한다

### UID 생성 규칙

apollo 캐시는 기본적으로 __typename 필드를 가진 객체마다 UID를 자동으로 생성해주는데 만약 해당 객체에 id 나 _id 필드가 존재하면 UID는 __typename:id 로 설정되고 그런 필드가 없으면서 keyField도 정의되지 않는다면 캐시에 따로 저장되지 않고 ROOT_QUERT에 값으로 저장된다. 반환된 데이터의 효율적인 캐싱을 위해 요청할 때 클라이언트측에서 각 객체 필드에 id 필드를 넣어주는 것이 좋다.

- 객체 내부에 다른 객체를 가진다면 apollo client는 해당 객체 데이터를 객체 안에 저장하지 않고 캐시 내부에 따로 저장 후 __ref 를 통해 가리키는 방식으로 데이터를 관리한다.

### keyFields

만약 서버에서 각 자료형에 대해 id 필드를 지원하지 않거나 테이터 분류 기준을 세부적으로 설정하고 싶으면 keyFields를 이용할 수 있다.

동일한 key 값을 가지는 객체는 서로 병합되며 keyFields는 문자열 배열 문자열 배열을 반환하는 함수, false로 설정할 수 있다.

### 캐시 데이터 읽기

```tsx
const cache = new InMemoryCache({
  typePolicies: {
    Person: {
      fields: {
        name: {
          read(name) {
            // Return the cached `name`, transformed to upper case
            return name.toUpperCase();
          },
        },
      },
    },
  },
});
```

### apollo client ssr mode

```tsx
import { useMemo } from "react";
import { ApolloClient, HttpLink, InMemoryCache } from "@apollo/client";
import merge from "deepmerge";

let apolloClient: ApolloClient<NormalizedCacheObject> = null;

function createApolloClient() {
  return new ApolloClient({
    ssrMode: typeof window === "undefined",
    link: new HttpLink({
      uri: "<https://nextjs-graphql-with-prisma-simple.vercel.app/api>", // 서버 URL (상대 주소가 아닌 절대 주소를 써야한다.)
      credentials: "same-origin", // `credentials`나 `headers`같은 추가적 fetch() 옵션
    }),
    cache: new InMemoryCache(),
  });
}
export function initializeApollo(initialState = null) {

  const _apolloClient = apolloClient ?? createApolloClient();
  // Next.js에서 Apollo Client를 이용해 데이터를 가져오는 함수가 있다면, 초기 상태값이 여기에서 합쳐진다.
  if (initialState) {
    // 클라이언트에서의 받은 데이터인 현재 캐시 데이터를 가져온다.
    const existingCache = _apolloClient.extract();
    // 현재 캐시와 SSR 메소드인 getStaticProps/getServerSideProps로 부터 받은 데이터를 합친다.
    const data = merge(initialState, existingCache);
    // 합쳐진 데이터를 저장한다.
    _apolloClient.cache.restore(data);
  }
  // SSG(Server Side Generation)와 SSR(Server Side Rendering)은 항상 새로운 Apollo Client를 생성한다.
  if (typeof window === "undefined") return _apolloClient;
  // 클라이언트의 Apollo Client는 한 번만 생성한다.
  if (!apolloClient) apolloClient = _apolloClient;
  return _apolloClient;
}
export function useApollo(initialState) {
  const store = useMemo(() => initializeApollo(initialState), [initialState]);
  return store;
}

```

```tsx
import { ALL_POSTS_QUERY } from "@/~";
import { allPostsQueryVars } from "@/~";
import { initializeApollo } from "../lib/apolloClient";

const Home = () => {
  // SSR에서 사용한 query와 같은 query(ALL_POSTS_QUERY)를 사용
  const { loading, error, data } = useQuery(ALL_POSTS_QUERY, {
    variables: allPostsQueryVars,
  });
};

export const getServerSideProps: GetServerSideProps<{}, {}> = async (ctx) => {
  /*
  1. 투표 등 실시간성 데이터가 계속 업데이트 되어야하는 경우 getStaticProps와 revalidate를 이용
  2. 일반적인 경우 getServerSideProps를 이용
  */
  // browser단의 context(headers)를 SSR에 넘기는 과정
  const apolloClient = initializeApollo(null, ctx);
  await apolloClient.query({
    query: ALL_POSTS_QUERY,
    variables: allPostsQueryVars,
  });
  return {
    props: {
      initialApolloState: apolloClient.cache.extract(),
    },
  };
};
export default Home;

```


```toc

```
