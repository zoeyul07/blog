---
emoji: ''
title: 'Front Test (feat:Cypress)'
date: '2022-01-05 22:00:00'
author: seoyul
tags: frontEnd
categories: FrontEnd
---

## 테스트의 기본

- 테스트 케이스는 어떠한 가정을 통해 참과 거짓을 반환하며 가정이 변하지 않는 이상 테스트의 결과는 항상 동일해야한다.
- 가정은 테스트 케이스의 의도를 포함하며 각 테스트 케이스 하나당 하나의 의도만을 포함해야한다.

## 테스트 분류

### Unit Test (단위 테스트)

- 모듈(함수/클래스) 단위의 테스트한다.
- 테스트할 부분의 코드를 다른 시스템으로부터 분리(isloate)시킨 채 테스트한다.
- 작성 비용이 적게 들고 **실행 속도가 빠르다**
- 실패했을 때 문제가 생긴 부분을 비교적 정확하게 파악할 수 있다.
- 경우에 따라 한두개의 단위를 모아서 하나의 단위로 취급하기도 한다.

### Integration Test (통합 테스트)

- 주로 단위 테스트보다 큰 범위의 테스트를 의미한다.
- 개별 모듈(함수/클래스)들이 연결되어 제대로 상호작용하는지를 테스트
- 단위 테스트에 비해 작성이 어렵고 실행 속도가 느리다
- 단위 테스트에 비해 실패 시 문제가 생긴 부분을 정확히 파악하기가 어렵다

### End-To-End Test (e2e-test)

- 실제 사용자가 사용하는 것과 같은 조건에서 전체 시스템을 테스트한다.
- API 서버, DB 등의 외부 서비스들을 모두 사용하여 통합된 시스템을 테스트한다.
- 단위/통합 테스트에 비해 작성이 어렵고 실행 속도가 가장 느리다
- 문제가 생긴 부분을 정확히 파악하기가 가장 어렵다
- 기능(Functional) 테스트와 비슷한 의미로 사용된다.

## 테스트 pattern

### Given-When-Then (Arrange-Act-Assert)

- BDD(Behavior-Driven-Develope) 중 하나
- **Given :** What software will look like to user (= pre-conditions)
- **When :** Things that the user will do (= event or operation)
- **Then :** What the user should expect (= post-conditions)

```jsx
기능 : 사용자 주식 트레이드

시나리오 : 트레이드가 마감되기 전에 사용자가 판매를 요청
  
"Given" 나는 MSFT 주식을 100가지고 있다. 
        그리고 나는 APPL 주식을 150가지고 있다. 
		그리고 시간은 트레이드가 종료되기 전이다.

"When"  나는 MSFT 주식 20을 팔도록 요청했다.
     
"Then"  나는 MSFT 주식 80 가지고 있어야 한다.
		그리고 나는 APPL 주식 150을 가지고 있어야 한다.
		그리고 MSFT 주식 20이 판매 요청이 실행되었어야 한다.
```

```jsx
it('should $1', () => {
  // Given
  const data = $4

  // When
  const result = $3

  // Then
  expect(result).toEqual($2)
})
```

1. 테스트 케이스의 목적을 분명히한다.
2. 테스트 케이스에서 확인해야하는 부분, 예상하는 값을 작성한다.
3. 어떠한 flow를 검증할지 작성한다.
4. 3번 검증을 위해 어떤 가정이 필요한지 작성한다.

## TDD? BDD?

TDD(Test-Driven-Development) 와 BDD(Behaviour-Driven-Developement) 는 애자일 방법론에서 가장 널리 쓰이는 것들이다.

TDD는 테스트 자체에 집중하여 개발하는 반면 BDD는 비즈니스 요구사항에 집중하여 테스트 케이스를 개발한다.

### TDD

요구사항에 맞는 테스트 케이스를 우선 작성한 다음 각 테스트를 통과하기 위한 최소한의 코드를 작성하고 리팩토링 하는 프로그래밍 방식으로 테스트 단위는 함수 단위로 매우 작고 거의 모든 함수가 테스트 대상에 포함된다. 모듈 크기를 작게헤고 모듈간 의존성을 작게한다.

ex) add(1,1)이 2인지 확인

### BDD

TDD에서 파생된 개녕으로 BDD에서는 사용자의 행위를 작성한 시나리오를 기반으로 테스트 케이스를 작성하며 함수 단위의 테스트를 권장하지 않는다. 하나의 시나리오는 초기 설정값(Given), 실행 조건(When), 기대결과(Then) 구조를 기본 패턴으로 갖으며 테스트 대상의 상태 변화를 테스트한다. 

ex) 사용자가"="눌렀을 때 1+1의 값 2가 화면에 표시 되는지 확인

## Cypress Tip

### share code between UI and test

```jsx
import { Ids } from '../../../src/app/constants';
//can share Id values between UI and Test to make sure the CSS selectors don't break

cy.get(`#${Ids.username}`)
  .type('john')
```

- cypress는 컴파일 되어 브라우저에서 실행되기 때문에 프로젝트 코드를 import 해와 테스트에서 사용할 수 있다.

### Creating Page Objects

```jsx
import { Ids } from '../../../src/app/constants';

class LoginPage {
  visit() {
    cy.visit('/login');
  }

  get username() {
    return cy.get(`#${Ids.username}`);
  }
}
const page = new LoginPage();

// Later
page.visit();

page.username.type('john');
```

- 페이지에서 필요한 인터렉션을 여러 테스트에서 사용할 수 있도록 만든다.

### Explicit assertion

```jsx
cy.get('#foo')
  .should('have.text', 'something')

cy.get('div')
  .should(($div) => {
    expect($div).to.have.length(1);
    expect($div[0].className).to.contain('heading');
  })
// This is just an example. Normally you would `.should('have.class', 'heading')
```

- selector를 통해 id나 classname을 가져와서 assertion을 실행할 수 있다.

### Commands and Chaining

```jsx
// Don't do this
cy.get(/**something*/)
  .should(/**something*/)
  .click()
  .should(/**something*/)
  .get(/**something else*/)
  .should(/**something*/)

// Prefer separating the two gets
cy.get(/**something*/)
  .should(/**something*/)
  .click()
  .should(/**something*/)

cy.get(/**something else*/)
  .should(/**something*/)
```

### Using Contains for easier querying

```jsx
cy.get('#foo')
  // Once #foo is found the following:
  .contains('Submit')
  .click()
  // ^ will continue to search for something that has text `Submit` and fail if it times out.
  // ^ After it is found trigger a click on the HTML Node that contained the text `Submit`.
```

### smart delays and retries

```jsx
// If there is no request against the `foo` alias cypress will wait for 4 seconds automatically
cy.wait('@foo')
// If there is no element with id #foo cypress will wait for 4 seconds automatically and keep retrying
cy.get('#foo')
```

### waiting for request

```jsx
cy.intercept("POST", "/graphql", (req) => {
      aliasMutation(req, "CreateLink");
    });

cy.wait("@gqlCreateLinkMutation")
```

### assering an request response

```jsx
cy.intercept("POST", "/graphql", (req) => {
      aliasMutation(req, "CreateLink");
    });

cy.wait("@gqlCreateLinkMutation")
      .its("response.body.data.createLink")
      .should("deep.include", { originalLink: graphqlResOriginalLink });

cy.wait("@gqlCreateLinkMutation")
      .its("response.body.data")
      .should("be.null");
```

### Mocking Time

```jsx
cy.wait(10000)
```

### Execution separation

```jsx
/// <reference types="cypress"/>

describe('Hello world', () => {
  it('demonstrate command - execution separation', () => {
    console.log('start');
    cy.visit('http://www.google.com');
    console.log('between');
    cy.get('.gLFyf').type('Hello world');
    console.log('end');
  });
});
// command가 실행되기 전에 start/between/end가 console에 찍힘
```

- cypress command 혹은 assertion을 실행하면 action을 실행하지 않고 function은 즉시 return한다. cypress test runner에게 action을 실행해야한다고 알려주는것
- 자동으로 retry

## cypress 환경 구축

## 관련 패키지 설치

```jsx
npm install cypress
npm install eslint-plugin-cypress 
//code coverage 확인을 위한 패키지
npm install @cypress/code-coverage babel-plugin-istanbul 
```

## 설정

- package.json
    
    ```json
    {
    	"scripts": {
    	  // headless 모드로 CI 서버에서 실행되며 터미널에서 실행되고 비디오 레코딩을 할 수 있다.
        "cypress": "cypress run",
    		//
        "cypress:open": "cypress open",
    		...
      },
    	...
    	"devDependencies": {
    		"cypress": "7.6.0",
    		"eslint-plugin-cypress": "2.11.3",
    		"@cypress/code-coverage": "3.9.10",
    		"babel-plugin-istanbul": "6.0.0",
    		...
    	},
    	...
    }
    ```
    
- .eslintrc
    
    ```json
    {
    	"env": {
    		...
        "cypress/globals": true
      },
    "plugins": [
        ...
        "cypress"
      ],
    "extends": [
        ...
        "plugin:cypress/recommended"
      ],
    }
    ```
    
- cypress.json
    
    ```tsx
    {
      "baseUrl": "http://localhost:3000",
    	// 아래 4줄은 cypress 가 최상단이 아닌 src 폴더 아래에서 돌아가도록 경로 설정 추가
      "integrationFolder": "src/cypress/integration",
      "fixturesFolder": "src/cypress/fixtures",
      "supportFile": "src/cypress/support/index.js",
      "pluginsFile": "src/cypress/plugins/index.js",
      "videoRecording": false
    }
    ```
    
- cypress.env.json (local 파일 /  gitignore)
    
    ```tsx
    {
      "baseUrl": "http://localhost:3000",
      "email": "seoyul.kim@beautydatalab.com",
      "verification-code": "beautydatalab",
    "clientSideEndPoint": "https://dev-auth.fitsme.kr/graphql",
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjEuMC4wIiwic2NvcGUiOiJhcHBsaWNhdGlvbiIsImlhdCI6MTU1NzkxMDM3NywiaXNzIjoiYmxlbmRlZCJ9.y5LEfaEcsluDtj_yJlMqU9hlyvkkzLsQ5w1X4_zfg2s"
    }
    ```
    
    - 테스트시 사용될 env를 선언해준다.
- jest.config.js
    
    ```tsx
    {
     testPathIgnorePatterns: ["<rootDir>/cypress/"],
    }
    ```
    
    - `testPathIgnorePatterns` 을 통해서 jest 실행시 cypress 폴더까지 검사해 에러 뜨는 현상 방지

## Cypress 폴더 구조

```jsx
/cypress - to hold all things Cypress i.e. tests, fixtures, page objects, utils, plugins, commands
	/fixtures - JSON files of common data objects needed in tests
	/integration - all of our tests, we would often create sub-folders per page/feature or even by group of tests
	/pages - page objects and each feature would often have its own sub-folder for the pages related to it
	/plugins - custom plugins to run in a Node server, each feature/page would have its own sub-folder for API teardown/setup
	/support - custom commands and types here
	/utils - extra utility files to be used throughout
		/action
		/graphql
	/config - environment configuration JSON files to extend/override the base cypress.json file - not all teams did this but it's another approach
```

- utils/graphql/alias.js
    
    ```jsx
    export const hasOperationName = (req, operationName) => {
      const { body } = req;
    
      return body.operationName === operationName;
    };
    
    export const aliasQuery = (req, operationName) => {
      if (hasOperationName(req, operationName)) {
        req.alias = `gql${operationName}Query`;
      }
    };
    
    export const aliasMutation = (req, operationName) => {
      if (hasOperationName(req, operationName)) {
        req.alias = `gql${operationName}Mutation`;
      }
    };
    ```
    

## Code Coverage 폴더 구조

```jsx
/coverage 
	/Icov-report 
		/app.link.hitple.com 
			/__generated__ 
			/pages 
			/src 
```

## 테스트

- 터미널에 `npm run cypress:open` 을 실행하면 작성한 테스트 파일 별로 실행할 수 있다.
- 로컬에서 테스트 하고 싶을 경우 `cypress.env.json` 파일에서 baseUrl 설정을 변경하거나 `cy.visit()` 할 경우 로컬 주소를 넣어준다.
- code coverage를 `open coverage/lcov-report/index.html` 명령어를 통해 html 파일로 확인할 수 있다. (혹은 최상단 codeCoverage 폴더 안에서 컴포넌트, 페이지 별로도 확인 가능)

### 형태

- beforeEach에 각 it 전에 동일하게 수행할 것들을 적어준다
- 테스트하고 싶은 기능별로 it으로 나누어 테스트 코드를 적어준다.

```jsx
describe("when you are in brand Page", () => {
  beforeEach(() => {
    cy.intercept("POST", "/graphql", (req) => {
      aliasMutation(req, "CreateLink");
    });

    login();
  });

  it("should success creating link", () => {
    const inputLink = "www.naver.com";
    const graphqlResOriginalLink = "http://www.naver.com";

    cy.findByPlaceholderText("줄이고자 하는 URL을 입력해주세요")
      .type(inputLink)
      .should("have.value", inputLink);
    cy.contains("생성").click();

    cy.wait("@gqlCreateLinkMutation")
      .its("response.body.data.createLink")
      .should("deep.include", { originalLink: graphqlResOriginalLink });

    cy.get(".Label__StyledLabel-k9469y-0").should("have.text", "기존 URL");
    cy.get(
      ":nth-child(2) > .LinkListItem__Container-j4stqv-0 > .LinkListItem__OriginalLink-j4stqv-6",
    ).should("have.text", graphqlResOriginalLink);
  });

  it("should fail creating link", () => {
    cy.findByPlaceholderText("줄이고자 하는 URL을 입력해주세요")
      .type("www.google")
      .should("have.value", "www.google");
    cy.get("#create-link-button").should("contain", "생성").click();

    cy.get(".Label__StyledLabel-k9469y-0").should(
      "have.text",
      "URL 형식을 다시 확인해주세요",
    );
  });
});
```

```jsx
cypress에서 권장하는 방식이지만 작업하기 번거로울 수 있음
=> cypress에서 선택시 위에서처럼 ID나 className을 선택하지 않고 
<button
  id="main"
  class="btn btn-large"
  name="submission"
  role="button"
  data-cy="submit"
>
  Submit
</button>
과 같이 attribute를 주어 선택하도록 하여 js나 css혹은 id, className 변경으로부터 자유룝게 작성하는 것이 권장된다.

cy.get('[data-cy=submit]').click()	 Always	Best. Isolated from all changes.
[https://docs.cypress.io/guides/references/best-practices]
```

### 모든 테스트 시 로그인을 반복해서 하지 않도록 한다

[https://docs.cypress.io/guides/getting-started/testing-your-app#Logging-in](https://docs.cypress.io/guides/getting-started/testing-your-app#Logging-in)

## 고려사항

- 테스트 파일명 및 분리 조건
- jest+cypress?
- Codecov를 사용하여 mr 마다 code coverage를 확인하고 전보다 낮아지지 않도록 한다.


```toc

```
