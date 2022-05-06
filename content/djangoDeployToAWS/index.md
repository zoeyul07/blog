---
emoji: ''
title: 'Django: deploy to AWS'
date: '2021-02-02 23:00:00'
author: seoyul
tags: backend
categories: BackEnd
---


## Deploy to AWS
- Django는 코드를 자체적으로 테스트 하기 위한 간단한 웹서버를 가지고 있지만 배포시에는 보안이 더 좋고 강력한 웹서버가 필요하기 때문에 nginx와 uWSGI를 연동하여 사용해보자.

### Nginx
- 가벼우면서도 강력한 프로그램을 목표로 만들어졌으며, Apache보다 동작이 단순하고 전달 역할만 하기 때문에 동시 접속 처리에 특화되어있다.

- 정적 파일을 처리하는 HTTP 서버로서의 역할, 응용 프로그램 서버에 요청을 보내는 reverse proxy로서의 역할을 한다. (캐싱, 리다이렉션 등 가능하다.)

> reverse proxy❔
-  클라이언트가 데이터를 요청하면 reverse proxy는 요청을 받아서 내부 서버에서 데이터를 받은 후 이 데이터를 클라이언트에 전달하게 된다.
![](https://images.velog.io/images/zoeyul/post/5bf97d31-0efb-4736-8a41-2f048d128f2e/Screen%20Shot%202021-02-03%20at%2012.52.06%20AM.png)

- 웹 응용프로그램에 reverse proxy를 두는 이유는 요청에 대한 버퍼링 떄문인데, WAS(Web Application Server)는 클라이언트의 하나의 요청에 대해 하나의 프로세스 또는 하나의 쓰레드를 할당해서 처리하는 방식을 취하고 있지만 proxy 서버를 둠으로 요청을 배분하는 역할을 한다.(nginx.conf 파일에서 location 지시어를 사용하여 요청을 배분할 수 있다.)
![](https://images.velog.io/images/zoeyul/post/c9b68574-2cbb-4f55-bd37-cdb364dd454d/Screen%20Shot%202021-02-03%20at%2012.35.58%20AM.png)

### WSGI(uWSGI) 
- WSGI는 Web Server Gateway Inerface의 약자로서, 웹 서버(nginx)와 파이썬 웹 애플리케이션 사이의 통신을 위한 프로토콜이며, 웹서버와 웹어플리케이션 간의 소통을 정의해 어플리케이션과 서버가 독립적으로 운영될 수 있게 돕는다.

- WSGI는 서버와 앱 양단으로 나뉘어져 있다. WSGI 리퀘스트를 처리하려면 서버에서 환경정보와 콜백함수를 앱에 제공해야 하며, 앱은 그 요청을 처리하고 콜백함수를 통해 서버에 응답한다.

-  Flask, django와 같은 프레임워크는 이미 WSGI 표준을 따르고 있기 때문에 바로 웹서버에 연결해 사용할 수 있지만 nginx 웹 서버는 WSGI 스펙을 구현해두지 않았기 때문에 django나 Flask 앱과 직접 통신할 수가 없기 때문에 uWSGI 서버를 중간에 둔다. 

- uWSGI, gunicorn 등이 middleware 역할을 하며, 어플리케이션의 관점에서는 이 미들웨어를 통해 앱이 실행되므로 앱을 실행시켜주는 어플리케이션 컨테이너(Application Container)라고도 할 수 있다.

#### uWSGI
- uWSGI는 Hosting server에서 full stack 개발이 가능하도록 하기 위해 개발되었다. 다양한 언어와 프로토콜, 프로세스 매니저, 프록시 등을 다양하게 다룰 수 있도록 개발되었다.

- uWSGI는 WSGI 스펙을 구현한 서버인 동시에 프로토콜이며, nginx와도 통신할 수 있다. 

- 정해진 시간에 명령을 실행할 수 있고(cron), 접속자가 없을 때 프로세스를 종료하는 cheap mode를 이용할 수 있다.( cheap 옵션은 첫번째 요청이 들어올 때까지 worker 프로세스를 실행하지 않도록 해준다. (master 프로세스는 항상 실행되어 있다) )

> ✔️ 확장성이 뛰어나며 강력하고 다양한 언어 위에서 작동하지만 너무 무거울 수 있다.

#### Gunicorn
- UNIX에서 사용하기 위해 만들어졌으며 상대적으로 빠르고 가볍고 사용이 용이하다.

- Gunicorn팀은 HTTP proxy server로 Nginx를 쓰길 권장한다. (Gunicorn은 localhost 8000번을 사용하는데 Nginx는 전형적으로 reverse proxy server를 이용한다.)

> ✔️ 속도가 빠르며 가벼움 그리고 django에서 자동으로 돌아간다.

![](https://images.velog.io/images/zoeyul/post/63613b95-0186-47d6-b769-beb5fdba1c6a/Screen%20Shot%202021-02-03%20at%201.23.29%20AM.png)

1. http 리퀘스트가 들어오면
2. 웹 서버가 그 리퀘스트를 받고
	- 서버사이드 처리가 필요 없으면 리스폰스를 리턴(static한 웹 서버)
3. 서버사이드 처리가 필요하면 wsgi 미들웨어를 통해 파이썬 어플리케이션으로 리퀘스트 전달
4. 파이썬 어플리케이션이 리퀘스트를 받아 처리 후 wsgi 미들웨어 - 웹서버를 통해 리스폰스를 리턴한다.

### Web server vs WAS(Web Application Server)
#### web server
- 웹 브라우저 클라이언트로부터 HTTP 요청을 받아 정적인 컨텐츠를 제공한다.

- HTTP 프로토콜을 기반으로 클라이언트의 요청을 서비스 한다.
	
    - 정적인 컨텐츠를 제공한다면 WAS를 거치지 않고 자원을 제공한다.
    - 동적인 컨텐츠를 제공한다면 클라이언트의 요청(Request)을 WAS에 보내고, WAS가 처리한 결과를 클라이언트에게 전달(응답, Response)한다.

#### WAS
- DB 조회나 다양한 로직 처리를 요구하는 동적인 컨텐츠를 제공하기 위해 만들어진 Application server

- web server의 기능들을 구조적으로 분리하여 처리하고자 하는 목적으로 제시되었으며, 분산 트랜잭션, 보안, 메시징, 쓰레드 처리 등의 기능을 처리하는 분산 환경에서 사용된다.

- 주요 기능으로는 프로그램 실행 환경과 DB접속 기능, 여러개의 트랜잭션 관리 기능, 비즈니스 로직 수행 등이 있다.

#### web server가 필요한 이유
- 이미지 파일과 같은 정적인 파일들은 클라이언트가 웹 문서를 먼저 받고 그에 맞게 필요한 이미지 파일들을 서버로 요청하면 보내게 되는데 web server를 통해 정적인 파일들을 application server까지 가지 않고 앞단에서 빠르게 내보내주면 web server에서는 정적 컨텐츠만 처리하도록 기능을 분배하여 서버의 부담을 줄일 수 있다.
#### WAS가 필요한 이유
- 웹 페이지는 정적 컨텐츠와 동적 컨텐츠가 모두 존재하는데, web server 만을 이용한다면 사용자가 원하는 요청에 대한 결과값을 모두 미리 만들어 놓고 서비스 해야하기 때문에 WAS를 통해 요청에 맞는 데이터를 DB에서 가져와서 비즈니스 로직에 맞게 결과를 만들어 제공함으로써 자원을 효율적으로 사용할 수 있다.

#### web server와 WAS를 분리하여 사용하는 이유
- 기능을 분리하여 서버 부하 방지
	
  - WAS는 기본적으로 동적 컨텐츠를 제공하기 위해 존재하는 서버이다. 만약 정적 컨텐츠 요청까지 WAS가 처리한다면 정적 데이터 처리로 인해 부하가 커지게 되고, 동적 컨텐츠의 처리가 지연됨에 따라 수행 속도가 느려지게 되며 이로 인해 페이지 노출 시간이 늘어나게 된다.
  
- 물리적으로 분리하여 보안 강화
	
  - SSL에 대한 암복호화 처리에 Web Server를 사용

- 여러 대의 WAS를 연결 가능

  - Load Balancing을 위해서 Web Server를 사용
  - fail over(장애 극복), fail back 처리에 유리
  - 대용량 웹 어플리케이션의 경우(여러 개의 서버 사용) Web Server와 WAS를 분리하여 무중단 운영을 위한 장애 극복에 쉽게 대응할 수 있다.( 앞 단의 Web Server에서 오류가 발생한 WAS를 이용하지 못하도록 한 후 WAS를 재시작함으로써 사용자는 오류를 느끼지 못하고 이용할 수 있다.)

- 여러 웹 어플리케이션 서비스 가능

- 접근 허용 IP 관리, 2대 이상의 서버에서의 세션 관리 등도 Web Server에서 처리하면 효율적이다.

즉, 자원 이용의 효율성 및 장애 극복, 배포 및 유지보수의 편의성 을 위해 Web Server와 WAS를 분리한다.

***
[Nginx 이해하기 및 기본 환경 설정 셋팅하기](https://whatisthenext.tistory.com/123)
[NGINX + HTTPS 적용하기](https://www.youtube.com/watch?v=6TYwnURF09w)
[WSGI에 대한 설명, WSGI란 무엇인가?](https://paphopu.tistory.com/entry/WSGI에-대한-설명-WSGI란-무엇인가)
[python개발자 uwsgi를 버리고 gunicorn(지-유니콘)으로 갈아타다.](https://medium.com/@elastic7327/python개발자-uwsgi를-버리고-gunicorn으로-갈아타다-df1c95f220c5)
[uWSGI의 고급 기능들](https://blog.sapzil.org/2015/10/24/advanced-uwsgi/)



```toc

```
