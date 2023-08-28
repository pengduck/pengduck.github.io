---
title: 정규표현식 삽질 - asterisk(*) vs plus(+)
author: 펭덕
date: 2023-08-27 21:40:00 +0900
categories: [Dev, Tips]
tags: [Regex]
math: true
mermaid: true
image:
   src: https://github.com/pengduck/pengduck.github.io/assets/82709090/34b68e67-f625-4bee-85e2-41d8dc995c68
---

오랜만의 포스팅. 정신없는 와중에.. 이 글은 나중에 혹시나 어리둥절할 일을 방지하기 위한 메모이다.


## 잘 써오던 로직

DB에 정규표현식을 저장하여, 데이터를 수집 후 해당 표현식에 맞는 데이터를 분류해주는 로직이 있었다. 예를 들어 `penguin*` 으로 저장된 표현식의 목적은, 데이터의 key가 `penguin_emperor`, `penguin_gentoo` 와 같은 경우 분류하기 위한 크게 복잡하지 않은 용도였다. 꽤 오랫동안 아무 트러블이 없었다.

<br>

## 결국 터진 시한폭탄

비록 정규식을 염두에 두고 만들어진 로직이지만, `*` 이외엔 딱히 쓸일이 없었고, 특정 코드값을 분류하는 과정에서 터졌다.

- `block_a*`
- `block_b*`
- ...

와 같은 방식으로 표현식을 정의하였는데, `block_`를 접두로 한 모든 key 값들이 다 한 곳에 분류되는 것이었다..! 아니 a, b는 어디가고?

<br>

## 해골물..

<https://learn.microsoft.com/en-us/dotnet/standard/base-types/quantifiers-in-regular-expressions>

![regex1](https://github.com/pengduck/pengduck.github.io/assets/82709090/e0b44c55-f2e1-4e4f-8caf-77c2a7a984e3)

흔히 파일을 다룰 때 등등 wildcard `*`를 사용하는 것처럼 단순히 생각한 것이 문제였다. 여기서 asterisk는 Matches zero or more times, 0번 이상 일치.. 어이가 상실되는 순간이었다. 

여태 해골물을 먹고 있었다..

<br>

## +를 씁시다..

<https://regexr.com/> 에서 테스트를 해보자. 

![regex2](https://github.com/pengduck/pengduck.github.io/assets/82709090/b28d1bfb-9120-401e-9c97-991aede2ad0c)

> 4회 매칭

![regex3](https://github.com/pengduck/pengduck.github.io/assets/82709090/4d42d2a3-0be0-4711-b532-394c3e353473)

> 1회 매칭

원래의 의도대로라면 `+`를 쓰는 것이 적절한 것이다.