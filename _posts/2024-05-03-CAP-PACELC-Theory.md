---
layout: post
title: "Theory of Distributed Data Store"
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
만약, 노드가 총 3대(A,B,C)가 있는 환경에서 A 노드에 데이터가 업데이트 되는 즉시 B와 C노드에서도 업데이트 된 내용이 확인될 수 있음을 의미합니다. 최종적 일관성(Eventual Consistency)과 혼동될 수 있지만, 최종적 일관성과는 다른 동일 시점의 일관성을 의미합니다.

### Availability (가용성)
모든 읽기/쓰기 작업은 가장 최근에 작성된 내용을 반환하지 않더라도 오류 응답을 하지 않습니다.
만약 특정한 노드에 Fault 상황이 발생하더라도, 클라이언트는 다른 노드로 요청을 처리할 수 있음을 의미합니다.

### Partition tolerance (분할 내성)
노드들 간의 Network I/O의 문제가 생기는 경우 (네트워크 장애, Latency, Packet Loss 등)에도 불구하고, 시스템은 정상적으로 동작하는 것을 의미합니다.

CAP이론은 이 3가지의 특성 중 두 가지를 선택하면 한 개는 포기해야 함을 의미하고 있습니다.

다만, 현실적으로 분산 시스템에서 각 노드간 네트워크 통신이 단절되는 상황은 절대적으로 막을 수 없기 때문에 CAP 이론은 `분할 내성(Partition Tolerance)`를 전제하고 `일관성(Consistency)`과 `가용성(Availability)` 둘 중 하나를 선택해야 하는 방향으로 해석됩니다.

다음 사진과 같이 총 세 가지로 분류할 수 있습니다.

![cap](https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/473c729e-f0c5-4b60-b6c6-7476e77e87fc)

각 케이스에 대해 그림으로 하나씩 살펴보겠습니다.

이해하기 쉽게, 분산 시스템은 Single Leader - Multi Follower 방식으로 설명하겠습니다.

### C와 P를 만족하는 시스템

Consistency와 Partition Tolerance를 만족하는 시스템을 그림과 표현하면 다음과 같습니다.
![CP_Example](https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/3ba50d9f-a805-4f5e-a02a-8949b6608e0d)

현재 각 컴퓨팅 자원들은 위쪽 partitioned(A)와 partitioned(B)로 분할된 상황입니다.

만약, 위쪽 파티션에 쓰기 요청을 한다면 위쪽 파티션은 마스터 노드와 정상적으로 통신이 가능한 상황이므로 consistency를 보장하고 있으며 아래쪽 파티션은 쿼리에 대한 응답이 불가능합니다.

이러한 상황에서 마스터 노드가 존재하지 않는 아래쪽 파티션은 정상적으로 동작하지 않으므로,  Partition tolerance(분할 내성)의 특징을 가지는 것이 맞는가? 라고 의문이 들 수도 있는데요,

P, 파티션 된 상황에서도, 노드들은 동일한 답변(C, 일관성)를 하기 위해서 아래쪽 파티션은 응답을 포기한(A, 가용성를 포기) 상황이라고 생각하시면 될 것 같습니다.
(즉, 시스템 전체 다운 현상 없이 위쪽 파티션은 제 역할을 하고 있고, 아래쪽 파티션은 역할을 하기는 하지만 거부하는 상황으로 분할 내성의 특징을 가지는 것으로 볼 수도 있겠습니다.)

CAP이론은 이름 그대로 '이론'으로서 Availability와 Consistency의 상충 관계를 설명하고자 하는 것을 기억하시면 좋을 것 같습니다.

### A와 P를 만족하는 시스템
Availability와 Partition Tolerance를 만족하는 시스템을 그림으로 표현하면 다음과 같습니다.
![CA_Example](https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/e4897dbd-d6c9-4638-92bd-160cc64d3922)

파티션이 두 개로 나누어진 상황에서도, 아래쪽은 새로운 파티션의 리더를 선출하고 요청에 따른 응답을 수행합니다.

두 개의 파티션의 마스터가 다르고(Split Brain 현상이라고도 알려짐) 각각에서 관리하는 데이터가 틀어질 수 있기에 C, 일관성을 포기하고 어떠한 순간에도 요청을 거부하지 않는 A, 가용성을 채택하는 구조입니다.

### C와 A를 만족하는 시스템
Consistency와 Availability를 만족하는 시스템을 그림으로 표현하면 다음과 같습니다.

![CA_EXAMPLE](https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/890d5f0d-10f7-4724-ad95-c215efa218ad)

![CA_EXAMPLE_2](https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/2cb75eb6-8b4d-451a-bee0-9f2285af315f)

마스터 노드와 연결이 끊어지는 아래쪽의 파티션의 경우 아래쪽 파티션으로는 더이상 아무런 요청이 접수되지 않습니다. (라우팅 자체가 되지 않는다고 생각하시면 될 것 같습니다.)

CA는 파티션이 나누어졌을 때 아래쪽 파티션으로는 더이상 요청이 가지 않는 것. 즉, 파티션을 허용하지 않음으로 생각하면 될 것 같습니다.

CA와 CP의 사례는 비슷하여 다소 혼란의 여지가 있는데요, 사실 현실적인 엔지니어링 세계에서 CA 시스템은 찾기 쉽지 않은 것 같습니다.

CAP 이론이 세상에 등장하고 이를 해석하고, 받아들이는 매우 다양한 견해가 있습니다.

저는 개인적으로 P(분할 내성)은 반드시 시스템이 갖추어야 하는 요건 중 하나이고 여기서 A(가용성) 또는 C(일관성) 둘 중에 어디에 조금 더 초점을 맞출 것인가? 로 해석합니다.
(왜냐하면 어떠한 이유로든 간에 네트워크 파티션은 항상 나누어질 수 있다는 점을 감안해야 하기 때문입니다. (물리적 이슈(네트워크 장비 결함), 전기적 신호 장애 등)


## CAP 이론 해석의 주의사항

위에서 `분할 내성(Partition Tolerance)`를 전제하에, `일관성(Consistency)`을 선택한 CP특징 또는 `가용성(Availability)`를 선택하는 AP특징 두 가지로 구분이 되지만 사실 이는 현실적 세계에서 큰 의미가 없습니다.

### 1. 완벽한 일관성(Consistency) 보장은 불가능하다.
완벽한 일관성을 위해 각 노드간 데이터 쓰기 작업은 모든 노드에 동기화되어야 합니다.

다음 그림과 같이 각 노드간 네트워크가 단절된 상황을 예시로 들어보겠습니다.

![network_fault](https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/3132d024-00f1-4d6a-b336-e4e12a752e24)

Node_A로부터 연결되는 구간 Node_C의 통신 구간에 문제가 생겨 더이상 통신이 불가능한 상황에서 일관성을 지키고자 한다면,

더이상 모든 Write I/O 작업을 수행할 수 없게 됩니다. 이론을 정의하는 관점에서는 의미가 있을 수 있지만, 현실적으로 

## Database System Design

CAP이론을 정립한 Eric Brewer에 따르면(IEEE 내용 발췌), 다음과 같이 전통적인 Transaction 4대 원칙 ACID를 보장하는 RDB의 경우에는 `Consistency(일관성)`에 조금 더 초점을 맞추었고, BASE 철학(Eventual consistency 등)을 기반으로 설계된 Not-Only SQL등의 시스템은 `Availability(가용성)`에 초점을 맞추었다고 합니다.

> Database systems designed with traditional ACID guarantees in mind such as RDBMS choose consistency over availability, whereas systems designed around the BASE philosophy, common in the NoSQL movement for example, choose availability over consistency.

## 참고 자료
- [CAP Twelve Years Later: How the "Rules" Have Changed](https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed/)
- [Brewer’s CAP Theorem](https://www.julianbrowne.com/article/brewers-cap-theorem/)









