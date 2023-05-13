---
title: 카프카 클러스터 (Kafka Cluster) - failover
author: 펭덕
date: 2023-04-26 20:45:00 +0900
categories: [Dev, Tips]
tags: [Kafka, Cluster, Failover, Topic]
math: true
mermaid: true
image:
   src: https://github.com/pengduck/pengduck.github.io/assets/82709090/384595a8-1066-4ea8-969e-2d09b1222c8b
---


## 뭐야 왜 하나 뻗으니 전부 다 작동안해?

카프카를 클러스터로 구성했으니 마땅히 하나가 뻗으면 나머지가 제 역할을 수행해 줄 것이라 굳게 믿고 있었으나 뒤통수를 맞고 말았다..  하나가 죽었는데 producing, consuming이 모두 뻗어버리고 만 것이다.

우선은 리더라는 개념이 있는 모양이다. 알아서 분산하는 줄 알고있었더니 아니었다.. Leader와 Follower가 있고, Leader가 죽을 경우 Follower가 Leader의 역할을 수행하지만 이는 ISR(In Sync Replica) 라는 그룹에 묶여 있을 때의 이야기이다.


```shell
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic test --describe
```

토픽을 describe 하고 여기서 봐야 하는 것이 파티션 정보. 각 팔로워가 해당 파티션을 모두 가지고 있어야 하는 것이다. 내 경우에는 성능을 위해 파티션을 꽤 많이 만들어두었었는데, 파티션을 골고루..가지고 있었다. 하나만 죽어도 정합성이 깨져 전체가 먹통이 되어버리는 것이다.

<br>

## 살려야한다

이를 예방하기 위한 방법으로 두 가지가 있다. 가령 3개로 클러스터를 구성한다고 한다면,

### properties에 default.replication.factor 옵션을 주고 실행 후 토픽 만들기

```
# kafka // server.properties
default.replication.factor=3
```

### 토픽 생성시 옵션을 주는 방법

```shell
./bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 3 --topic test
```

클러스터가 3개인 경우 복제 factor를 3개 두어서 토픽을 만들 때 모두 복제하게끔 설정하는 것이다. 그러나 이것은 그저 예방의 방법. 이미 토픽을 생성한 뒤에 외양간이라도 고치려면(뒤늦게 복제를 행하려면) 약간 번거로운 방법을 행해야 한다.

### 이미 만들어진 토픽은?

```
[{"topic":"test", "partition": "0", "replicas":{0,1,2}},
{"topic":"test", "partition": "1", "replicas":{1,2,0}},
{"topic":"test", "partition": "2", "replicas":{2,0,1}}]
```

대략 요로코롬 생긴 json 파일이 필요한데, 토픽과 파티션의 숫자만큼 적어주고 리더가 될 브로커 순서를 명시해둔다.

그리고 나서

```shell
./bin/kafka-reassign-partitions.sh --bootstrap-server localhost:9092 --reassignment-json-file test.json --execute
```

로 실행해주면, 모든 카프카 클러스터 각 브로커의 logs.dir 위치에, 모든 파티션들이 나타남을 확인할 수 있을 것이다.

그렇게 되면 카프카 중 하나가 죽어도 리더가 바뀌며 컨슈밍/프로듀싱에 애로사항이 나타나지 않을 수 있다.


