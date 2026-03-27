---
layout: post
title: "Lost In The Middle - How Language Models Use Long Contexts"
description: "LLM 모델의 입력 Context 활용도에 대한 연구 논문"
date: 2026-03-27
tags: AI
comments: true
---

## 서론
Stanford University와 University of California, Berkeley의 연구팀이 현대 LLM 모델에서 입력 컨텍스트 내의 정보의 **길이와 위치**에 따른 성능의 변화를 연구했다. <br>

이 논문의 연구가 2023년도에 이루어져 빠르게 발전된 현재와는 안 맞을 수 있다고 생각할 수 있지만, 지속적으로 더 큰 Context Window를 가진 언어모델이 등장하더라도, 여전히 **트랜스포머(Transformer)기반의 아키텍처**가 다운스트림 태스크 수행 시 입력 컨텍스트를 어떻게 활용하는지에 대해서는 여전히 불분명하며 근본적인 한계가 존재한다. <br>

특히 2026년 현재 화두되고 있는 **AI 에이전틱 시스템(Agentic System)** 처럼 추론(Reasoning), 행동(Action) 계획(Planning)과 실행(Execute)하는 과정에서 장기적으로 긴 컨텍스트를 계속 공유하는 시스템 또는, RAG 등을 구축할 때 검색 엔진 또는 Vector DB에서 가져온 수많은 문서 중 **'배치 순서'**가 모델의 성능에 어떠한 영향을 미치는지에 대한 실험과 그 결과를 제시하므로, 앞으로 우리가 설계하고 개발할 AI Agentic 시스템에서 컨텍스트를 어떻게 설계해야 하는지에 대한 어느정도 가이드라인을 제시해줄 것으로 기대한다.

### 귀무가설(H-Zero) 설정
이 논문의 귀무가설(H-Zero)은 다음과 같이 제시하였다.

"언어 모델이 입력된 정보를 견고하게 활용한다면, 정보의 위치에 관계없이 성능이 일정하게 유지되어야 한다."

지금부터 언급할 모든 실험에 사용되는 두 가지 통제가능한 독립변수는 다음과 같다.
- 컨텍스트의 크기 (Context Size)
- 정보의 위치 (Position)

### 실험 대상 모델
위에서 언급했지만, 2023년에 진행된 연구이기 때문에 비교적 과거에서 가장 성능이 뛰어난 모델로 이루어졌다. 
하지만, 현재 시점에서 등장한 최신 모델들도 위에서 언급한 **길이와 위치**에 대한 편향(Positional Bias)를 보이고 있기 때문에 실험 결과는 현재에서도 여전히 유효한 통찰을 제공한다.
- OpenAI의 GPT 3.5 Turbo
- Claude 1.3

# 참고 문헌
