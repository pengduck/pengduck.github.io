---
title: 몽고DB (MongoDB) - value type 변경
author: 펭덕
date: 2023-05-13 21:40:00 +0900
categories: [Dev, Tips]
tags: [MongoDB, Type casting, document]
math: true
mermaid: true
image:
   src: https://github.com/pengduck/pengduck.github.io/assets/82709090/847f4310-4d8a-46ab-a5be-071bcb3664d4
---

## 뭐지 이거?

몽고DB를 사용하여 운영하는 중 쿼리를 사용해 데이터를 집어넣는 경우가 있었는데, 이후로 type casting 문제로 앱이 잘 안도는 것이었다. 황급히 find 쿼리로 보면 그냥 숫자값인데 왜? 앱에서는 double로 인식하고 있었다.

logstash로 수집 데이터를 MongoDB로 넣어준 뒤 그걸 꺼내 확인하는 앱인데 logstash로 넣은 데이터와 직접 수작업 쿼리로 넣은 게 타입이 다른가?

라는 의문은 잠시 뒤로하고, 결과가 그러하니 그렇게 이해하는 것으로 넘어가기로 하고 (다음에 알아볼수 있으면 알아봐야지..) 우선은 문제부터 해결해야 할 것이야.

<br>

## 필드 value의 타입 확인하기

우선 몽고 shell로 접속.

자바스크립트처럼 간단히 타입을 확인할 수 있다고 한다.

```shell
> typeof db.collection.findOne().field
```

허나 이것은 Object라던지, number라던지, 자바스크립트 답게 리턴값이 그런식으로 날라온다. (...) 내가 원하는 게 아니야!

aggregate를 쓰는 방법도 있었지만, 뭐.. 얼추 넣은 값을 알고 있었는데다가 모든 필드를 확인할 필요도 없으니, 하나하나 대입하여 맞추는게 가장 빠르겠더라.

- 아래 몽고DB의 데이터 타입 표를 참고하자. 어차피 숫자인걸 알고있으니, 1, 16, 18 세번만 하면 때려맞출 수 있지

| Type                | Number | Alias        |
|:--------------------|:------:|-------------:|
| Double              | 1      | "double"     |
| String              | 2      | "string"     |
| Object              | 3      | "object"     |
| Array               | 4      | "array"      |
| Binary data         | 5      | "binData"    |
| ObjectId            | 7      | "objectId"   |
| Boolean             | 8      | "bool"       |
| Date                | 9      | "date"       |
| Null                | 10     | "null"       |
| Regular Expression  | 11     | "regex"      |
| JavaScript          | 13     | "javascript" |
| 32-bit integer      | 16     | "int"        |
| Timestamp           | 17     | "timestamp"  |
| 64-bit integer      | 18     | "long"       |
| Decimal128          | 19     | "decimal"    |
| Min key             | -1     | "minKey"     |
| Max key             | 127    | "maxKey"     |

<https://www.mongodb.com/docs/manual/reference/operator/query/type/>

사용법은 그저..마구 검색해버리면 되는것이다. 아래는 field라는 필드(컬럼) 의 value 값이 double인지 확인하는 쿼리이다.

```shell
> db.collection.find({field:{$type:1}})
```

depth가 있는 필드를 알아보려면 따옴표를 붙이고 `"field.depth1"` 같은 방식으로 비교하면 된다. 
내 문제에서는 Double이었으므로 1을 넣었을 때 값이 출력되었다. 숫자를 쓰고 싶지 않다면 `"double"` 과 같은 `Alias`를 이용하면 된다.

<br>

## 필드 value 타입 변경

이제 들어간 값을 바꿔버릴 차례다.

```shell
> db.collection.updateMany(
  { field : { $type: 1 } },
  [{ $set: { field: { $toInt: "$field" } } }]
)
```

MongoDB는 각 `document` 녀석들이 자유분방하기 때문에 들어가 있는 값들을 `updateMany`로 통으로 바꿔줄 필요가 있다. 그리하여 updateMany의 첫 parameter는 filter(condition이라고 하는게 더 좋을거 같은데..)를, 두번째 parameter에 $set으로 field의 값을 바꿔주는 형태를 취하는 것이다. 

String으로 바꾸고 싶으면 `$toString`을 사용한다. 이 역시 depth가 있는 필드의 type을 변경하려면 `"$field.depth"`와 같이 사용할 수 있으니 적절히 사용해보자. 날짜같은 걸 바꾸려면 다소 고생하겠지만.. 언젠가 해볼 기회가 있겠지?

아래 레퍼런스를 참고해보자.

<https://www.mongodb.com/docs/manual/reference/operator/aggregation/>


<br>

아무튼 해프닝은 해결되었다. 그래도 앱의 로직에서 타입을 검사해서 수정하는 편이 좀더 좋은 방법일 거야.