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

