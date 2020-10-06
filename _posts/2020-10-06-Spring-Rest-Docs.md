---
layout:     post
title:      Spring Rest Doc을 활용하여 API 문서 자동화 하기
author:     HUIWOO PARK
tags: 		  SpringFramework SpringBoot
subtitle:  	Swagger보다 구조구억배 좋은 API 문서 자동화 구축하기. (사실 자동이 아닐 수도..)
category:   Spring, SpringBoot
---

> 안녕하세요, 이번 포스팅에서는 현재 사내에서 새롭게 진행 중인 프로젝트에 `Spring Rest Doc`을 이용해서 API 문서를 만든 경험을 공유 해보려고 합니다.

참고사항) 본 포스팅은 Spring Boot 2.3, Gradle Project (6.3version) 을 기준으로 설명합니다. Maven Project의 Dependency 등은 Spring Rest Doc Reference를 참고해주세요.
본 포스팅에 사용된 전체 코드는 github 에서 확인할 수 있습니다.

## 1. 왜 Spring Rest Doc을 채택했을까요 ?
이전에 회사에서 운영하는 서비스들의 API문서는 각 서비스 팀별로 달랐는데요, 제가 속한 팀에서는 Swagger로 API 명세를 클라이언트 개발팀에게 공유하고 있었습니다.

어느정도 일관성을 맞추기 위해 제가 맡게된 신규 프로젝트도 Swagger를 채택하려고 했으나 팀 내에서 고민한 결과 `Spring Rest Doc`을 사용하기로 했답니다 !

그 이유는 다음과 같아요

(1) 클라이언트 개발팀에서의 불편함
사실 개인적으로 Swagger는 서버 개발자의 입장으로 봤을 때 매우 편하다고 생각하는 입장입니다.

(2) 코드가 지저분해짐.

(3) Spring Rest Doc은 테스트가 통과해야만 문서 작성이 됨.
특히나 (3)번이 결정에 있어서 크게 작용했던 부분인데요, 

간단하게, Swagger와 Spring Rest Doc을 비교한 내용을 봅시다.

## 2. Swagger vs Spring Rest Doc


## 3. 시작하기
이제부터 설정을 시작해봐요.

### Domain

### Controller

### build.gradle

### 

#### 참고자료
* Spring Rest Doc Reference Guide
* Woowarbros 기술 블로그
* Woowarbros 기술 블로그
*
