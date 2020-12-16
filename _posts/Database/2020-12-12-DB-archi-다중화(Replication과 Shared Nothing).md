---
layout: post
title: 데이터베이스 아키텍처 - 다중화(Replication과 SharedNothing)
category: Database
tags: [DB, 다중화, 클러스터링, 리플리케이션, Shared Nothing, 샤딩]
comments: true
---

# Index

- 데이터베이스의 아키텍처
  - 역사와 개요
  - 가용성과 확장성의 확보
- **다중화(Redundancy)**
  - 클러스터링(Clustering) - DB 서버의 다중화
  - **리플리케이션(Replication) - DB 서버와 데이터의 다중화**
  - **Shared Nothing - 성능을 추구하기 위한 다중화**

---

책 데이터베이스 첫걸음 - 4장 데이터베이스와 아키텍처 구성을 정리한 내용입니다.

---

# Replication - DB 서버와 데이터의 다중화

<br>

이전 포스트에서 설명한 Active-Active와 Active-Standby 클러스터 구성에서는 서버 부분은 다중화해도 저장소 부분은 다중화하지 않는다는 공통적인 단점이 있었다.
즉 저장소에 문제가 생기면 데이터를 잃게 될 것임을 의미한다! 이런 상황에 대응할 수 있는 클러스터 구성이 리플리케이션(Replication, 복제)이다.

리플리케이션(Replication)이란 DB 서버와 저장소를 세트로 하여 복수로 준비하는 것을 의미한다.

![alt text](/public/img/archi/db_replication_basic.png "리플리케이션은 데이터를 복제한다")

사용자가 마스터DB 서버에 (SELECT/INSERT/UPDATE/DELETE) 작업을 하면 Master DB는 그 데이터를 Slave DB 쪽으로 복제를 하게 된다. 하지만, 이 구조에서는 Slave DB가 놀게 된다는 단점이 있다.

이를 보완하기 위해 Master DB에는 쓰기만, Slave DB에는 읽기만 허용함으로써 DB 서버의 부하를 분산시킬 수 있다.

![alt text](/public/img/archi/db_replication_m_s.png "M에서는 쓰기만, S에서는 읽기만")

## Master - Slave 간의 동기화 방법

- 해당 구조에서 주의할 점은 Standby(Slave)측 데이터의 최신화를 유지해야 한다는 점이다. 즉, Master 디비와 Slave DB간의 동기화를 일정 주기로 수행해 나가야 하는데, Slave 측 갱신 주기를 얼마로 할 것인가와 성능 사이에 trade-off 관계가 생겨난다.

- (?) 실무에서는 어떻게 동기화 하는가, 캐시 이용? 일정주기로? 타임스탬프?

# 성능을 추구하기 위한 다중화 - Shared Nothing

M-S 구조임에도 불구하고, 테이블 데이터 자체가 많을 경우 Slave DB를 N대로 늘리더라도 데이터 검색 속도는 빨라지지 못 할 것이다. 이때 사용가능 한 것이 바로 Shared Nothing 구조이다.

## Shared Disk와 Shared Nothing

복수의 서버가 한 개의 저장소를 공유하는 것을 Shared Disk라 하며 네트워크 이외의 자원을 아무것도 공유하지 않는 방식을 Shared Nothing 구조라 한다.

Shared Nothing에서는 DB 서버와 저장소로 구성된 세트를 늘려서 저장소가 병목 지점이 되는 것을 방지한다. Shared Nothing 구조로 샤딩이 있다. 하지만, 저장소를 공유하지 않는 것은 결국 '각각의 DB 서버가 동일한 1개의 데이터에 접근 할 수 없다'를 의미한다. 예로, 시 단위 'DB 서버 + 저장소' 세트를 갖춘 Shared Nothing 구성에는 고양시 데이터를 가진 DB 서버가 접근할 수 있는 것은 고양시 데이터뿐이다. 때문에 시별 인구 데이터를 합산해서 경기도 인구를 계산하려는 경우 각 세트로부터 시별 인구를 모아서 집계하는 정리 서버가 필요하게 된다. 또한 고양시의 DB 서버가 다운되었을 때는 고양시 데이터에 접근 할 수 없는 문제가 발생한다.

이런 문제에 대처하려면 DB 서버 하나가 다운되었을 때 다른 DB 서버가 이를 이어 받아 계속 처리할 수 있게 하는 커버링(Covering 구성) 등을 고려해야한다.

---

## [[다음 포스트 보기] - 샤딩](https://yjna2316.github.io/database/2020/12/13/Sharding/)
