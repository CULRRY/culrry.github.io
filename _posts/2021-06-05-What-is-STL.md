---
title: "[STL] STL이란?"
date: "2021-06-05 20:44:00 +0900"
last_modified_at: "2021-06-05 20:44:00 +0900"
categories:
    - STL

tags:
    - [C++, STL]

toc: true
excerpt: "STL이란 표준 템플릿 라이브러리(Standard Template Library)의 약자로써, C++에서 프로그래밍에 필요한 자료구조와 알고리즘을 Template의 형태로 제공하는 C++ 라이브러리이다."
description: "STL이란 표준 템플릿 라이브러리(Standard Template Library)의 약자로써, C++에서 프로그래밍에 필요한 자료구조와 알고리즘을 Template의 형태로 제공하는 C++ 라이브러리이다."
---

## STL(Standard Template Library) 이란?

STL이란 표준 템플릿 라이브러리(Standard Template Library)의 약자로써, C++에서 프로그래밍에 필요한 자료구조와 알고리즘을 Template의 형태로 제공하는 C++ 라이브러리이다.

## STL의 장단점

장점

-   일반화를 지원한다.
-   컴파일 타임 매커니즘을 이용하므로 실행 시 효율 저하가 적다.
-   사용자의 알고리즘을 적용시키는 등의 확장성이 우수하다.
-   소스코드의 길이가 대폭 감소된다.

단점

-   템플릿 기반이므로 함수, 클래스가 매번 구체화되어 코드가 비대해진다.
-   디버깅이 어렵다.

## STL의 구성

### 1\. 컨테이너(Container)

자료를 저장하는 객체, 자료구조를 모아둔 집합이다.

-   **순차 컨테이너 (Sequence Container)**
    -   자료를 순차적으로 저장하는 컨테이너이다.
    -   자료를 입력한 순서대로 저장한다.
    -   vector, list, deque 등이 존재한다.

-   **연관 컨테이너 (Associative Container)**
    -   트리 자료구조로 구성되어 있다.
    -   검색, 삽입, 삭제 속도가 빠르다.
    -   set, multiset, map, multimap이 존재한다.

-   **컨테이너 어댑터 (Container Adaptor)**
    -   기존의 컨테이너의 일부의 기능만 사용가능하게 하여 기능이 제한되거나 변형된 컨테이너이다.
    -   stack, queue, priority\_queue가 존재한다.

### 2\. 반복자(Iterator)

-   현재 처리하고 있는 자료의 위치를 기억하는 객체
-   포인터와 유사하다.
-   \*연산자, ++연산자를 사용 가능 하다.

### 3\. 알고리즘(Algorithm)

STL 기반의 정렬, 삭제, 검색, 연산 알고리즘을 구현해놓은 함수 템플릿이다.

### 4\. 함수자(Fuctor)

함수처럼 동작하는 객체이다. 컨테이너와 알고리즘에서 정렬 기준을 정의해준다던지와 같은 조건을 설정해주는 용도로 사용된다.

### 5\. 할당기(Allocator)

 컨테이너의 메모리 할당 정책을 캡슐화한 객체이다. 모든 컨테이너는 자신의 할당기를 가지고 있다.
