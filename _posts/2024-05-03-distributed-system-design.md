---
layout: post
title: "Considerations in Distributed System Design"
description: "분산 시스템에서 고려해야 할 사항들에 대한 기본 개념"
date: 2024-05-03
tags: Architecture
comments: true
---

분산 시스템을 다루거나 공부할 때마다 항상 일관성, 중복, 순서보장 등에 대해서 많은 고민을 하게 된다.

발생할 수 있는 문제, 컴퓨터 과학에서 이때까지 정리된 많은 이론적 내용 등을 잊어버리지 않고 기록하려고 합니다.

## Replication Problems

복제가 필요한 이유는 다음과 같다.
- Physical Locality 확보를 통해 데이터 전송 시간을 축소하기 위해서
- 일부 노드가 fault 상황이어도 정상적으로 동작하게 처리를 하기 위해서
- READ I/O 작업에 대해서 퍼포먼스를 올리기 위함

### (1) Replication Lag Problem (복제 지연 문제)

