---
layout: post
title: 왜 Immutable해야 하는가
category: Spring
tags: [Spring, 스터디, Immutable, 디자인패턴]
permalink: /study/:year/:month/:day/:title/
comments: true
---

---

> Programmers 웹백엔드 스터디 7기 진행시 했던 고민과 공부한 내용들을 정리한 글입니다.

---

미션 기능 구현 중 setter 메소드를 사용하지 말고 Immutable(불변성)으로 해야 하는다는 말을 들었다. 왜 그래야 하는걸까? 한번 알아보자.

불변이란 **생성 시점 이후로 상태가 변하지 않음을 의미한다.**

1. **mutable 객체는 수정자 호출 시 객체 상태가 어떻게 바뀔지 확신할 수 없다.** 안정적으로 사용 어렵다.
   => 하지만, immutable 객체는 상태값이 변하지 않으므로 안정적이다.
2. 스레드에 안전하다. 동기화가 필요없다.
3. 언제나 재사용이 가능하고 공유할 수 있어 메모리 요구량과 가비지 컬렉터 수집 비용이 줄어든다.

대신 값마다 별도의 객체를 만들어야 하는 단점이 있다.

**불변 만드는 규칙**

- setter메소드 같이 상태 변경 메소드를 제공하지 않는다.
- 계승 금지
- 모든 필드를 private & final로 선언해 직접 수정 막는다
- 생성자 private

변경 가능한 클래스로 만들 타당한 이유가 없다면, 반드시 변경 불가능한 클래스로 만들자.

설계, 구현, 사용이 쉽고 오류 가능성도 적고 더 안전하다.

## references

Effective Java 2/E<br>
https://www.yegor256.com/2014/11/07/how-immutability-helps.html<br>
https://www.ibm.com/developerworks/java/library/j-jtp02183/index.html<br>
https://reflectoring.io/java-immutables/#value-objects
