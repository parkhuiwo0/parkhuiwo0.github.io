---
layout: post
title: "테이블 설계 시 기본키 (Primary Key)를 Surrogate Key를 사용할 것인가? Natural Key를 사용할 것인가?"
description: "테이블 설계의 기본, 기본 키의 기준을 어떻게 정할 것인가에 대한 의사결정 과정의 기록"
date: 2023-08-22
tags: DB
comments: true
---

# 개요

최근에 회사에서 프로젝트를 진행하면서, RDB 리소스의 신규 데이터 스키마가 필요했고 테이블을 설계하는 과정에서 고민했던 내용 중 하나를 기록하려 한다.

참고로 나는 프로젝트를 진행하면서 MySQL 중에서도 InnoDB 스토리지 엔진을 사용했기에 이 포스팅에는 InnoDB 스토리지 엔진에 의존된 내용들이 많이 포함된다.


## 고민의 내용은 다음과 같다.

MySQL InnoDB 스토리지 엔진에는 모든 테이블에 레코드(행)를 고유하게 식별할 수 있는 Primary Key가 존재한다. (PK를 선언하지 않았더라도.)

만약 우리가 정의한 테이블의 스키마에서 어떤 특정한 레코드(행)을 고유하게 식별할 수 있는 어트리뷰트(컬럼)이 존재한다면 즉, 자연키(Natural Key) 해당 어트리뷰트 혹은 어트리뷰트들의 집합으로 기본키(Primary Key)를 두는 것이 좋을까?

아니면, 대체키 (Surrogate Key) (거의 대부분의 테이블에서 보이는 패턴인 ID 즉, Auto Increment)를 기본키로 두는 것이 좋을까?

먼저 결과를 스포하자면, **나는 대체키(Auto-Increment 속성을 가지는 ID)를 기본키**로 선택하였다. 다만, 그 *결정에 대한 근거 및 논리는 빈약한 상태*이다. 

오랜 기간동안 리서치를 해보지는 않았지만 DB 진영에서 아직도 이러한 주제로 갑론을박이 많은 것 같다.

비슷한 고민을 하거나 좋은 아이디어 혹은 영감이 떠오를 때마다 지속적으로 이 포스팅에 업데이트 하도록 할 것이다.

아래에 적힌 의사결정 근거들은 단순 내가 대리키를 기본키로 택한 이유를 나열하는 것이 아니라 대리키를 기본키로 택하는 과정에 있어서 고민했던 내용들 그리고 알아낸 정보들을 나열한 것이다.


## *:의사결정 근거#1 - IEEE 논문 HighPerformance SQL*

아래는 *[High Performance SQL - Finesse for Lucrative Programming](https://www.academia.edu/12998233/High_Performance_SQL_IEEE_)* 논문으로부터 발췌한 글이다.

> *33) Composite Key vs. Surrogate Key: Replacing ugly, big composite keys and natural keys with tight integer surrogate key is bound to enhance join performance. The storage requirements are minimized & index lookups would appear to be simpler. A Surrogate Key could be a system created sequence number or a grouping of parts of a column that serve to make the row unique. (33 of 3 Page)
큰 복합키와 자연키를 정수 대리키(Surrogate Key)로 교체하면 조인 성능이 향상된다.스토리지 요구 사항이 최소화되고, 인덱스 조회가 더 간단하게 보여진다. 대리키는 시스템에서 만든 시퀀스 넘버 (auto-increment와 같은)이거나, 행을 고유하게 만드는 데 사용되는 컬럼(Attribute)의 일부 집합이다.*


만들려는 테이블의 특성상 테이블의 속성 전체 혹은 일부가 다른 테이블에서도 식별자로 쓰여진다면 (어떠한 관계를 표현하거나, 유일성이 필요없는 키로 사용된다면) 다른 테이블의 레코드 사이즈도 증가하고, 조인에 대한 부담감도 커지는 것은 사실이다.


## *:의사결정 근거#2 - 클러스터 된 인덱스 (Clustered Index of InnoDB Primary Key)*


## *:의사결정 근거#3 - InnoDB Index Data Structure Size*

InnoDB의 Secondary Index는 스토리지의 물리적인 주소를 가지고 있지 않고, 데이터 주소를 가지고 있는 Primary Key의 주소를 가르키고 있다.

아래는 간단하게 세컨더리 인덱스의 구조를 나타낸 것이다.

![스크린샷 2023-09-02 오후 11 53 14](https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/59cfa8d6-54ef-4865-b2e0-e5b3e1f321e6)


만약 Primary Key 자체의 사이즈가 증가된다면, Primary Key를 가지고 있는 **모든 세컨더리 인덱스의 사이즈 또한 증가하게 된다**는 단점이 존재한다.



## *:의사결정 근거#4 - Storage Optimizing (데이터 탐색 패턴)*



## *:의사결정 근거#5 - Locking M**echanism***

만약 단일 키가 아닌 복합키를 자연 기본키로 사용하는 상황임을 가정해보자.

InnoDB 스토리지 엔진의 동시성 제어 메커니즘의 Lock은 Index를 기반으로 동작한다.

이러한 상황이 존재할 지는 모르겠으나, 만약 복합 컬럼 (A,B,C)로 기본키가 구성되어있다고 가정해보자.

만약, A,B,C를 조건으로 레코드 잠금을 수행하려고 시도할 때, 옵티마이저가 A,B만을 가지고 인덱스 레코드 탐색에 성공(혹은 A만을 가지고)했고 C는 필터링 조건으로 사용되었다면 

A,B를 기준으로 탐색된 인덱스 레코드 전체에 잠금이 걸리게 된다.

즉, 불필요하게 과도한 레코드에 잠금이 발생할 여지가 있음을 의미한다.

다만, Surrogate Key를 auto-increment의 PK로 두며 A,B,C 를 기본키로 두지 않고 Composite Secondary Index로 두더라도 비슷한 현상은 있지만 PK라는 대안책이 있기에 괜찮다고 생각한다.

(이 사실은 정확하지 않으며, 상상에 의해 작성된 내용임을 한 번 더 강조한다.)



# 참고 문서

- [도서] Real MySQL 8.0 vol1
- [논문] [IEEE HIgh Performance SQL](https://www.academia.edu/12998233/High_Performance_SQL_IEEE_)
-
