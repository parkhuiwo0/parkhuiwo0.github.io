---
layout: post
title: "리드 어헤드 (Read Ahead)"
description: "파일 시스템 그리고 탐색에서 중요하게 작용하는 리드 어헤드(Read Ahead) 용어에 대한 정리"
date: 2023-06-11
tags: DB
comments: true
---

리드 어헤드의 용어에 대해서 간단하게 알아봅시다.


## 리눅스 커널의 리드 어헤드(Read Ahead)

Linux Kernal의 System Call 중 하나이다.

파일의 내용을 미리 캐시에 로드하여, 파일에 액세스 할 때 캐시 레이어에서 읽어오기 때문에 디스크 I/O 효율 증가(파일 액세스 latency 감소) 등의 이점을 누릴 수 있다.

파일에 엑세스할 때 디스크 레벨이 아닌, 메모리(RAM 등)에서 내용을 읽어오기 때문에 File Access Latency 감소 효과가 있다.

특히나 순차적으로 접근되는 패턴이 예상되는 경우에 꽤나 많은 이점을 누릴 수 있다. (특히나 플래터가 회전하면서 읽는 HDD에서)

- Read-Ahead with File Unit (파일 단위 리드 어헤드)
    - 디스크 수준의 파일의 특정 블록을 단위로 미리 읽는다.

- Read-Ahead with Page Unit (페이지 단위 리드 어헤드)
    - 메모리 수준의 페이지(Page) 단위로 데이터를 미리 읽는다.


## RAID 컨트롤러 캐시의 리드 어헤드(Read Ahead)

RAID의 컨트롤러 캐시의 정책 중 읽기 정책에서도 리드 어헤드와 관련된 내용이 언급된다.

읽기 요청이 들어왔을 때, 특정 읽기를 수행하려고 하는 주소 이후의 내용까지도 일부분을 읽어와 메모리에 저장한다.

리드 어헤드는 맥락이 비슷하다는 것을 확인할 수 있다.

미리 읽어온 데이터를 I/O 비용이 저렴한 캐시레이어에 올려두고, 접근하도록 하는 기술인 것이다.


## DB 시스템 내의 리드 어헤드(Read Ahead) - InnoDB Buffer Pool Prefetching

DB도 결국 파일을 관리하는 스토리지이기 때문에 내가 사용하는 MySQL에서도 리드 어헤드가 적용되어있는지 확인해봤다.

처음 사용자 요청에 의해 페이지 블록이 DB엔진의 포그라운드 스레드에 의해 처리되며, 순차적으로 데이터가 읽히는 패턴이 생기면 백그라운드 스레드에 의해 다음 페이지들을 미리 읽어 버퍼풀(Buffer-Pool)에 저장하는 기능이 있다.

연관된 환경변수는 다음과 같다. 

- `innodb_read_ahead_threshold` : 얼마나 많은 페이지가 순차적으로 읽혀야 리드어헤드가 활성화 되는지

통계와 관련된 정보를 확인하기 위해서는 다음 정보를 활용한다.
- `Innodb_buffer_pool_read_ahead`
- `Innodb_buffer_pool_read_ahead_evicted`
- `Innodb_buffer_pool_read_ahead_rnd`

![스크린샷 2023-06-13 오후 5 01 19](https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/a94fb672-a3f6-4c06-aaf9-f35144e7f44f)

## 정리하며

Read-Ahead에 대한 용어를 얼떨결에 들었는데, 생각보다 탐색과 관련된 부분에서 중요하게 작용되는 용어인 것 같아 기억을 하기 위해 정리해보았다.

또한, 내가 사용하고 있는 DB(InnoDB)에서도 비슷한 역할을 해주고 있었기에 조금 더 흥미롭게 보았다.

## 참고 문서
- [RAID and filesystems](https://raid.wiki.kernel.org/index.php/RAID_and_filesystems)
- [Configuring InnoDB Buffer Pool Prefetching (Read-Ahead)](https://dev.mysql.com/doc/refman/8.0/en/innodb-performance-read_ahead.html)

