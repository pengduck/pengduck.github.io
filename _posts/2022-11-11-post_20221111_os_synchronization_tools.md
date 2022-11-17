---
title: 동기화 도구들 (Synchronization Tools)
author: 펭덕
date: 2022-11-11 22:55:00 +0900
categories: [CS지식, OS]
tags: [공룡책, OS, Operating System, 운영체제, Computer, 컴퓨터, 동기화]
math: true
mermaid: true
image:
   src: https://user-images.githubusercontent.com/82709090/201459213-aac54b5b-1ae6-4ed8-85d2-867002b3b7db.png
---

공룡책(10판) Chapter 6 요약

여러 프로세스간의 경쟁 문제를 다루는 챕터..!

## 배경

- 협력적 프로세스(Cooperating process)
   - 시스템 내에서 서로 영향을 주거나 받는 프로세스
   - 논리적 주소를 공유 혹은은 데이터를 공유한다. 
   - 공유 데이터를 동시에 접근하면 데이터의 일관성을 망칠 수 있다.

데이터 무결성 확보 - 동시 데이터 접근 시 실행 순서를 보장해줘야 논리적 주소나 공유 데이터가 유지될 수 있다. 

생산자 소비자 문제에서 바라보았을때 버퍼를 공유하는 (shared memory, message queue) 비동기적 동작에서 버퍼 항목의 데이터의 무결성 문제가 대두될 수 있다.

`count++`, `count--` 을 생각해볼 때,

```
// count++
register1 = count
register1 = register + 1
count - register1

// count--
register2 = count
register2 = register2 -1
count = register2
```
각각은 기계어로 구현될 때 여러 statement가 되는데 이를 병행하게 실행하려면 각 3개의 절이 뒤섞인 채로 순차적 실행될 수 있기에 부정확한 상태에 도달할 수 있다.

### - 경쟁 상황 _ Race condition 

동시에 여러 개의 프로세스가 동일한 자료를 접근하여 조작하고, 실행 결과가 접근이 발생한 특정 순서에 의존하여 달라질 수 있는 상황이다.

이를 해결하기 위해서 협력적 프로세스 간의 **프로세스 동기화**, **조정** 등의 방법이 제시된다.

<br>

## 임계구역 문제 _ The Critical-Section Problem

하나의 프로세스가 임계 구역에 들어가는 경우 일종의 락(lock)을 걸어 나머지 프로세스가 거기에 들어올 수 없도록 하는 것으로, 프로세스들이 데이터를 협력적으로 공유하기 위해서 활동을 동기화할 때 사용하는 프로토콜을 설계하는 것이다.

동시에 두 프로세스를 같은 구역에서 실행하지 않는다. 각 프로세스는 자신의 임계구역으로 진입할 때 허가를 요청하여야 한다.

코드는 다음과 같은 구역들로 나뉜다.

- 진입 구역 (entry section) : 요청을 구현하는 부분
- 임계 구역 (critical section)
- 퇴출 구역 (exit section) : 임계구역에서 나오는 부분
- 나머지 구역 (remainder section) : 나머지 부분

임계구역 문제에 대한 해결안은 다음 세 가지 요구 조건을 충족해야 한다.

1. 상호 배제 (mutual exclusion) : 프로세스가 자기의 임계구역에서 실행된다면 다른 프로세스들은 임계구역에 진입할 수 없다.
2. 진행 (progress) : 임계구역에서 실행중인 프로세스가 없다면 다른 프로세스가 들어갈 수 있어야 한다. - deadlock (임계구역에 아무도 못들어가는 현상) 회피
3. 한정된 대기 (bounded waiting) : 프로세스가 임계구역에 들어가기를 무한히 대기해서는 안된다, 즉 진입 허용 횟수에 제한을 두어야 한다. - starvation(새치기로 인하여 어떤 프로세스가 못들어가는 현상) 회피


단일 코어에서는 임계구역 문제를 해결할 방법으로 가장 간단한 것은 인터럽트 발생을 막는 것이지만 다중 처리기 환경에서는 이는 불가하다.

다중 코어 시스템에서의 접근법으로 두가지

- 비선점형 커널 : 커널모드에 한번 진입하고 나면 cpu를 계속 쓰게 해주기에 context switch가 없어 경쟁상태가 발생할 여지가 없다. 허나 비효율적이라 성능이 느려 사용하지 않음.
- 선점형 커널 : 프로세스가 언제든 선점될 수 있기 때문에 동기화 문제가 반드시 발생, 다루기 어렵다. 허나 대기 중인 프로세스에 CPU를 양도하기 전에 오랫동안 실행될 위험이 적어서 응답이 민첩할 수 있다. 

<br>

## Peterson의 해결안 _ Petetson's Solution

while 문으로 무한루프 돌고 있으면 대기 중인 것이다. 여러가지 알고리즘들을 간단히 살펴보자.

- Dekker의 알고리즘
아래는 Pi 프로세스의 경우이다. `bool flag[2]`와 `int turn`의 공유 변수를 가진다. 
```C
while(1) {
   ...
   flag[i] = true;      // 임계영역 진입시도
   while (flag[j]) {    // Pj가 임계영역에 있는지 확인
      if(turn == j) {   // 있다면
         flag[i] = false;  // 진입 취소하고
         while (turn == j) {  // 대기한다
            flag[i] = true;   // 재진입 시도
         }
      }
   }
   // 임계영역(Critical section)

   turn = j;         // 진입순서양보
   flag[i] = false;  // 퇴출
}

```

- Eisenberg and McGuire의 알고리즘

n개의 프로세스에 대한 cs문제의 해결 방안으로 제시되었다.

```C
do {
   while(1) {
      flag[i] = want_in;
      j = turn;
      while (j != I) {
         if(flag[j] != idle)
            j = turn;
         else
            j= (j+1) % n;
      }
      flag[i] = in_cs;
      j=0;
      while ((j<n) && (j == I || flag[j] != in_cs))
         j++;
      if((j>=n) && (turn == I || flag[turn] == idle)) break;
   }
   turn = i;

   // critical section

   j = (turn + 1) % n;
   while (flag[j] == idle)
      j = (j+1) % n;
   turn = j;
   flag[i] = idle;

   // remainder section
} while(1);

```
wait 시간이 한없이 길어지지 않는다는데.. 자세히 봐야겠다. turn 변수는 0 또는 1이기에 둘중 하나만이 `while ((j<n)...` 문을 통과할 수 있을 것이다.

> 복잡해보인다...!

- Bakery Ramport 알고리즘

빵집에서 번호표를 나누어주고 차례로 빵을 파는데에서 유래했다고 한다.

```C
while(1) {
   ...
   choosing[i] = true; // 번호표 받을 준비

   number[i] = max(number[0], number[1], ..., number[n-1]) + 1;
   // 다음 번호 할당
   choosing[i] = false; // 번호표 받았다
   for (j=0; j<n; j++) {   // 모든 프로세스 번호표 비교
      while (choosing[j]); // Pj가 번호표 받을 때까지 대기
         while (number[j] && (number[j], j) < (number[i], i));
         // Pj가 번호표 가지고 있고
         // Pj 번호표가 Pi번호표보다 작거나 
         // 또는 번호표가 같을 경우,
         // j가 i보다 작다면 
         // Pj의 종료(number[j]=0)까지 대기
   }
   // CS

   number[i] = 0; // 사용완료
}
```

- 피터슨 알고리즘

본론이다. 복잡한줄 알았는데 이해하는데는 역시 이게 그나마 간단해 보인다. 다만 현대 컴퓨터 구조가 `load`와 `store` 같은 기본적인 기계어를 수행하는 방식이라 이 알고리즘이 현대 컴퓨터 구조에서 바르게 실행된다 보장할 수는 없다고 한다.

두 프로세스가 두개의 데이터 항목을 공유한다.
```C
int turn;
boolean flag[2];
```

turn은 다음에 실행될 프로세스를 가리키고 하나가 true면 다른 프로세스가 실행되지 못하도록 제한한다.
```C
while (true) {
   flag[i] = true;   // 임계 구역 진입을 위해 flag 수정
   turn = j;   // 다음 차례를 j에게 넘김
   while (flag[j] && turn == j)  // Pj의 실행 대기
      ;  // Pj의 플래그가 false이거나 현재 턴이 i인 경우 대기 해제, 임계영역에 진입한다.

      /* 임계영역 */
   
   flag[i] = false;  // 임계영역 탈출

      /*  나머지 영역 */
}
```

<br>

## 동기화를 위한 하드웨어 지원 _ Hardware Support for Synchronization

위에서 소프트웨어 기반 해결책을 이야기했지만, 역시 문제는 하드웨어 지원이 없으면 어렵다. 기계어 레벨에서의 원자성 보존이 잘 되지 않는다. 

### - 메모리 장벽 _ Memory Barriers

앞서서 시스템 statement의 순서가 재정렬 될 수 있다는 것을 확인했다. 컴퓨터 아키텍처는 application 에게 메모리 접근 방식을 제공하는데, 이를 메모리 모델이라 한다.

메모리의 모든 변경 사항을 모든 프로세서로 전파하는 명령어를 제공하여, 다른 프로세서에서 실행중인 스레드에 메모리 변경 사항이 보이게 보장한다. 이런 명령어를 메모리 장벽(Memory Barriers), 메모리 펜스(Memory fences)라고 한다. 메모리 장벽 명령어가 실행될 때, 시스템은 후속 적재, 저장 연산이 수행되기 전에 모든 적재 및 저장이 완료되도록 한다.

### - 하드웨어 명령어 _ Hardware instructions

modify, swap을 하드웨어적으로 인터럽트 되지 않는 하나의 단위로 명령어를 제공한다.

- `test_and_set()` : while문으로 조건을 test and set 을 사용하여 false로 초기화되는 boolean 변수를 선언하여 상호 배제를 이룰 수 있다.

```C++
boolean test_and_set(boolean *target) {
   boolean rv = *target;
   *target = true;

   return rv;
}
```

> 메서드가 원자적으로 실행된다.


- `compare_and_swap()` (CAS) : value를 swap 해준다.integer lock 을 swap, atomic 하게 상호 배제

```C++
int compare_and_swap(int *value, int expected, int new_value) {
   int temp = *value;
   if(*value == expected) *value = new_value;

   return temp;
}
```

### - 원자적 변수 _ Atomic Variables

원자적 동작이라고 하는 것은 인터럽트를 할 수 없는 오퍼레이션의 단위이다. (cpu 1 clk로 할 수 있도록)

`compare_and_swap()`은 상호 배제를 위해 직접 사용되기보다 원자적 변수를 구축하기 위해 사용된다. 정수나 부울 같은 기본 데이터 유형에 대한 원자적 연산을 제공한다. 원자적 갱신을 제공하지만 모든 상황에서 경쟁 조건을 완벽히 해결해주지는 않는다.

> 유한 버퍼 문제에서 조건이 count값에 따라 달라지는 while 루프에서, `(count > 0)` 생산자가 버퍼에 원소를 넣으면 단지 1로 세트되었는데도 두 소비자가 while루프를 빠져나오는 경우가 생긴다.

대개 카운터나 시퀀스 생성기와 같은 공유 데이터 한 에만 제한되어 사용되는 경우가 많다.



```JAVA
public class Peterson1 {
   static int count = 0;
   static int turn = 0;
   static boolean[] flag = new boolean[2];

   public static void main(String[] args) throws Exception {
      Thread t1 = new Thread(new Producer());
      Thread t1 = new Thread(new Consumer());
      t1.start(); t2.start();
      t1.join(); t2.join();
      System.out.println(peterson1.count);

   }

   static class Producer implements Runnable {
      @Override
      public void run() {
         for (int k=0; k<10000; k++) {
            /* entry section */
            flag[0] = true;
            turn = 1;
            while (flag[1] && turn == 1)
               ;
            
            /* critical section */
            count++;

            /* exit section */
            flag[0] = false;

            /* remainder section */
         }
      }
   }

   static class Consumer implements Runnable {
      @Override
      public void run() {
         for (int k=0; k<10000; k++) {
            /* entry section */
            flag[1] = true;
            turn = 0;
            while (flag[0] && turn == 0)
               ;
            
            /* critical section */
            count--;

            /* exit section */
            flag[1] = false;

            /* remainder section */
         }
      }
   }
}
```

