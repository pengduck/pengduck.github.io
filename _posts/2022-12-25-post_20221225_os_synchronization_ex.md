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

### 유한 버퍼 문제 _ The Bouonded-Buffer Problem

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


### Readers-Writers 문제

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


### 식사하는 철학자들 문제

...

<br>

## 커널 안에서의 동기화

다음 시간에?