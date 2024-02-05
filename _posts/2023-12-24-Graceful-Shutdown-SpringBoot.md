---
layout: post
title: "Graceful Shutdown in Spring Boot Application"
description: "봄의 고상하고 기품이 있으며 아름다운 종료처리"
date: 2023-12-24
tags: SpringFramework
comments: true
---

# 그레이스풀 셧다운 (Graceful Shutdown)

## 우아한 종료, Graceful Shutdown

우리는 애플리케이션을 구동하기 위해서 자원(스레드를 생성하거나, HTTP 통신을 위한 커넥션 풀을 만들거나, DB I/O를 위한 커넥션 풀을 만들거나 등)을 사용한다. 

사용할 자원을 적절히 할당하는 것도 중요하지만 그만큼 중요한 것은 사용된 자원을 회수하는 것이다.

Graceful Shutdown은 말 그대로 우아한 종료를 의미한다.

우아한 종료란 '애플리케이션이 종료될 때 별도의 `사이드 이펙트`를 발생하지 않는 것'을 의미한다. (대부분의 사이드 이펙트는 진행중인 프로세스가 중단됨에 따라 작업이 정상적으로 처리되지 않거나, 하드웨어 리소스가 반환되지 않거나 등이 원인일 것이다.)

그러면 어떻게 애플리케이션을 우아하게 종료할 수 있을까?

시스템의 전원을 강제적으로 차단하지 않는 이상 우리는 애플리케이션이 종료된다는 신호를 전달 받을 수 있다.

일반적으로 리눅스 환경에서 서비스를 많이 하기 때문에 다음 [리눅스 Termination Signal 문서](https://man7.org/linux/man-pages/man7/signal.7.html)를 참고하면 대표적으로 애플리케이션을 종료할 때 사용하는 두 가지 터미네이션 신호는 다음과 같다.

- SIGTERM
- SIGINT

Java에서는 다음 Runtime 클래스([Java 11 Runtime Document](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Runtime.html#addShutdownHook(java.lang.Thread)))에서 종료 시그널을 전달 받고 처리하는 ShutdownHook을 등록할 수  있다.

<img width="1631" alt="스크린샷 2024-02-04 오후 5 54 10" src="https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/55aca50d-9648-44c8-ac9a-6efb64ad8957">


일반적인 자바 애플리케이션은 우리는 생성한 리소스들을 이렇게 셧다운 훅을 통해 정리할 수 있다.

## 배포 전략에 따른 자원 회수

요즘 대다수의 애플리케이션은 Blue/Green Deployment 혹은 Canary Deployment 전략을 채택하고 있기 때문에, 신규 버전의 애플리케이션으로 트래픽을 전환하고 이전 버전의 애플리케이션은 종료되지 않고 최소한 몇 분은 기다리고 있다.

따라서 이전 버전이 즉시 종료되지 않기 때문에 자원회수에 신경을 쓰지 않아도 몇 분 이내(애플리케이션이 종료가 되기 전에)에 자원이 자동적으로 회수 되어 문제 상황이 없었을 수도 있다.

하지만 이는 명백히 소프트웨어의 결함이고, 언제든지 시스템의 신뢰성을 깨트릴 수 있는 요인 중 하나이다.

### AWS Spot Instance (AWS 스팟 인스턴스)

스팟 인스턴스에 대한 자세한 내용은 [AWS 공식 문서 링크](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/using-spot-instances.html)를 참고하면 된다.

AWS만 한정해서 이야기 하는 것이지만 스팟 인스턴스를 사용하는 애플리케이션은 관리자(개발자 등)의 의지와는 상관없이 언제든지 리소스가 종료될 수 있기 때문에 Graceful Shutdown은 더더욱 신경써야 한다.

현재 내가 근무하고 있는 조직에서는 스팟 인스턴스를 매우 잘 활용하고 있기(비용절감의 목적으로) 때문에 Graceful Shutdown에 꽤 많은 신경을 쓰고 있다.

## Graceful Shutdown in Spring Application

나는 스프링을 사용하여 개발하고 있기 때문에, 스프링 애플리케이션을 가지고 Graceful Shutdown을 어떻게 구성할 수 있는지 몇 가지 예시를 작성하겠다.

### 1. 톰캣 스레드 풀 자원 회수 (HTTP Tomcat Thread Pool)

만약 웹 서버에 클라이언트가 요청을 보내고 해당 요청이 처리되기 전에 (HTTP Response가 전달되기 전에) 서비스가 중단되면 어떻게 될까?

다음 간단히 정리한 그림과 같이 당연히 응답은 정상적으로 나가지 않게되고 클라이언트는 응답이 올 것이라는 정상적인 상황에 대한 신뢰도가 깨지게 된다.

<img width="791" alt="스크린샷 2024-02-04 오후 5 43 25" src="https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/ed7db8dd-9759-4dbc-b703-35de09800e5c">


시스템의 신뢰성을 지키기 위해서는 요청을 처리중인 톰캣 스레드가 존재한다면 해당 요청이 모두 완벽히 처리될 때까지 애플리케이션은 종료되면 안 된다.

어떻게 처리중인 스레드가 존재할 때 애플리케이션을 중단시키지 않고 요청을 처리하도록 유지할 수 있을까?

만약 Spring Boot 2.3 이상의 버전을 사용하고 있다면 다음과 같이 설정에 추가해주면 된다.

```yaml
server:
  shutdown: graceful
```

Spring Boot 2.3 미만의 버전을 사용하고 있다면 Spring Framework의 [ContextClosedEvent 클래스](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/event/ContextClosedEvent.html) 를 이용하여 애플리케이션 콘텍스트가 종료될 때 WAS 스레드 자원회수를 신경써주면 된다.

shutdown 옵션을 `graceful로` 설정할 경우 다음과 같이 요청을 처리 중에 셧다운 이벤트가 들어오더라도 해당 요청을 모두 처리하고 WAS 스레드가 반납될 때까지 대기하는 것을 볼 수 있다.

<img width="1606" alt="스크린샷 2024-02-04 오후 7 17 55" src="https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/85182600-149f-44fe-aa89-b8d561d92ab0">

만약, 셧다운이 처리되고 있는 서버 애플리케이션에 새로운 요청이 들어오면 어떻게 될까?

최초로 요청을 보내고 해당 요청이 처리되기 전 shutdown signal을 통해 애플리케이션이 종료 절차를 밟도록 했다.

그리고, 새로운 요청을 여러 번 보낸 결과 아래의 사진과 같이 TCP 패킷의 RST Flag를 통해 요청을 거부하는 것을 볼 수 있다.

<img width="1547" alt="스크린샷 2024-02-04 오후 6 23 54" src="https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/97b57d64-0a8d-4b2d-9e76-92d36d854b5e">


Graceful Shutdown을 사용하면서 한 가지 더 살펴봐야 하는 옵션이 있는데 다음 셧다운 Phase의 타임아웃을 설정하는 옵션이다.

다음 옵션은 애플리케이션이 종료 시그널을 받고 일정 시간 동안 자원이 회수가 되지 않으면 강제로 종료하겠다는 옵션을 제공한다.

```yaml
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s # default : 30s
```

만약, 특정한 요청사항이 매우 길다면 (어떤 fan-out 현상 등에 의해) 해당 작업이 완료되기 까지 기다려야 할 수 있다.

여기서 주의사항은 해당 옵션은 무조건 30초를 기다린다는 것이 아니다.

만약, WAS 스레드의 모든 자원이 회수되었다면 30초가 지나지 않았더라도 애플리케이션을 즉시 종료한다는 점을 꼭 기억해야 한다.

그러면, WAS 스레드 외에 또 어떤 상황이 있을까?

### 2. 스레드 자원 회수 (Thread Resource Reclamation)

톰캣 스레드가 아닌, 시스템에서 자체적으로 생성한 스레드 풀 (즉 톰캣 애플리케이션이 감지할 수 없는 영역의 리소스 중 하나)에 제출된 작업도 완료될 때까지 신경을 써야할 필요가 있다.

만약, Java 코드를 통해 Thread Pool을 자체적으로 생산했고 웹 요청에 의해 해당 스레드 풀에 작업이 제출되었다고 가정해보자.

<img width="988" alt="스크린샷 2024-02-04 오후 6 33 31" src="https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/7e187086-b5c2-4731-877e-1b6e79a79712">

마침 또, 우리가 감지할 수 있는 Tomcat Thread Pool의 스레드가 해당 별도 스레드 작업에 대한 완료여부를 신경쓰지 않는 구조라면 (WAS 스레드는 별도의 스레드 풀에 작업 제출을 하고 완료 여부를 신경쓰지 않고 바로 응답을 수행 후 자원 반납)

언제든지 WAS 애플리케이션이 종료될 때 별도 스레드 풀에 제출된 작업이 완료되지 않고 종료될 수 있다.

흔히 자바, 스프링 애플리케이션을 개발하면서 많이들 하는 실수가 다음과 같은 실수라고 생각한다.

- 아래와 같이 CompletableFuture을 사용하여 runAsync(Runnable)에 제출된 작업이 오래 걸리는 상황인 경우
- 해당 작업의 완료 여부를 톰캣 스레드가 신경쓰지 않는 경우

이런 상황에서는 언제든지 애플리케이션은 Graceful Shutdown을 완벽하게 보장하지 못한다.

<img width="754" alt="스크린샷 2024-02-04 오후 6 39 37" src="https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/9e845121-5828-4ae9-8ca7-29c0980ca86b">


다음 시나리오를 예제 코드를 통해 간단하게 살펴보자.

- WAS 스레드가 50초가 소요되는(50초를 슬립하는) 작업이 스레드 풀에 제출한다.
- 제출 즉시 애플리케이션은 종료 시그널을 전달 받는다.
- WAS 스레드는 제출한 작업을 기다리지 않는다.


<img width="685" alt="스크린샷 2024-02-04 오후 6 45 12" src="https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/cd26a94b-9a8c-46f5-9857-40a8b4e23efc">


다음 사진의 결과처럼, 비동기 작업이 제출되었으나 완료되지 않고 애플리케이션이 즉시 종료되는 상황을 볼 수 있다.

<img width="1566" alt="스크린샷 2024-02-04 오후 6 46 08" src="https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/a041ebb1-e0e1-449d-9cfb-ed0981caae57">


그러면, 어떻게 별도 스레드풀에 제출된 작업 또한 우아하게 종료되도록 할 수 있을까?

우리는 위에서 Tomcat WAS 스레드풀에 대한 상태를 감지할 수 있도록 변경했다. 제일 쉬운 방법은 Tomcat WAS 스레드 풀의 스레드가 비동기로 제출된 작업이 완료되도록 기다리면 된다.

하지만, 이 방법이 근본적인 해결 방법은 되지 않는다. 때로는 시스템의 반응성을 위해 클라이언트에게 빠르게 결과를 응답해주어야 할 때도 있기 때문이다.

나는 다음과 같이 별도의 스레드 풀을 스프링 빈으로 관리하고, 해당 스레드 풀을 빈 소멸 (DisposableBean)단계에서 작업이 모두 완료될 때까지 기다리도록 대기했다.

<img width="1009" alt="스크린샷 2024-02-04 오후 6 52 48" src="https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/f2655e3a-bf68-4a81-bfc9-801324fac206">

방법은 여러가지가 있을 수도 있겠다. 하지만, 나는 스프링 애플리케이션을 사용하고 있었고 빈 소멸 단계에서 적절히 자원을 회수하는 것이 조금 더 직관적이라고 생각해서 이렇게 작업하였다.

위에서 봤던 예제 코드블럭을 다음과 같이 수정하고 테스트해보자.

<img width="656" alt="스크린샷 2024-02-04 오후 6 56 58" src="https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/5f7f6346-775c-4c6a-9402-46fdcf2865ed">

<img width="1526" alt="스크린샷 2024-02-04 오후 6 58 45" src="https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/7939839c-478e-4845-bbf0-8fafad0e00d6">

또 다른 방법은 스프링에서 제공하는 [`ThreadPoolTaskExecutor`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.html)를 활용하는 방법도 있다.

다음 JavaDoc에서 확인할 수 있듯이 `setWaitForTasksToCompleteOnShutdown` 함수를 이용하여 셧다운 시 작업 완료 대기를 설정하면 된다.

다만, 조금 더 스레드 풀의 상세한 설정이 필요하고 직접 `Executor` 관련 인터페이스를 제어하고자 한다면 위에서 작업한 코드처럼 별도의 제어가 필요하다.

<img width="1836" alt="image" src="https://github.com/parkhuiwo0/parkhuiwo0.github.io/assets/48363085/23636eb0-605c-49ca-9ff2-d44ebd17ace0">


### 3. 카프카 컨슈머의 처리중인 Offset 관리

만약, 카프카 컨슈머의 ackMode가 `BATCH` 로 설정되어있고, 한번에 많은 양의 오프셋을 `poll()` 했을 때 처리하다가 중간에 종료 시그널을 받게 되면 어떻게 될까?

기본적으로 Shutdown Phase가 30초로 설정되어있기 때문에 30초 이내에 작업이 완료되고 커밋된다면 상관없겠지만 컨슈머가 `poll()`을 수행하고, 가져온 레코드가 엄청 많을 경우 30초 이상의 시간이 소요될 가능성이 존재한다.

- 실제로 작업은 처리되었으나, 오프셋 커밋에 실패해 다른 인스턴스에서 중복 메시지가 수신될 가능성
    - (물론 애초에 카프카 컨슈머는 멱등성 있도록 설계해야 한다.)

만약 스프링 부트 2.7 이상의 버전을 사용하고 있다면, 아래 옵션을 통해 종료 이벤트를 수신하면 즉시 처리중인 오프셋까지만 커밋을 하고 더 이상 poll을 하지 않도록 설정할 수 있다.

```yaml
spring:
    kafka:
        listener:
            immediate-stop: true
```

이 옵션이 정확히 어떻게 동작하는지 보고 싶다면, `org.apache.kafka.common.KafkaException` 를 참고하면 된다.

만약, 부트 2.7 이하 버전을 사용하고 있다면 별도 셧다운 훅에 카프카 리스너(컨슈머)들에 대한 제어를 해주면 된다. (KafkaException과 Runtime Shutdown Hook을 이용한다면 어렵지 않게 구현할 수 있다.)

## 정리하며
애플리케이션에 적절한 자원을 할당하는 것도 매우 중요하지만, 그만큼 사용된 자원들을 적절히 해제하는 것도 매우 중요합니다.

특히나 제가 근무하는 조직은 관리자의 의지와는 상관없이 언제든지 서비스가 종료될 수 있는 AWS Spot Instance를 사용하고 있기 때문에, 더더욱 신경을 써야했던 포인트였습니다.

물론 위의 사례들을 제외하고 또 다른 자원 회수 요인들이 있겠지만 흔히 나오는 케이스 세 가지 정도만 간추려서 작성해보았고 나중에 또 기회가 된다면 다른 요인들도 추가해보겠습니다.

읽어주셔서 감사합니다.

## 참고문서
- [Linux Termination Signals](https://man7.org/linux/man-pages/man7/signal.7.html)
