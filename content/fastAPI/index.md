---
emoji: ''
title: 'Fast API'
date: '2022-01-05 23:00:00'
author: seoyul
tags: backend
categories: BackEnd
---

# Fast API

1. Starlette 프레임워크를 기반으로 비동기 API 서버를 지원
2. Pydantic 라이브러리와의 호환으로 데이터 검증 지원
3. OpenAPI 지원을 통해 자동 스웨거 생성 가능
4. 성능적인 측면에서는 Node와 Go와 동등한 높은 성능

> [Starlette](https://github.com/encode/starlette)는 다른 파이썬 웹 프레임워크로, Sanic, Flask, Django 등과 비교하면 가볍고 강력한 ASGI(Asynchronous Server Gateway Interface) 프레임워크/툴킷이다. fastapi는 Starlette를 한번 감싸기 때문에 Stalette를 직접 사용하는 것보다는 성능이 떨어질 수밖에 없지만 개발 속도를 폭증시켜주는 등 fastapi의 장점은 이 모든 것을 상쇄시킬 수 있다. Starlette이 강력한 성능을 보장하는 이유는 내부적으로 [uvicorn](https://github.com/encode/uvicorn)을 사용하고 있기 때문이다. uvicorn은 uvloops와 httptools를 사용하는 초고속 ASGI 서버이다. uvloop의 성능 비밀은 libuv(Node.js가 동작하는 환경)과 Cython에 있다. fastapi가 Starlette 보다 성능이 떨어지는 것처럼 Starlette도 uvicorn을 직접 사용하는 것보다 더 많은 코드가 실행되기 때문에 더 느릴 수밖에 없다. 하지만 PATH 기반의 라우팅 등으로 간단한 웹 애플리케이션을 빌드하는 도구를 제공하기 때문에 생산성 측면에서 훨씬 더 도움이 된다.

> [Pydantic](https://github.com/samuelcolvin/pydantic/) 은 데이터 구문 분석 및 유효성 검사에 유용한 라이브러리이다. 입력 유형을 선언 된 유형 (유형 힌트 사용)으로 강제 변환하고 `ValidationError` 를 사용하여 모든 오류를 축적하며 쉽게 발견 할 수 있도록 [문서화](https://pydantic-docs.helpmanual.io/) 되어 있다. **Pydantic은 Parsing을 도와주는 것**일 뿐, **Validation Check을 하기 위한 라이브러리는 아니**다. a의 타입이 float으로 설정되어 있을 경우 값이 `str`으로 들어와도 이를 `float`으로 **Parsing** 해둔다. 하지만 만약 Parsing이 불가능한 데이터 형식이 들어왔을 때는 **`Validation error`를 `raise`한다**.

## WSGI vs ASGI

### WSGI

python 어플리케이션, 스크립트가 웹 서버와 통신하기 위한 인터페이스로 CGI 디자인 패턴을 코태로 만들어진것이지만 실제 CGI와는 차이가 있다.

웹서버와 어플리케이션 사이에 미들웨어 역할을 하며 기술적으로는 웹 서버도 wsgi에 대한 작동 코드가 필요하고 어플리케이션 또한 wsgi에 대한 작동 코드가 필요한 client-server model을 응용한 것이며, 웹 서버가 어플리케이션의 코드르 직접적으로 읽을 수 없으므로 중간의 미들웨어가 해당 코드를 읽어서 결과를 대신 반환해주는 역할을 한다.

### ASGI

python에서는 asyncio, coroutine 과 같은 비동기 처리를 지원한다. 그러나 wsgi는 동기 함수 처리만을 지원하여 여러 작업을 동시에 처리하는 것에 한계가 있기 때문에 웹 서비스의 대용량 트래픽 처리를 유연하게 처리하기 어렵다.

웹 서비스에서 웹 소켓 등을 사용한 실시간 채팅 서비스 등을 할 수 있는데 wsgi로는 이러한 서비스를 구현하는데 어려움이 있다.(비동기 큐와 같은 celery를 잘만 활용하면 대용량 트래픽 처리를 요구하는 서비스 구현이 불가능 한것은 아니지만 트레이싱 등과 같은 유지보수, 기본적인 구현이 쉽지 않다.)

따라서 최근에는 django 3.0 뿐만 아니라 FastAPI 등 프레임워크에서도 asgi 인터페이스를 적용하였다.

운영 아키텍처로 봤을 떄는 크게 다르지 않지만 wsgi와 다르게 asgi는 기본적으로 요청을 비동기 처리하여 wsgi에서는 지원되지 않는 websocket 프로토콜과 http 2.0을 지원한다.

이러한 대표적인 asgi web app에는 uvicorn 있으며 asgi 기반의 웹 어플리케이션 서버로서 그 내장 모듈로 uvloop을 사용한다. uvloop에서 uv는 libuv 즉 javascript v8에서 사용되는 비동기 모둘을 사용한다.

asgi는 Cython 기반으로 C++ 언어로 작성되어 매우 빠른 속도를 제공하는 것이 특징인데다 libuv를 사용하여 비동기 처리를 하기 때문에 node.js와 같은 비동기 처리 속도를 누릴 수 있다.

## 특징

- https://github.com/tiangolo/fastapi
- fast to code: 약 200%에서 300%까지 기능 개발 속도 증가.
- fewer bugs : 사람(개발자)에 의한 에러 약 40% 감소
- intuitive : 훌륭한 편집기 지원. 모든 곳에서 자동완성. 적은 디버깅 시간.
- easy: 쉽게 사용하고 배우도록 설계. 적은 문서 읽기 시간.
  - [https://fastapi.tiangolo.com/ko/](https://fastapi.tiangolo.com/ko/)
- short: 코드 중복 최소화. 각 매개변수 선언의 여러 기능. 적은 버그.

  - 파라미터 타입을 명시할 수있다.(파이썬 type hints)
  - 이로 인해 Type Check를 할 수 있으며, data의 validation을 자동으로 해주고 오류 시 error를 자동으로 생성한다.

  ```python
  from typing import List, Dict
  from datetime import date

  from pydantic import BaseModel

  # Declare a variable as a str
  # and get editor support inside the function
  def main(user_id: str):
      return user_id

  # A Pydantic model
  class User(BaseModel):
      id: int
      name: str
      joined: date
  ```

- robust: 문서 자동화 및 쉬운 배포
  - FastAPI에서는 별도의 스웨거 파일 정의 없이 자동으로 생성하여 준다. 요청과 응답 부분에 pydantic 모델을 사용하며 배포시 자동으로 json 형태 변환되어 스웨거의 요청과 응답 모델에 자동으로 매핑 된다.
  - 서버를 실행시키고, `/docs` 로 url를 접속하면 아래와 같은 Swagger UI가 만들어진다.
  - /redoc 으로 접속하면 생성 가능하다.
  - 복잡한 시나리오를 품고 있는 API가 아니라면 스웨거 환경에서 충분히 테스트할 수 있다.(Try it out을 누르고 Excuse로 실행)
- standards-based: API에 대한 (완전히 호환되는) 개방형 표준 기반: OpenAPI (이전에 Swagger로 알려졌던) 및 JSON 스키마.

## 설치

```jsx
pip install fastapi
pip install uvicon[standard]

//서버 실행 localhost:8000에서 확인 가능
uvicorn main:app --reload

main: the file main.py (the Python "module").
app: the object created inside of main.py with the line app = FastAPI().
--reload: make the server restart after code changes. Only do this for development.

뒤쪽에 붙은 --reload 옵션은 hot reloading 기능으로 소스 코드가 수정되면 서버를 바로 재시작하는
기능으로 개발 환경에서 활용하면 좋다.
```

## 기본 코드

```python
from typing import Optional
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
  return {"hello":"world"}

@app.get("/items/{item_id}")
def read_item(item_id:int, q:Optional[str]=None):
  return{"item_id":item_id, "q":q}

Receives HTTP requests in the paths / and /items/{item_id}.
Both paths take GET operations (also known as HTTP methods).
The path /items/{item_id} has a path parameter item_id that should be an int.
The path /items/{item_id} has an optional str query parameter q.
```

- 비동기처리가 필요하면 async와 await를 사용할 수 있다. 비동기로 호출이 필요한 함수를 def가 아닌 async def로 처리해주고 함수 호출하는 곳에 await를 걸어주어 데이터베이스 조회나 I/O를 처리해야 하는 곳에 사용해주면 된다.

```python
@app.get("/")
async def read_root():
  return {"hello":"world"}
```

- FastAPI는 uvicorn에서 제공되는 비동기 이벤트 루프를 사용한다. 또한 run_in_executor 안에서 threadpool을 사용해서 동기 함수를 처리한다. 즉, 이미 threadpool을 갖고 있다. 따라서 gunicorn을 사용하는 경우 퍼포먼스를 끌어올리고자 thread를 활성화 시켜서는 안된다. [오히려 성능이 저하되고 최악의 경우에는 컨텍스트에 안전하지 않은 코드가 있는 경우 문제가 발생할 수 있다고 한다](https://github.com/tiangolo/fastapi/issues/551#issuecomment-584308118).

```jsx
@app.put("/items/{item_id}")
def update_item(item_id:int, item:Item):
  return { "item_name" : item.name, "item_id" : item_id }

//item 타입이 지정되어 있기 때문에 editor에서 auto-complete이 가능하다.
/docs로 접속 후 try it out 을 통해 parameter를 넣어주어 테스트한다.

Validate that there is an item_id in the path for GET and PUT requests.
Validate that the item_id is of type int for GET and PUT requests.
If it is not, the client will see a useful, clear error.
Check if there is an optional query parameter named q (as in http://127.0.0.1:8000/items/foo?q=somequery) for GET requests.
As the q parameter is declared with = None, it is optional.
Without the None it would be required (as is the body in the case with PUT).
For PUT requests to /items/{item_id}, Read the body as JSON:
Check that it has a required attribute name that should be a str.
Check that it has a required attribute price that has to be a float.
Check that it has an optional attribute is_offer, that should be a bool, if present.
All this would also work for deeply nested JSON objects.
Convert from and to JSON automatically.
Document everything with OpenAPI, that can be used by:
Interactive documentation systems.
Automatic client code generation systems, for many languages.
Provide 2 interactive documentation web interfaces directly.
```

- 이러한 작성 방식으로 인해 다음과 같은 장점을 얻을 수 있다.
  - Editor support, including:
    - Completion.
    - Type checks.
  - Validation of data:
    - Automatic and clear errors when the data is invalid.
    - Validation even for deeply nested JSON objects.
  - Conversion of input data: coming from the network to Python data and types. Reading from:
    - JSON.
    - Path parameters.
    - Query parameters.
    - Cookies.
    - Headers.
    - Forms.
    - Files.
  - Conversion of output data: converting from Python data and types to network data (as JSON):
    - Convert Python types (`str`, `int`, `float`, `bool`, `list`, etc).
    - `datetime` objects.
    - `UUID` objects.
    - Database models.
    - ...and many more.
  - Automatic interactive API documentation, including 2 alternative user interfaces:
    - Swagger UI.
    - ReDoc.

```toc

```
