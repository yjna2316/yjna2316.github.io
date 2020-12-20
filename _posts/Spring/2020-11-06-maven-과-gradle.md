---
layout: post
title: Maven vs Gradle
category: Spring
tags: [Spring, 스터디, Maven, Gradle, 빌드툴, 빌드]
permalink: /study/:year/:month/:day/:title/
comments: true
---

---

**Maven과 Gradle**

스프링 빌드 도구이다. 기존에 사용하던 빌드 툴 Ant의 불편함을 해소하고자 나온게 아파치의 Maven이고 Maven의 부족함을 채워 나온게 Groovy 기반의 Gradle이다.

Maven은 pom.xml로 관리하며 멀티 프로젝트 구성시 상속 구조를 이용해야하는 불편함이 있는데 Gradle은 설정 주입 방식을 이용해서 빌드 환경 구성이 편하다. (Gradle은 build.gradle 파일로 관리)

프로젝트 단위가 커질 수록 빌드와 테스트가 길어지는데 Gradle은 캐시를 사용하기 때문에 훨씬 속도가 빠르다.

**컴파일(Compile)**

구글 번역기이다. 인간이 쓴 코드를 컴퓨터 언어로 번역해준다.
*기계어\_CPU가 이해할 수 있는 0과 1로 이뤄진 바이너리 코드  
*바이트 코드\_가상머신이 이해할 수 있는 중간 코드(0과 1로 구성, .java -> .class)

**빌드(Build)**

매장 진열대에 올릴 수 있는 상태까지 만드는 것이다. 서버에 올릴 수 있는 완성품 / 실행가능한 상태를 만든다. war, jar, exe로 압축된 파일 형태.

빌드 과정에는 전처리, 컴파일, 패키징, 테스팅, 배포가 있다. 이 빌드 과정을 도와주는 친구가 빌드 도구이고, Ant, Maven, Gradle이 있다. 여기서 배포는 실운영서버 배포가 아닌, 프로젝트 버전 관리를 위한 저장소에 배포하는 것을 의미한다.

**배포(Deploy)**

매장 진열대에 올리는 것이다. 서버에 올려 사용자가 사용할 수 있게 한다.

**좀 더 생각해보기**

빌드 자동화란? 빌드 자동화에 사용하는 Travis CI는 뭐지?

**references**

- 다시 써보는 스프링 5 레시피
- 초보 웹 개발자를 위한 스프링5 프로그래밍 입문
- https://itholic.github.io/qa-compile-build-deploy/
