---
layout: post
title: "Effective java 11장, 79절, Prefer executors, tasks, and streams to threads"
tags: [java]
comments: true
---

## ExecutorService
- JVM에서의 비동기적 작업을 관리해주는 기능을 제공한다.
- 왜 만들어졌을까
	- AS-IS: thread를 생성해서 작업을 처리한후, 처리가 완료되면 해당 Thread를 제거하는 작업을 진행해야 한다.
	- TO-BE: ExecutorService 에 쓰레드를 생성해서 전달하면 제거까지 알아서 해준다.
- ExecutorService 의 쓰레드풀보다 쓰레드의 갯수가 많다면 어떻게 처리될까?
	+ 내부의 Queue를 통해 순서대로 처리된다.

```java
ExecutorService exec = Executors.newSingleThreadExecutor();

// 쓰레드를 받아 실행한다.
exec.execute(runnable);

// 쓰레드가 모두 종료될때까지 기다린다음 종료한다.
exec.shutdown();
```

## 주요 메소드
```java
// 리턴 타입이 void로 Task의 실행 결과나 Task의 상태를 알 수 없다.
execute()

// Task를 할당하고 Future 타입의 결과값을 받는다. 결과 리턴이 되어야해서 Callable을 구현한 Task를 인자로 준다.
submit()

// Task를 Collection에 넣어서 인자로 넘겨준다. 실행에 성공한 Task 중 하나의 리턴값을 반환한다.
invokeAny()

// Task를 Collection에 넣어서 인자로 넘겨준다. 모든 Task의 리턴값을 List<Future<>>로 반환한다.
invokeAll()
```

## ForkJoinPool
- ForkJoinPool은 ExecutorService와 유사하나 병렬처리로 수행할 수 있다.
- ForkJoinPool가 조금 다른 부분은 Task의 크기에 따라 분할(Fork)하고, 분할된 Task가 처리되면 그것을 합쳐(Join) 반환한다.(Divide And Conquer) 알고리즘처럼 말이다.

## 참고자료
- Effective java 11장, 80절
- https://velog.io/@backtony/Java-ExecutorService%EC%99%80-ForkJoinPool#forkjoinpool


