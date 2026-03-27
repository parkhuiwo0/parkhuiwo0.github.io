---
layout: post
title: "Lost In The Middle - How Language Models Use Long Contexts"
description: "LLM 모델의 입력 Context 활용도에 대한 연구 논문"
date: 2026-03-27
tags: AI
comments: true
---

# 서론
Stanford University와 University of California, Berkeley의 연구팀이 현대 LLM 모델에서 입력 컨텍스트 내의 정보의 길이와 위치에 따른 성능의 변화를 연구했다.
아무래도, Transformer 기반의 현재 LLM 모델들은 이 논문에서 연구한 내용과 결론에 대해 아직도 해답을 찾고 있는 과정이고 
2026년 현재 화두되고 있는 AI Agentic System과 같이 지속적으로 사용자 프롬프트에 대해 추론(Reasoning)과 행동(Action) 그리고 계획(Planning)과 실행(Execute)하는 과정에서
장기적으로 긴 컨텍스트를 계속 전달해가며 이동시키는 

이는 단순히 입력에 대한 Context Size를 늘리는 것에 대한 기술적 내용이 아니라,
Context 내에 정보에 따라 LLM 모델의 성능이 어떻게 달라지는지, 그리고 앞으로 우리가 설계하고 개발할 AI Agentic 시스템에서 컨텍스트를 어떻게 설계해야하는지에 대한 어느정도 가이드라인을 제시해줄 것으로 기대한다.

### Null Hypothesis
이 논문의 귀무가설(H-Zero)는 "LLM 모델이 정보를 잘 활용한다면, 정보의 위치는 상관이 없어야 한다."로 설정되어있다.

지금부터 언급할 모든 실험에 사용되는 두 가지 통제가능한 독립변수는 다음과 같다.
- 컨텍스트의 크기 (Context Size)
- 정보의 위치 (Position)

# 참고 문헌
