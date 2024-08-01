---
emoji: ''
title: 'websocket, socket.io, webRTC 그리고 datachannel'
date: '2024-08-01 16:00:00'
author: seoyul
tags: web
categories: web
---

- nomad coder의 WebRTC 강의를 듣고 정리한 내용입니다.

## websocket vs http

- 둘다 프로토콜이다.
    - 프로토콜이란 둘이 만났을 때 어떻게 일들이 진행될지 결정하고 어떻게 돌아가야할지에 대한 규칙을 만든다.
    - 프로그래머는 이 규칙을 갖고 그를 따르는 코드를 만들어 실행한다.
    - 표준이 되는 규칙, protocol 이 먼저 만들어져야하고, 개발자들이 이 규칙을 코드에 녹여내야한다.

### http

- stateless
- 브라우저가 요청을 해야 서버가 응답을 할 수 있다.

### websocket

- real-time 기능을 만들 수 있다
- wss (secure websocket) 프로토콜을 사용한다.(wss://)
- 브라우저가 서버로 websocket request 를 보내면 서버가 받거나 거절한다.
    - 악수가 한번 성립되면 연결이 establish 된다.
- 연결이 되어있기 때문에 서버는 유저를 기억할 수 있다.
    - 연결이 되어있기 때문에 요청을 기다리지 않고도 메세지를 보낼 수 있다.
    - bi-directional
- 브라우저에 내장된 websocket api 가 있다.
- 브라우저와 백엔드 사이에서만 작동하지 않고 2개의 백엔드 사이에서도 작동한다.
- ws -nodejs websocket library
    - 프로토콜로 어떤 규칙을 따르는 코드
    - websocket protocol을 실행하는 패키지
    - implementation, core of websocket

## socket.io vs websocket

- 실시간, 양방향, Event 기반의 통신을 가능하게 한다.
- websocket을 실행하는게 아닌 framework인데, 웹소켓보다 탄력적이다.
- 웹소켓은 socket.io가 실시간, 양방향, 이벤트 기반 통신을 제공하는 방법중 하나이다.
    - socket.io는 웹소켓에 문제가 생겨도 다른 방법으로 작동
    - http long polling을 사용한다.

### Adapter

- 다른 서버들 사이에 실시간 어플리케이션을 동기화하는것
- In memory adapter를 사용하면 여러개의 서버를 만들었을 때 같은 connection이라도 같은 memory pool을 공유하지 않아 서버가 분리되어있기 때문에 다른 서버의 클라이언트에는 메세지를 보낼 수 없다.
    - 이를 개선하기 위해서 mongodb adapter.. 등을 이용할 수 있다.
    - adapter는 어플리케이션으로 통하는 창문으로 누가 연결되었는지, 현재 어플리케이션에 room이 얼마나 있는지 알려준다.

## admin ui

- socket.io admin pannel

## stream

- track을 갖을 수 있다, 비디오가 하나의 Track 이됨

## WebRTC

- web realtime communication
- peer to peer
    - 영상, 오디오, 텍스트가 서버로 가지 않고 직접 상대방에가 감, 브라우저끼리 연동
    - 서버가 필요하지 않아서 빠름
    - websocket은 서버에 전달하면 서버가 다시 전달하는 방식
    - 시그널링을 위한 서버가 존재하는 이유는
        - 연결하려는 브라우저가 어디에 있는지 알기 위해서
        - 한 브라우저가 서버에 configuration을 전달하면 다른 브라우저도 연결을 원할때 같은 동작을 하게된다.
        - 그러면 서버가 특정 브라우저가 어디에 있는지 알려주고 peer-to-peer connection이 이뤄진다.
        - 서버는 영상, 텍스트를 전송하는것이 아닌 브라우저의 위치, setting, 방화벽, 라우터 등을 전달하는 역할만 한다.

### 단점

- peer 수가 늘어나면 느려지기 시작한다.
    - 모든 피어들이 서로 연결되어있어 피어들이 많이 늘어나면 같은 스트림을 여러번 업로드하게 되어안좋을 수 있음
    - 업로드와 다운로드가많아지며 wifi가 느려짐
    - Mesh architecture라 느려짐
        - 3명정도 이상은 안좋음
        - 사람이 많아질수록 더 무거워짐
        - 비디오를 업로드 하는 동시에 모든 사람의 비디오를 다운받아야하기 때문
            - 사람이 다섯이면 내것을 네번이나 업로드해야함

### SFU

- selective forwarding ui
    - 사람이 넷이라면 세개의 스트림을 다운로드 해야하지만 내것은 한번만 업로드하면 된다.
- 서버가 모두로부터 스트림을 받는데 이 서버는 그 스트림들을 압축한다.
    - 다운로드할 때 질이 조금 안좋은 스트림을 다운로드 하게 된다.
- 누가 이방의 호스트인지 누가 말하고, 스크린을 공유하고 , 발표하고 있는지 알고있다
    - 누가 말하느냐에 따라 질이 좋고 안좋고를 판단해서 다운로드 하게된다.

** 데이터 채널을 이용하면 그냥 텍스트라 업로드와 다운로드가 빨라 위 구조를 고민하지 않아도된다.

** 서버가 필요하다면 socket.io도 괜찮고, peer to peer 혹은 메세지를 보내는데 서버는 필요없다면 data channel을 사용할수도 있다.

## local tunnel

- 서버를 전세계와 공유하게 해준다.

```jsx
> lt --port 3000
```

## stun server

- 휴대폰과 노트북이 같은 wifi 가 아닐때 접속이 불가능 한것은 stun server가 필요하기 때문이다.
    - 서버가 공용 Ip주소를 찾게해준다.
        - 장치에 공용 주소를 알려주는 서버
        - 그래야 다른 네트워크에 있는 장치들이 서로 찾을 수 있게된다.
- stun server는 어떤것을 Request 하면 인터넷에서 누군지 알려주는 서버이다. 공용 ip를 알려줌

## data channel

- peer to peer user 가 모든 종류의 데이터를 주고 받을 수 있는 채널
- 플레이어의 지도상 위치.. 이미지, 파일, 텍스트, 게입 업데이트 패킷 같은 것들을 주고받을 수 있다.
- 원하면 무엇이든 주고 받을 수 있고, socket.io, websocket 없이도 채팅을 만들수 있음
- 무언가를 offer하는 socket이 data channel을 생성하는 주체가 되어야한다.
    - 그리고 Offer를 만들기 전에 data channel을 만들어야한다.