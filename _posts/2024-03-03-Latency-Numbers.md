---
layout: post
title: "Latency Numbers Every Programmer Should Know"
description: "모든 프로그래머가 알아야 할 지연 시간"
date: 2024-03-03
tags: ETC
comments: true
---

# Latency Numbers Every Programmer Should Know

각각의 행위(연산 등)에 대한 대략적인 시간을 알고 있으면 도움이 될 것 같아서 포스팅으로 기록해두기 위한 것.

특히 분산 시스템에서는 각 구간과 행위에 Latency를 고려하여 일관성을 맞추거나 캐싱 전략을 세우거나 등 고민이 많이 필요한데 도움이 될 것 같다.

- L1 Cache Reference : 1ns
- L2 Cache Reference : 4ns
- Mutex lock/Unlock : 17ns
- SSD Random Access I/O (Read) : 17μs
- Send 2,000 bytes over commodity network : 44μs
- Sequentially Read 1MB(1,000,000 bytes) from SSD : 49μs
- Main Memory Reference : 100ns
- Sequentially Read 1MB(1,000,000 bytes) from Disk : 825μs
- Disk Seek : 2ms
- Packet roundtrip CA to Netherlands : 150ms

읽어주셔서 감사합니다.

## 참고문서
- https://colin-scott.github.io/personal_website/research/interactive_latency.html
