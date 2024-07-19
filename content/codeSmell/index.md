---
emoji: ''
title: 'code smell in css'
date: '2023-11-03 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

# code smells in css

### css 규칙

- 스타일은 추가만하고 취소는 하지 않도록 한다.
- 매직넘버(반드시 그렇게만 작동하는 값) 피하기
- 조건부 선택자를 피한다.
    - 재사용 불가능하고
    - 특정도를 높이며
    - 선택자를 한정함으로 브라우저 작업량을 증가시켜 성능을 저하시킨다.
    
    ```powershell
    //x
    ul.nav {}
    a.button {}
    div.header {}
    
    //o
    .nav {}
    .button {}
    .header {}
    ```
    
- 꼭 필요한 경우를 제외하고 절대값을 피하고 상대값을 사용한다.
    
    ```powershell
    //x
    h1 {
        font-size: 24px;
        line-height: 32px;
    }
    
    /**
     * Main site `h1`
     */
    .site-title {
        font-size: 36px;
        line-height: 48px;
    }
    
    //o
    h1 {
        font-size: 24px;
        line-height: 1.333;
    }
    
    /**
     * Main site `h1`
     */
    .site-title {
        font-size: 36px;
    }
    ```
    
- 무차별 강제 스타일을 피한다.
    
    ```powershell
    //x
    .foo {
        margin-left: -3px;
        position: relative;
        z-index: 99999;
        height: 59px;
        float: left;
    }
    ```
    
- 위험한 선택자(너무 광범위하게 적용될 수 있는 것)를 피한다.
    
    ```powershell
    div {
       background-color: #ffc;
       padding: 1em;
    }
    
    - 선택자를 의도가 명확한 선택자(selector intent)로 만든다.
    
    ul {
        font-weight: bold;
    }
    header .media {
        float: left;
    }
    ```
    
- !Important 피하기
    - 사용시 명확한 의도를 갖고 사용하도록 한다.
- ID 보다 class를 선호하라
    - class는 한 페이지에 한번만 존재해도 되고 여러번 존재해도 되며 재사용이 가능해지는 경우가 많다.
- 느슨한 클래스 이름(의도한 목적을 충분히 드러내지 못하는)을 피한다.
    - 

### Reference

[CSS 코드 냄새](https://mytory.net/archives/8982)