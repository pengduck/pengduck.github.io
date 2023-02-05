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

위 4가지 중 하나를 막는 방법은 신통치 않고 사이드 이펙트도 많고 장치 이용률도 떨어지고 시스템 처리율도 감소시킨다. 그래서 회피할 방법을 강구해보자.

회피하는 대안으로는 자원이 어떻게 요청될지에 대한 정보를 제공하도록 요구하는 것이다. 각 스레드의 요청과 방출에 대한 완전한 순서를 파악한다면 가능한 미래의 교착 상태를 피하고자 스레드의 대기 여부를 결정할 수 있다. 

이 방법을 사용하는 가장 단순한 모델은 각 스레드가 자신이 필요로 하는 각 유형의 자원마다 최대 수를 선언하도록 요구하는 것이다. 이를 통해 시스템이 교착 상태에 들어가지 않을 것을 보장하는 알고리즘을 만들 수 있다.

### 안전 상태

시스템 상태가 `안전(safe)`하다 = 시스템이 어떤 순서로든 스레드가 요청하는 모든 자원을 교착 상태를 야기시키지 않고 차례로 할당해 줄 수 있다. 즉 시스템이 `안전 순서(safe sequence)`를 찾을 수 있으면 시스템은 안전하다고 말할 수 있다. 아닐 경우 `불안전(unsafe)`하다고 한다.

단 불안전하다고 반드시 교착 상태라는 것은 아니다. 교착 상태로 갈 수도 있다는 것을 의미한다.

안전이라는 개념을 통해 회피 알고리즘이 교착 상태를 어떻게 회피할 지 정의할 수 있으며, 기본 원칙은 **시스템이 항상 안전상태에 있도록 고수**하는 것이다. 하지만, 이 방법은 프로세스나 스레드가 요청한 자원보다 많은 양을 시스템이 보유하더라도 프로세스를 기다리게 하는 상황이 발생할 수 있어, 자원의 이용률이 낮아질 수 있다.

### 자원 할당 그래프 알고리즘

자원 할당 그래프를 그려보며 사이클이 생기지 않도록 하여 데드락을 회피한다. 자원 할당 그래프에서 요청 간선과 할당 간선에 추가로 `예약 간선 (claim edge)` 이라는 점선을 추가한다.

![graph5](https://user-images.githubusercontent.com/82709090/215323743-f67aa80b-0d33-44e5-8d91-d12cb0527b34.png)

스레드 T<sub>i</sub> 가 실행되기 전에 스레드의 모든 예약 간선이 자원 할당 그래프에 표시되어야 하며, 스레드 스레드 T<sub>i</sub>와 연관된 모든 간선들이 예약 간선일 때만 예약 간선 T<sub>i</sub> -> R<sub>j</sub> 를 그래프에 추가하도록 허용하게 한다.

스레드 T<sub>i</sub>가 R<sub>j</sub>를 요청할 때, 요청 간선 T<sub>i</sub> -> R<sub>j</sub>가 할당 간선 R<sub>j</sub> -> T<sub>i</sub>로 변환해도 자원 할당 그래프에 사이클을 형성하지 않을 때만 요청을 허용할 수 있다.

이 그래프에서 사이클을 탐지하는 알고리즘은 n<sup>2</sup> 차수의 연산이 필요하다. (n은 스레드의 수)

### 은행원 알고리즘

자원 할당 그래프 알고리즘은 종류마다 자원이 여러개면 사용할 수 없다. 그래서 이럴 때는 은행원 알고리즘을 사용한다.

스레드가 시작할 때 스레드가 가지고 있어야 할 자원의 최대 개수를 자원 종류마다 미리 신고한다. 스레드가 자원을 요청하면 시스템은 그것을 들어주었을 때 시스템이 계속 안전 상태에 머무르게 되는지 여부를 판단해야 한다.

은행원 알고리즘을 구현하려면 몇 가지 자료구조가 필요한데, 여기서 n은 스레드 수, m이 자원 종류 수이다.

- Available : 각 종류별로 가용한 자원의 개수를 나타내는 크기 m의 벡터
- Max : 각 스레드가 최대로 필요로 하는 자원의 개수를 나타내는 n * m 행렬
- Allocation : 각 스레드에 현재 할당된 자원의 개수를 나타내는 n * m 행렬
- Need : Max[i][j] - Allocation[i][j] 의 관계가 있다.


> 행렬이 어쩌구..뭔소린지 모르겠으니 예제로 알아보자.

- T = {T<sub>0</sub>, T<sub>1</sub>, T<sub>2</sub>, T<sub>3</sub>, T<sub>4</sub>}
- R = {A, B, C}
- 인스턴스의 수 A = 10 B = 5 C = 7 일 때, 아래 스냅샷을 확인하자.

```
    Allocation     Max     Available
    --------------------------------
      A  B  C    A  B  C    A  B  C
T0    0  1  0    7  5  3    3  3  2
T1    2  0  0    3  2  2
T2    3  0  2    9  0  2
T3    2  1  1    2  2  2
T4    0  0  2    4  3  3
```

Need 행렬의 값은 (Max-Allocation)으로부터 얻어진다.

```
       Need  
    -----------
      A  B  C 
T0    7  4  3 
T1    1  2  2 
T2    6  0  0 
T3    0  1  1
T4    4  3  1
```

#### 안전성 알고리즘

1. Work 벡터를 초기화 = Available 값 = (3, 3, 2), 그리고 Finish 벡터 값 T<sub>0</sub>~T<sub>5</sub> 를 모두 false로 초기화
2. Finish[i]가 false인 놈을 찾았을 때 Need<sub>i</sub> <= Work 이면
3. Work = Work + Allocation<sub>i</sub> 로 된다.
그리고 Finish[i] = true
4. 모든 i에 대해 Finish[i] = true이면 `안전 상태`라고 할 수 있다!

```
     Need       Work     finish   Work + Alloc = Work  
i=0 (7,4,3) <= (3,3,2)   false
i=1 (1,2,2) <= (3,3,2)   true    3,3,2 + 2,0,0 = 5,3,2
i=2 (6,0,0) <= (5,3,2)   false
i=3 (0,1,1) <= (5,3,2)   true    5,3,2 + 2,1,1 = 7,4,3
i=4 (4,3,1) <= (7,4,3)   true    7,4,3 + 0,0,2 = 7,4,5
다시
i=0 (7,4,3) <= (7,4,5)   true    7,4,5 + 0,1,0 = 7,5,5
i=2 (6,0,0) <= (7,5,5)   true    7,5,5 + 3,0,2 = 10,5,7
```

이렇게 하면 모두 true가 된다.!

T1, T3, T4, T0, T2 의 시퀀스가 안전성 알고리즘을 만족한다. T1, T3, T4, T2, T0도 만족

이런 시퀀스 하나라도 존재하면 안전하다.

#### 자원 요청 알고리즘

여기서 새로운 request (1, 0, 2)가 들어왔다고 했을 때,
grant 할것이냐 말것이냐를 판단하려면 `자원요청 알고리즘`을 사용한다.

1. Request<sub>i</sub> <= Need<sub>i</sub> 인가 확인
2. Request<sub>i</sub><= Available인가 확인

둘 다 만족하면 

3. - Available = Available - Request<sub>i</sub>
   - Allocation<sub>i</sub> = Allocation<sub>i</sub> + Request<sub>i</sub>
   - Need<sub>i</sub> = Need<sub>i</sub> - Request<sub>i</sub>

이렇게 바꿨을 때 안전하다면 자원할당, 아니면 Ti는 만족할때까지 대기한다.

가령 T1이 (1, 0, 2) 요청을 받는다면
1. (1, 0, 2) <= (1, 2, 2) 만족
2. (1, 0, 2) <= (3, 3, 2) 만족
3. - 2, 0, 0 + 1, 0, 2 = 3, 0, 2 ==> T1의 Allocation
   - 1, 2, 2 - 1, 0, 2 = 0, 2, 0 ==> T1의 Need
   - 3, 3, 2 - 1, 0, 2 = 2, 3, 0 ==> Available

이렇게 치환하고 나서 다시 안전성 알고리즘을 돌린다.

13402가 만족한다고 한다.

<br>

## 교착 상태 탐지 _ Deadlock Detection

데드락 발생토록 허용해두고 발생하는 것을 체크해준다.

![graph6](https://user-images.githubusercontent.com/82709090/215324843-71433a03-ff00-4a55-a9e5-61f3e53e0f66.png)

> 대기 그래프 (wait for Graph)

- 자원 유형이 하나만 있는 경우 : `wait-for graph` 스레드의 의존성 확인하는 그래프; 자원 유형의 노드를 제거하고 간선을 결합하여 대기 그래프를 얻는다.
- 여러개 있는 경우 : 은행원 알고리즘과 비슷; Available, Allocation, Request를 사용
- 단 Allocation이 0 일 경우 finish[i] = true로 놓는다.

- 탐지 알고리즘의 사용
  1. 데드락이 얼마나 자주 일어나는가?
  2. 데드락에 몇 개 스레드가 연루되는가?

## 교착 상태로부터 회복 _ Recovery from Deadlock

- 프로세스나 스레드 종료
  1. 프로세스 모두 중지
  2. 제거될 때까지 하나씩 중지
- 자원 선점; 다음 사항을 고려한다.
  1. 희생자 선택 ; 자원 선점의 순서를 결정
  2. 롤백 ; 프로세스를 안전한 상태로 후퇴
  3. 기아 상태 ; 희생자 횟수를 체크

