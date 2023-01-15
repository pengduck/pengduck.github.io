---
title: 교착 상태 (Deadlocks)
author: 펭덕
date: 2023-01-15 18:45:00 +0900
categories: [CS지식, OS]
tags: [공룡책, OS, Operating System, 운영체제, 데드락, Deadlock, 교착상태]
math: true
mermaid: true
image:
   src: https://user-images.githubusercontent.com/82709090/212537984-2eb37a55-4e83-416b-bbf6-29f6e91b19d3.png
---

공룡책(10판) Chapter 8 요약

까다롭다..! 대기 중인 스레드들이 요청한 자원들이 다른 스레드에 의해 점유되어 있고 그 다른 스레드들도 대기 상태에 있어서 상태 변경이 이루어지지 않을 때, 그것을 교착 상태, 데드락이라 한다.

<br>

## 시스템 모델 _ System model

여러 유형의 자원들이 스레드들에 의해 공유된다. 프로세스는 다음 순서로 자원을 사용한다.

> Request(요청) - Use(사용) - Release(방출)

한 스레드 집합 내의 모든 스레드가 다른 스레드에 의해서만 발생될 수 있는 이벤트를 기다릴 때, 그 스레드 집합은 교착 상태에 있다.

<br>

## 다중 스레드 응용에서의 교착 상태 _ Deadlock in Multithreaded Applications

다음은 교착 상태에 대한 예시 코드이다. 

```c
/* 초기화 */
pthread_mutex_t first_mutex;
pthread_mutex_t second_mutex;

pthread_mutex_init(&first_mutex,NULL);
pthread_mutex_init(&second_mutex,NULL);

/* thread one runs in this function */
void *do work one(void *param)
{
   pthread mutex lock(&first mutex);
   pthread mutex lock(&second mutex);
   /**
   * Do some work
   */
   pthread mutex unlock(&second mutex);
   pthread mutex unlock(&first mutex);
   pthread exit(0);
}

/* thread two runs in this function */
void *do work two(void *param)
{
   pthread mutex lock(&second mutex);
   pthread mutex lock(&first mutex);
   /**
   * Do some work
   */
   pthread mutex unlock(&first mutex);
   pthread mutex unlock(&second mutex);
   pthread exit(0);
}
```

thread_one은 first, second 순서로 mutex락을 획득하려고 하고 동시에 thread_two는 second, first 순서로 획득하려고 한다. one이 first, two가 second를 획득하였다면 교착 상태가 가능하다. 물론 락 획득 전에 뮤텍스를 획득, 방출할 수 있으면 교착 상태는 발생하지 않는다.

<br>

## 교착 상태 특성 _ Deadlock Characterization

데드락의 필요조건이다.

1. 상호 배제 (Mutual Exclusion) : 최소한 하나의 자원이 비공유 모드로 점유되어야 한다
2. 점유 대기 (hold-and-wait) : 스레드가 자원을 점유한 채로 대기하여야 한다.
3. 선점 불가, 비선점 (No preemption) : 자원을 선점할 수 없어야 한다.
4. 순환 대기 (Circular wait) : 원형 대기, 서로 연쇄적인 대기상태

4개를 다 만족해야하기 때믄에 재현이 어렵다.

### 자원 할당 그래프 _ Resource-Allocation Graph

데드락을 쉽게 이해하기 위한 그래프. 사이클이 존재하는지를 확인하면 된다.  Vertix 집합 V와 Edge 집합 E로 구성되며, V는 활성 스레드 T = {T1, T2, ... Tn} 와 모든 자원 유형의 집합인 R = {R1, R2, ... Rn} 으로 구별된다. T가 R을 요청하면 request edge 이고 역 (R에서 T)은 assignment edge 이다.

![graph1](https://user-images.githubusercontent.com/82709090/212538012-e7c54d2a-c8d1-4523-9713-c70f08ff4157.png)

이렇게 싸이클을 타면 교착 상태라 할 수 있다.

![graph2](https://user-images.githubusercontent.com/82709090/212538020-0662bb01-2811-4144-975e-cb6d489fc592.png)

이는 사이클이 없기에 교착 상태가 아니다...!

![graph3](https://user-images.githubusercontent.com/82709090/212538031-db7621f2-78ec-4acb-9954-72ca1c913d4a.png)

두개의 사이클! 확실한 교착 상태, 데드락이다.

![graph4](https://user-images.githubusercontent.com/82709090/212538045-397526a1-5ec4-4db1-b51e-d52e8052ea35.png)

사이클이 있지만 위처럼 인스턴스가 여러개면 데드락이 발생하지 않을 수 있다.

<br>

## 교착 상태 처리 방법 _ Methods for Handling Deadlocks

1. 데드락이 나던 말던 무시; 자주 발생하는게 아니니까...
2. 데드락 예방 프로토콜, 데드락 회피; 뱅커의 알고리즘
3. 데드락 허용 후 복구; 1번과 같은 이유, 한번씩 리부팅 해주자.

<br>

## 교착 상태 예방 _ Deadlock Prevention

교착 상태의 필요조건 중 최소 하나를 성립하지 않도록 막는 방법으로 교착 상태 발생을 예방할 수 있다.

1. 상호 배제
   - 최소 하나의 자원을 공유 불가능하게 한다. -> 불가능하다. 고려하지말자.
2.  점유하며 대기
   - 스레드가 새로운 리소스를 요청할 때는 다른 자원을 보유하지 않도록 보장한다. 실용적이지 않다.
3. 선점불가, 비선점
   - 이미 할당된 자원이 선점되지 않아야 한다. 뺏기면 문제가 되기 때문에 일반적으로는 사용하기 어렵다
4. 순환 대기
   - 그나마 유용. 자원 유형에 순서를 부여하여 특정 순서로만 자원을 요청할 수 있도록 한다. 그러면 순환 대기는 예방할 수 있다. 허나 허나 구조를 정하는것 만으로는 예방할 수 없는것이, 프로그램 작성은 응용 프로그래머에게 달려 있기 때문이다. 학의 개수도 수백, 수천개가 될 경우도 어렵다. 또한 락이 동적으로 획득 가능하다면 순서 부여로 예방할 수는 없다.

결론 : 1~3은 사실상 불가하며 4도 뭔가 변변치 않다는 것

<br>

## 교착 상태 회피 _ Deadlock Avoidance
