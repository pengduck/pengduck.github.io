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

#### Dekker의 알고리즘

아래는 Pi 프로세스의 경우이다. `bool flag[2]`와 `int turn`의 공유 변수를 가진다. 

```c
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

#### Eisenberg and McGuire의 알고리즘

n개의 프로세스에 대한 cs문제의 해결 방안으로 제시되었다.

```c
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

#### Bakery Ramport 알고리즘

빵집에서 번호표를 나누어주고 차례로 빵을 파는데에서 유래했다고 한다.

```c
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

#### 피터슨 알고리즘

본론이다. 복잡한줄 알았는데 이해하는데는 역시 이게 그나마 간단해 보인다. 다만 현대 컴퓨터 구조가 `load`와 `store` 같은 기본적인 기계어를 수행하는 방식이라 이 알고리즘이 현대 컴퓨터 구조에서 바르게 실행된다 보장할 수는 없다고 한다.

두 프로세스가 두개의 데이터 항목을 공유한다.

```c
int turn;
boolean flag[2];
```

`turn`은 다음에 실행될 프로세스를 가리키고 하나가 true면 다른 프로세스가 실행되지 못하도록 제한한다.

```c
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

```c
boolean test_and_set(boolean *target) {
   boolean rv = *target;
   *target = true;

   return rv;
}
```

> 메서드가 원자적으로 실행된다.


- `compare_and_swap()` (CAS) : value를 swap 해준다.integer lock 을 swap, atomic 하게 상호 배제

```c
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


## Mutex Locks

csp 해결을 위한 고급 레벨의 소프트웨어 툴 

- 뮤텍스 락 : 동기화를 위한 가장 간단한 도구 2개 제어
- 세마포어 : n개를 제어할 수 있어서 매우 편리
- 모니터 : 뮤텍스와 세마포어의 단점을 극복 -> Java에서 사용 (wait, notify)
- Liveness : 프로그레스, 데드락 문제 해결

> Mutex : **mut**ual **ex**clusion 상호 배제

임계 영역을 보호하고 경쟁 상태, 공용 자원 엑세스를 방지한다. lock을 사용한다. cs에 진입하기 위해 lock을 획득하는 과정과 exit할 때 release (반납)하는 과정이 있다.

락을 획득하고 반환하는 `acquire()`, `release()` 두개의 함수 제공, available이라는 boolean variable 이용해서 lock을 결정

```c
acquire() {
   while (!available)
      ; /* busy wait */
   avavilable = false;
}

release() {
   available = true;
}
```

이 두 과정도 원자적으로 실행되어야 한다. 

하지만 `바쁜 대기 (Busy waiting)` 문제가 여전히 존재; 임계구역 들어가기 전에 `acquire()` 반복문의 무한 루프를 도는 현상, cpu 자원을 계속 먹는다.
 `스핀락(spinlock)`은 busy wait를 하는 락 유형이다. CPU 코어가 여러개인 경우 문맥 교환이 일어나지 않기에 해당 시간을 절약할 수 있다.

<br>

## 세마포 Semaphores

신호장치, 신호기.

- binary semaphore : 0하고 1만 왔다갔다, 뮤텍스 락 하고 같다.
- Counting semaphore : 무한대로 늘어날 수 있다. 여러개의 인스턴스를 가진 자원

함수는 `wait()`, `signal()` 혹은 `P()`, `V()`

Proberen(to test) Verhogen(to increment)

```c
wait(S) {
   while (S <= 0)
      ; // busy wait
   S--;
}

signal(S) {
   S++;
}
```

이것도 둘다 atomic 하게 구현되어야 함. 정수를 사용하여 n개의 인스턴스를 가진 자원을 써먹을 수 있다.

resource가 available한 개수로 초기화를 한다. 리소스를 사용하려면 wait()를 사용 count를 감소, 리소스를 다 썼으면 signal()을 사용 count를 증가

> count가 0인 상태에서는 모든 리소스가 사용중이라는 것

이것도 busy waiting 문제가 있다. 이 문제를 해결하려면 wait()를 할 때 while문을 돌지 말고 waiting queue에 들어가면 된다.

<br>

## 모니터 _ Monitors

뮤텍스와 세마포어가 편리하지만 *타이밍 오류*가 자주 발생한다. 특정한 시퀀스를 잘못 쓰면 항상 일어나지도 않고 잡기 어려운 문제가 발생한다. 가령 연산의 순서가 바뀔 경우 상호 배제가 지켜지지 않는 현상이 발생할 수 있는데, 이런 상황이 언제나 재현 가능하지도 않다.

문제를 낮출 방법. 심플한 동기화 툴을 사용하자. 모니터는 에러가 잘 발생하지 않는 기법이다.


### 모니터 사용법 _ Usage

추상화된 데이터 형 `ADT (abstract data type)` 은 데이터와 데이터 조작 함수 집합을 하나의 단위로 묶어 보호한다. 즉 상호 배제 자체의 틀을 제공하며, 
variable을 선언하고 거기에 define된 인스턴스를 호출하도록 한다. 자체적으로 하면 약간 부족해서, conditional variable을 도입한다.

```java
condition x, y;
x.wait(); // 을 호출 하면
x.signal(); // 다른 프로세스가 이것을 호출할때까지 일시중지된다.
y.wait();
y.signal();
```

각각의 conditional 변수에 호출될 수 있는 연산은 wait()와 signal()이다. wait() 를 호출한 프로세스는 다른 프로세스가 signal()을 호출할 때까지 기다린다.


### Java Monitors

자바에서는 monitor-lock 혹은 intrinsic-lock을 제공한다. 기본 단위가 thread이므로 thread synchronization을 구현

- synchronized keyword : 임계영역에 해당하는 코드 블록을 선언할 때 사용하는 자바 키워드

해당 코드 블록(임계영역)에는 모니터락을 획득해야 진입 가능하며, 모니터락을 가진 객체 인스턴스를 지정할 수 있다. 메소드에 선언하면 메소드 코드 블록 저체가 임계영역으로 지정된다.

 - 이 때, 모니터락을 가진 객체 인스턴스는 this 객체 인스턴스임

```java
synchronized (object) { // 이 때 object는 모니터락
   //critical section
} // 임계영역만 정해주면 나머지 엔트리 섹션 등은 자바가 알아서 해주겠다.

public synchronized void add() {
   //critical section
} // 메소드를 호출할 때는 this객체의 모니터락을 획득하고 진입했다가 나올 때 반납하면 된다.
```

`wait()` `notify()` 메소드
- java.lang.Object 클래스에 선언됨 : 모든 자바 객체가 가진 메소드임
- wait:wait, notify:signal
- 스레드가 어떤 객체의 wait() 메소드를 호출하면 해당 객체의 모니터락을 획득하기 위해 대기 상태로 진입함.
- 스레드가 어떤 객체의 notify() 메소드를 호출하면 해당 객체 모니터에 대기중인 스레드 하나를 깨움.
- notify() 대힌에 notifyAll() 메소드를 호출하면 해당 객체 모니터에 대기중인 스레드 전부를 깨움 -> 모두가 레디 큐에 들어갔다가 하나만 실행, 나머지는 다시 대기

```java
public class SynchExample1 {
   static class Counter {
      public static int count = 0;
      public static void increment() {
         count++;
      }
   }

   static class MyRunnable implements Runnable {
      @Override
      public void run() {
         for (int i=0; i<10000; i++)
            Counter.increment();
      }
   }

   public static void main(String[] args) throws Exception {
      Thread[] threads = new Thread[5];
      for (int i=0; i<threads.length; i++) {
         threads[i] = new Thread(new MyRunnable());
         threads[i].start();
      }
      for (int i=0; i<threads.length; i++) {
         threads[i].join();
      }
      System.out.println("counter = " + Counter.count);
   }
}
```

race condition이 발생해 50000근처에도 못간다.


```java
public class SynchExample2 {
   static class Counter {
      public static int count = 0;
      synchronized public static void increment() {
         count++;
      } // synchronized 만 선언해도 임계영역이 된다.
   }

   static class MyRunnable implements Runnable {
      @Override
      public void run() {
         for (int i=0; i<10000; i++)
            Counter.increment();
      }
   }

   public static void main(String[] args) throws Exception {
      Thread[] threads = new Thread[5];
      for (int i=0; i<threads.length; i++) {
         threads[i] = new Thread(new MyRunnable());
         threads[i].start();
      }
      for (int i=0; i<threads.length; i++) {
         threads[i].join();
      }
      System.out.println("counter = " + Counter.count);
   }
}
```

아주 잘된다. 편하긴 하지만 허나 남발하면 임계영역 구간이 길어지며 동기화되면 멀티스레딩의 장점이 사라진다.

```java
   static class Counter {
      private static Object object = new Object();
      public static int count = 0;
      public static void increment() {
         synchronized(object) {
            count++;
         } // 멍따 object 선언해서 락을 주자.
      }
   }
```

```java
public class SynchExample4 {
   static class Counter {
      public static int count = 0; // 공유
      public void increment() {
         synchronized (this) {
            Counter.count++;
         } // 자기 객체 인스턴스의 monitor lock을 획득해서 증가시켜라
      }
   }

   static class MyRunnable implements Runnable {
      Counter counter;
      public MyRunnable(Counter counter) {
         // 생성자
         this.counter = counter;
      }
      @Override
      public void run() {
         for (int i=0; i<10000; i++)
            counter.increment();
      }
   }

   public static void main(String[] args) throws Exception {
      Thread[] threads = new Thread[5];
      for (int i=0; i<threads.length; i++) {
         threads[i] = new Thread(new MyRunnable(new Counter())); // 카운터 인스턴스 선언 (5개, static int count를 공유)
         threads[i].start();
      }
      for (int i=0; i<threads.length; i++) {
         threads[i].join();
      }
      System.out.println("counter = " + Counter.count);
   }
}
```
또 동기화 문제 발생 : this가 각각의 인스턴스라서 그렇다 (5개) : lock이 달라서, counter가 새로 생성되어서

> 해결책

```Java
   public static void main(String[] args) throws Exception {
      Thread[] threads = new Thread[5];
      Counter counter = new Counter();
      for (int i=0; i<threads.length; i++) {
         threads[i] = new Thread(new MyRunnable(counter)); // 카운터 인스턴스 하나로 다섯개 스레드가 돈다 this (lock)가 하나가 된다.
         threads[i].start();
      }
      for (int i=0; i<threads.length; i++) {
         threads[i].join();
      }
      System.out.println("counter = " + Counter.count);
   }
```

<br>

## 라이브니스 _ Liveness

mutex, semaphore 등은 상호 배제만 보장하지 deadlock과 starvation 문제를 해결해주진 못한다. 라이브니스는 프로세스 실행 수명주기동안 진행을 보장하기 위해 시스템이 충족해야 하는 속성을 뜻한다. 일종의 약속인듯

라이브니스 실패로 이어질 수 있는 상황은 다음과 같다.

- 교착 상태 (Deadlock) :두개 이상의 프로세스가 영원히 기다리는 상태로, 대기 중인 프로세스 중 하나에 의해서만 야기될 수 있는 이벤트를 무한정 기다린다.

- 우선순위 역전 (Priority Inversion) : 우선순위가 높은 프로세스가 낮은 프로세스에게 밀리는 현상 : 커널 데이터는 락에 의해 보호되기 때문에, 낮은 우선순위 프로세스가 자원 사용을 마칠 때 까지 높은 프로시스가 기다리게 된다. 우선순위 상속 프로토콜 (priority-inheritance protocol) 을 구현하여 해결한다.