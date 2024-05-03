---
layout: post
title: "CAP(Consistency, Availability, Partition Tolerence)"
description: "분산 컴퓨팅 데이터베이스 이론 CAP, PACELC"
date: 2024-05-03
tags: DB
comments: true
---
CAP이론은 분산 컴퓨팅 환경에서 데이터베이스의 특징을 정리하기 위해 등장한 이론입니다.

CAP이론에서 이야기 하고자 하는 내용이 무엇인지 알아보고, 현대에서 사용중인 스토리지 엔진들은 어떤 전략을 채택하고 있는지도 알아보겠습니다.

## CAP
먼저, CAP 이론의 세 가지 용어에 대해서 알아보겠습니다.

### Consistency (일관성)
모든 컴퓨팅 자원은 가장 최근에 쓴 데이터를 항상 읽을 수 있어야 함을 의미합니다.
만약, 노드가 총 3대(A,B,C)가 있는 환경에서 A 노드에 데이터가 업데이트 되는 즉시 B와 C노드에서도 업데이트 된 내용이 확인될 수 있음을 의미합니다.

### Availability (가용성)
모든 읽기/쓰기 작업은 가장 최근에 작성된 내용을 반환하지 않더라도 오류 응답을 하지 않습니다.
만약 특정한 노드에 Fault 상황이 발생하더라도, 클라이언트는 다른 노드로 요청을 처리할 수 있음을 의미합니다.

### Partition tolerance (분할 내성)
노드들 간의 Network I/O의 문제가 생기는 경우 (네트워크 장애, Latency, Packet Loss 등)에도 불구하고, 시스템은 정상적으로 동작하는 것을 의미합니다.

CAP이론은 이 3가지의 특성 중 두 가지를 선택하면 한 개는 포기해야 함을 의미하고 있습니다.

다만, 현실적으로 분산 시스템에서 각 노드간 네트워크 통신이 단절되는 상황은 절대적으로 막을 수 없기 때문에 CAP 이론은 `분할 내성(Partition Tolerance)`를 전제하고 `일관성(Consistency)`과 `가용성(Availability)` 둘 중 하나를 선택해야 하는 방향으로 해석됩니다.

다음 사진과 같이 총 세 가지로 분류할 수 있습니다.

![cap](https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/473c729e-f0c5-4b60-b6c6-7476e77e87fc)


## CAP 이론 해석의 주의사항

위에서 `분할 내성(Partition Tolerance)`를 전제하에, `일관성(Consistency)`을 선택한 CP특징 또는 `가용성(Availability)`를 선택하는 AP특징 두 가지로 구분이 되지만 사실 이는 현실적 세계에서 큰 의미가 없습니다.

### 1. 완벽한 일관성(Consistency) 보장은 불가능하다.
완벽한 일관성을 위해 각 노드간 데이터 쓰기 작업은 모든 노드에 동기화되어야 합니다.

다음 그림과 같이 각 노드간 네트워크가 단절된 상황을 예시로 들어보겠습니다.

![network_fault](https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/3132d024-00f1-4d6a-b336-e4e12a752e24)

Node_A로부터 연결되는 구간 Node_C의 통신 구간에 문제가 생겨 더이상 통신이 불가능한 상황에서 일관성을 지키고자 한다면,

더이상 모든 Write I/O 작업을 수행할 수 없게 됩니다. 이론을 정의하는 관점에서는 의미가 있을 수 있지만, 현실적으로 

## Databased System Design

CAP이론을 정립한 Eric Brewer에 따르면(IEEE 내용 발췌), 다음과 같이 전통적인 Transaction 4대 원칙 ACID를 보장하는 RDB의 경우에는 `Consistency(일관성)`에 조금 더 초점을 맞추었고, BASE 철학(Eventual consistency 등)을 기반으로 설계된 Not-Only SQL등의 시스템은 `Availability(가용성)`에 초점을 맞추었다고 합니다.

> Database systems designed with traditional ACID guarantees in mind such as RDBMS choose consistency over availability, whereas systems designed around the BASE philosophy, common in the NoSQL movement for example, choose availability over consistency.

tktlf









