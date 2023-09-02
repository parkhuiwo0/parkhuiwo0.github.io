---
layout: post
title: "확률적 자료구조 블룸 필터, Bloom Filter"
description: "원소의 집합 내 존재유무를 체크하는 블룸 필터 자료구조에 대해서 알아봅시다."
date: 2023-09-01
tags: DATASTRUCTURE
comments: true
---

# 개요

블룸 필터(BloomFilter) 자료구조에 대해서 알아보자.

블룸 필터 (Bloom filter)는 특정한 원소가 집합에 속하는지 여부를 검사할 때 사용되는 확률적 자료구조 (probability data structure)이다.

여기서 이름 자체에 확률적(probability)이 포함되어있다. 확률적이 의미하는 것은, 가끔 틀린 사실을 뱉어낼 수도 있다는 사실이다.

# 특성

- 집합에 원소를 추가하는 것은 가능하지만, 집합에 원소를 삭제하는 것은 불가능하다.
- 특정한 원소가 집합에 속한다고 판단되었으나, 실제로는 그렇지 않은 **긍정 오류(False Postive)**가 발생할 가능성이 있다.
- 특정한 원소가 집합에 속하지 않았다고 판단되었으나, 실제로는 그렇지 않은 경우 **부정 오류 (False Negative)** 상황은 **없다.**
- 집합의 원소의 개수가 증가할 수록 긍정 오류 발생 확률 또한 증가한다.

# 구조

- *m*비트 크기의 비트 배열(*bit array)* 구조이다.
- *k*가지의 서로 다른 해시 함수(*hash function)*을 사용한다.
    - 각 *hash function*는 입력된 원소에 대해 **************m**************가지의 값을 균등한 확률로 출력해야 한다.
    - 추가(ADD) 오퍼레이션의 경우 추가하려는 원소에 대해 *k*가지의 해시 값을 계산한 다음, 각 해시 값에 대응하는 비트를 1로 설정한다.
    - 검사(CHECK) 오퍼레이션의 경우 해당 원소에 대해 ***************k***************가지의 해시 값을 계산한 다음에, 각 해시값에 대응하는 비트값을 읽고 전체 비트가 1인 경우 원소가 집합에 포함된다고 판단한다.

## 원소가 집합에 존재하는 경우

<img width="632" alt="스크린샷 2023-09-02 오후 7 37 21" src="https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/9ec57f57-bf09-47dc-941d-158dd1344959">


elementA와 elementB는 해싱 알고리즘들에 의해 대응되는 비트가 모두 1이므로, 집합에서 원소가 존재한다고 볼 수 있다.

## 원소가 집합 내에 존재하지 않는 경우

<img width="669" alt="스크린샷 2023-09-02 오후 7 38 42" src="https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/75657fb9-14a5-45e5-b127-8b92d66f8b13">


해시 함수에 의해 대응되는 비트 전체가 1이 아니기에, *element-c*는 집합 내에 원소가 존재하지 않는다.

여기까지 이해가 되었다면, 이 자료구조가 **왜 부정 오류(False Negative)가 발생하지 않는 것인지 이해가 될 것**이라고 본다.

# 확률과 시간복잡도

- 원소를 추가하는 경우에는, 단순 존재하는 해시 함수(Hash Function)들을 거치면 되므로, *O(k)*이다. (k는 해시함수들의 개수)
- m길이의 크기 구조에 자료구조에 원소를 n개 추가하였을 때, 특정 비트가 0일 확률은 다음과 같이 계산할 수 있다.
<img width="134" alt="스크린샷 2023-09-02 오후 8 00 18" src="https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/a409bf1b-f50a-40d4-9a3b-34b901c65e9d">

- 원소를 검사할 때 *k*개의 해시값이 모두 1이 될 확률 (존재할 확률)은 다음과 같이 계산할 수 있다.

<img width="206" alt="스크린샷 2023-09-02 오후 8 00 21" src="https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/335bf77a-e115-4359-a870-9cb85dbc9b47">


# 사용 사례

블룸필터 자료구조는 어떠한 상황에서 사용되는 것이 적합한가? 또 어떠한 상황에서 유용하게 쓰일 수 있는가?

먼저, 블룸 필터 자료구조의 본질은 **원소의 존재 여부를 체크하는 것**에 있다. 따라서, 존재 여부를 체크하는 알고리즘 어디에서나 확장하여 적용이 가능하다고 생각한다.

또한, 블룸 필터는 필터를 위해 해시함수와 비트 마스킹을 통해 적은 양의 데이터를 사용하며, 매우 빠른 속도로 처리가 가능하기 때문에 대량의 데이터의 필터링 전 1차 필터링 기능으로 활용할 수 있겠다.

특히, 원소가 집합 내에 존재하는지 존재하지 않는지 검증할 때, 존재하지 않을 경우를 검사해내는 데에 활용할 수 있다.

대표적인 사례로, 구글의 크롬 웹 브라우저 (Google Chrome)의 세이프 브라우징 기능에 블룸필터가 적용되어있다. (블룸필터 코드는 [다음](https://chromium.googlesource.com/chromium/chromium/+/refs/heads/main/chrome/browser/safe_browsing/bloom_filter.cc)에서 확인할 수 있다.)

구글의 세이프 브라우징 서비스는 특정 리소스 웹 페이지에 대해 블룸필터 자료구조를 통해 1차 필터링을 통해 필터링 대상 페이지인지 확인 후에, 필터링 대상 페이지가 맞다면, 방대한 자료구조에 접근하여 진짜 필터링을 해야하는지 검사하는 원리로 동작하게 된다.

# 참고 문서
- [Network Applications of Bloom Filters: A Survey](https://www.eecs.harvard.edu/~michaelm/NEWWORK/postscripts/BloomFilterSurvey.pdf)
- [구글 크롬 GIT 블룸필터](https://chromium.googlesource.com/chromium/chromium/+/refs/heads/main/chrome/browser/safe_browsing/bloom_filter.cc)
- [위키피디아 블룸필터](https://ko.wikipedia.org/wiki/%EB%B8%94%EB%A3%B8_%ED%95%84%ED%84%B0)

n차원 데이터의 distnace를 고려한 확률적 해싱 알고리즘 Latent Semantic Hashing 또한 유사한 계보이다.
