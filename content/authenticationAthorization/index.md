---
emoji: ''
title: 'Authentication & Authorization'
date: '2021-09-15 22:00:00'
author: seoyul
tags: backend
categories: BackEnd
---

# Authorization & Authentication

## Authentication(인증)

- 사용자가 누구인지 확인하는 절차로 회원가입, 로그인 과정이 인증의 대표적인 예시이다.
  1. 아이디 및 비밀번호를 생성한다.
  2. 비밀번화를 암호화하여 db 에 저장한다.
  3. 등록된 아이디와 비밀번호를 입력한다.
  4. 암호화되어 db에 저장된 사용자의 비밀번호가 서로 일치하는지 비교한다.
  5. 일치하면 로그인, 일지하지 않을 경우 에러를 보낸다
  6. 로그인에 성공하면 access token을 클라이언트에 전송한다.
  7. 최초 로그인 성공 후 access token을 첨부하여 서버에 요청을 전송함으로써 매번 로그인하는 과정을 생략할 수 있다.

## Authorization(인가)

- 사용자가 요청을 실행할 수 있는 권한 여부를 확인하는 절차
- HTTP의 특징인 request/response, stateless한 성질때문에 서버는 사용자가 로그인 했을 경우 header에 메타데이터를 보내서 확인하는데, 이 메타정보를 JWT라고 한다.
  1.  인증 절차를 통해 사용자의 정보(email, id)를 담고 있는 access token을 생성한다.
  2.  사용자가 요청을 보낼 때 access token을 첨부하여 보낸다
  3.  서버는 해당 access token을 복호화하고 정보를 얻는다.
  4.  얻은 정보를 사용하여 사용자 권한을 확인한다.
  5.  사용자의 권한이 확인되면 해당 요청을 처리하고 권한이 없다면 에러를 보낸다.

## http 요청 인증 방식

### cookie

- 클라이언트에 저장되는 key-value 형태의 텍스트 형식의 데이터
- **Response header에 Set-Cookie 속성을 사용**하면 **쿠키를 만들 수 있으며** 사용자가 따로 처리 안해도 **HTTP가 Request시 header에 자동으로 넣는다.**
- 사용자 인증이 유효한 시간을 명시할 수 있으며 한번 유효시간이 정해지면 브라우저를 끄더라도 인증이 유지된다.
- 구성요소
  **1) 이름 : 각각의 쿠키를 구별하는데 사용되는 이름**
  **2) 값 : 쿠키의 이름과 관련된 값**
  **3) 유효시간 : 쿠키의 유지시간**
  **4) 도메인 : 쿠키를 전송할 도메인**
  **5) 경로 : 쿠키를 전송할 요청 경로**

### session

- 세션은 쿠키를 기반으로 하며 사용자 정보 파일을 브라우저에 저장하는 쿠키와 달리 서버측에서 관리한다.
- 세션 ID를 부여하며 웹 브라우저가 서버에 접속해서 브라우저를 종료할 때까지 인증상태를 유지한다.
- 사용자에 대한 정보를 서버에 저장하기 때문에 쿠키보다 보안에 좋지만 사용자가 많아질수록 서버 메모리를 많이 자지하게 되기 때문에 동시접속자가 많은 웹사이트의 경우 서버에 과부하를 주게되므로 성능 저하의 요인이 될 수 있다.
  **1. 클라이언트 HTTP Request를 서버에게 보냄**
  **2.** **서버에서 유효성(회원인지)확인 후 / 고유한 Session-ID를 발급하여 쿠키에 넣음** **& HTTP Response header에 넣어 응답**
  **3. 클라이언트는 HTTP Response header에서 이 쿠키를 추출하여 저장**
  **4. 클라이언트가 HTTP Request를 보낼 때, HTTP가 해당 쿠키를 찾아 header에 자동으로 넣어서 전송**
  **5. 서버는 HTTP Request header에서 쿠키안에 Session-ID를 확인하고 통신**

### cookie vs session

- **저장위치**
  - **쿠키 : 클라이언트(브라우저 or 디바이스)**
  - **세션 : 서버**
- **보안**
  - **쿠키 : 클라이언트에 저장되므로 보안에 매우 취약**
  - **세션 : 쿠키 내부에 Session-ID만 있기에, 직접적인 정보노출은 없다** **그리고 서버에서 관리하므로 쿠키보다 비교적 보안성이 좋다**
- **LifeCycle**
  - **쿠키 : 만료시간 / 만료시간이 없다면 브라우저 종료해도 남아있음**
  - **세션 : 만료시간 / 브라우저 종료시 무조건 삭제**
- **속도**
  - **쿠키 : 클라이언트에 저장되어 있어서 서버 요청시 빠름**
  - **세션 : 실제 저장된 정보가 있으므로 서버의 처리가 필요해 쿠키보다 느림**

### JWT(Json Web Token)

- Json 데이터 구조로 표현한 토큰
- **토큰은 서버의 상태를 저장하지 않음 (Stateless)** -> 토큰 자체로 정보를 가지고 있어 별도의 인증 서버 필요하지 않다.

- JWT는 유저 정보를 담은 JSON 데이터를 암호화해서 클라이언트와 서버간에 주고 받는다

  1. 사용자가 로그인을 한다.

  2. 서버에서는 계정정보를 읽어 사용자를 확인 후, 사용자의 고유한 ID값을 부여한 후, 기타 정보와 함께 Payload에 넣는다.

  3. JWT 토큰의 유효기간을 설정한다.

  4. 암호화할 SECRET KEY를 이용해 ACCESS TOKEN을 발급한다.

  5. 사용자는 Access Token을 받아 저장한 후, 인증이 필요한 요청마다 토큰을 헤더에 실어 보낸다.

  6. 서버에서는 해당 토큰의 Verify Signature를 SECRET KEY로 복호화한 후, 조작 여부, 유효기간을 확인한다.

  7. 검증이 완료된다면, Payload를 디코딩하여 사용자의 ID에 맞는 데이터를 가져온다.

- 장점
  - 확장성이 뛰어나며 토큰 기반으로 하는 다른 인증 시스템에 접근이 가능하다. 예를 들어 Facebook 로그인, Google 로그인 등은 모두 토큰을 기반으로 인증을 하며 선택적으로 이름이나 이메일 등을 받을 수 있는 권한도 받을 수 있다.
- 단점
  - 이미 발급된 JWT에 대해서는 돌이킬 수 없다. 세션/쿠키의 경우 만일 쿠키가 악의적으로 이용된다면, 해당하는 세션을 지워버리면 되지만 JWT는 한 번 발급되면 유효기간이 완료될 때 까지는 계속 사용이 가능하기 때문에 악의적인 사용자는 유효기간이 지나기 전까지 정보들을 털어갈 수 있다.
    → 해결책으로 기존의 Access Token의 유효기간을 짧게 하고 Refresh Token이라는 새로운 토큰을 발급한다. 그렇게 되면 Access Token을 탈취당해도 상대적으로 피해를 줄일 수 있다.

### Refresh Token

- **Access Token이 만료** 되었을 때, **사용자가 다시 인증 과정을 거치지 않고 Access Token을 다시 받게 해주는 Token**
- **Access Token보다는 만료 기간이 길다**
- **Refresh Token이 만료되면 다시 인증과정을 거처 발급** 받아야 한다

```toc

```