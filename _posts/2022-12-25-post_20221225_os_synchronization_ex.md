---
title: 동기화 예제 (Synchronization Examples)
author: 펭덕
date: 2022-12-25 14:55:00 +0900
categories: [CS지식, OS]
tags: [공룡책, OS, Operating System, 운영체제, Computer, 컴퓨터, 동기화]
math: true
mermaid: true
image:
   src: https://user-images.githubusercontent.com/82709090/209460706-7dabafde-75c2-4843-8c67-183f5de00dd5.png
---

공룡책(10판) Chapter 7 요약

이전에는 Race condition에 대한 이야기와 해결 도구들을 살펴봤다면, 이번에는 동기화 문제를 직접 살펴보고 적용하는 단원이라고 하는 듯. 무슨 차이지..?


## 고전적인 동기화 문제들

동기화 문제에 대한 해결책을 제시할 때 전통적으로 세마포를 많이 이용해 왔다. 여기서도 세마포를 사용할 예정. 단 이진 세마포 대신에 mutex락도 사용 가능하다.

### - 유한 버퍼 문제 _ The Bouonded-Buffer Problem

생산자 - 소비자 문제를 유한 버퍼 문제로, n개의 버퍼가 있을 때 각각의 버퍼 하나는 한 항목을 저장할 수 있다. 생산자는 풀 버퍼를 만들고 소비자는 empty 버퍼를 만든다.

```c
int n;
semaphore mutex = 1; // 이진 세마포
semaphore empty = n; // 비어 있는 버퍼의 수
semaphore full = 0; // 꽉찬 버퍼의 수
```

> 코드 간의 대칭성에 주목한다.

```c
/* 생산자 프로세스 */
while (true) {
   // 다음 생산할 항목을 생산
   /* produce an item in next_produced */
   wait(empty);
   wait(mutex);

   // 버퍼에 다음 생산 항목을 추가
   /* add next _produced to the buffer */
   signal(mutex);
   signal(full);
}
```

```c
/* 소비자 프로세스 */
while (true) {
   wait(full);
   wait(mutex);
   // 다음 소비할 항목을 버퍼에서 제거
   /* remove an item from buffer to next_consumed */

   signal(mutex);
   signal(empty);
   // 다음 소비 항목을 소비
   /* consume the item in next_consumed */
}
```


### - Readers-Writers 문제

여러 개의 concurrent 프로세스들이 하나의 database 에 엑세스할 때, 어떤 프로세스는 읽기만 하고 어떤 프로세스는 갱신(읽기+쓰기) 한다. 전자를 `readers`, 후자를 `writers`라고 구별하자.

- 첫번째 readers-writers 문제 : writer가 기다리고 있다고 reader가 기다리면 안된다, writer가 기아 상태에 빠질 가능성
- 두번째 readers-writers 문제 : writer에게 우선순위를 준다. reader가 기아 상태에 빠질 가능성

이 문제에서 reader 프로세스는 다음과 같은 자료구조를 공유한다.

```c
semaphore rw_mutex = 1;
semaphore mutex = 1;
int read_count = 0; // reader의 개수가 0이면 writer가 진입할 수 있다.
```

writer : rw mutex를 획득하면 들어간다.

```c
/* Writer 프로세스 */
while(true) {
   wait (rw_mutex);

   /* 쓰기 작업이 이루어지는 곳 */

   signal(rw_mutex);
}
```

reader : read count를 사용하여 0일 때 rw_mutex를 깨운다.

```c
/* Reader 프로세스 */
while(true) {
   wait (mutex);
   read_count++;
   if (read_count == 1)
      wait(rw_mutex);
   signal(mutex);

   /* 읽기 작업이 이루어지는 곳 */

   wait(mutex);
   read_count--;
   if (read_count == 0)
      signal(rw_mutex);
   signal(mutex);
}
```

reader의 하나만 rw_mutex를, 나머지 reader (n-1 개)는 mutex를 가진다. reader, writer의 우선권은 스케줄러가 결정한다.

이 문제는 일반화 되어 있어 reader-writer 락이 제공되어 있다.


### - 식사하는 철학자들 문제

![table](https://user-images.githubusercontent.com/82709090/211191852-f574ed24-e795-416f-84d3-6500f9df9bc1.png)

5명의 철학자들이 생각하면서 먹으며 삶을 보낸다. 원형 테이블에 앉아 다섯 짝의 젓가락을 공유한다. 가끔씩 철학자는 배가 고파져서 자신 옆의 한 쌍의 젓가락을 집어 밥을 먹으려 하며, 먹는동안은 젓가락을 놓지 않는다. 

고전적인 동기화 문제로, 여러 종류의 리소스와 다른 종류의 프로세스끼리의 deadlock-free, starvation-free 문제이다.

#### * 세마포 해결안

철학자는 `wait()` 연산을 실행하여 젓가락을 집으려 하고, `signal()` 연산으로 젓가락을 놓는다. chopstick의 원소들은 1로 초기화된다.

```c
semaphore chopstick[5];
```

```c
while(true) {
   wait(chopstick[i]);
   wait(chopstick[(i+1) % 5]);

   /* 냠냠..(임계영역) */

   signal(chopstick[i]);
   signal(chopstick[(i+1) % 5]);

   /* 생각중.. */
}
```

이 해결안은 인접한 두 철학자가 동시에 식사하지 않는다는 것을 보장하나 데드락 상태를 야기할 가능성이 있다. 5명의 철학자가 모두 배가 고파 왼쪽 젓가락을 잡으면 오른쪽 젓가락은 모두 영원히 잡을수 없이 굶어 죽게된다.

이 해결안으로는

- 최대 4명의 철학자만이 테이블에 동시에 앉도록
- 한 철학자가 젓가락 두개를 잡을 수 있을 때만 잡도록 허용
- 비대칭 해결안 (홀수, 짝수 철학자는 각각 먼저 잡는 젓가락의 위치를 원칙으로 정한다.)

허나 이 해결안이 기아의 가능성도 제거하는 것은 아니다.

#### * 모니터 해결안

양쪽 젓가락 모두 얻을 수 있을 때만 집을 수 있다는 제한을 강제한다. 이를 위해 세가지 상태를 구분한다.

```c
enum {THINKING, HUNGRY, EATING} state[5];
```

- EATING ; 젓가락을 사용하고 있는 상태
- HUNGRY ; wait

철학자는 자신 양쪽의 이웃이 eating 상태가 아닐 때만 eating 상태로 바꿀 수 있다.

DiningPhilosopher 라는 모니터로 조절해보자. `pickup()`; 젓가락을 집어 식사하며, `putdown()`; 식사를 마친 후 호출한다.

```c
DiningPhilosophers.pickup(i);
/* 냠냠... */
DiningPhilosophers.putdown(i);
```

```c
monitor DiningPhilosophers
{
   enum {THINKING, HUNGRY, EATING} state[5];
   condition self[5]; // 자기 상태를 나타냄

   void pickup(int i) {
      state[i] = HUNGRY;
      test(i); // 배고플 때 양쪽 젓가락을 잡을 수 있는가?
      if (state[i] != EATING) // eating이 아니면 wait
         self[i].wait();
   }

   void putdown(int i) {
      state[i] = THINKING; // 젓가락 내려놓으면 생각한다.
      test((i + 4) % 5); // 오른쪽 젓가락
      test((i + 1) % 5); // 왼쪽 젓가락
   }

   void test(int i) {
      if ((state[(i + 4) % 5] != EATING) && (state[i] == HUNGRY) && (state[(i + 1) % 5] != EATING)) {
         state[i] = EATING;
         self[i].signal();
      } // 왼쪽과 오른쪽이 not eating 이고 내가 배고프면 먹는다, 먹는다는것을 알려준다.
   }

   initialization_code() {
      for (int i=0; i<5; i++)
         state[i] = THINKING; // 초기값은 생각
   }
}

```

이로써 이웃한 두 철학자가 동시에 식사하지 않고 교착 상태가 발생하지는 않으나, 여전히 철학자는 굶어죽을 수 있다. 기아 문제까지 해결하려면, 두 젓가락을 한번에 집어야 하고, 각각의 철학자가 젓가락을 들고 놓는 행위까지 세마포어로 관리를 해야 하는거 같다.

<br>

## 대체 방안들

- Thread-Safe Concurrent Applications : Concurrent app은 장점이 많지만 **race condition**과 **liveness hazard** 문제가 발생한다. 그래서 이에 대항하기 위해 사용
   1. Transaction Memory : 트랜잭션 메모리, atomic한 operation, 원자적인 실행의 단위이다. 모든 연산이 확정(`commit`)될 때까지 진행되며 그렇지 못하면 롤백(`rollback`).
   2. OpenMP : `#pragma omp parallel`, 병렬 구역을 지정
   3. Functional Programming Language : 함수형 프로그래밍을 사용. 명령어 기반의 프로그래밍을 하지 않는다. 함수형 언어는 변경 가능 상태를 허용하지 않기에 경쟁, 교착 등 쟁점에 신경쓸 필요가 없다.