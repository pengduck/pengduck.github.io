---
title: 보호 (Protection)
author: 펭덕
date: 2023-03-13 20:45:00 +0900
categories: [CS지식, OS]
tags: [공룡책, OS, Operating System, 운영체제, 보호]
math: true
mermaid: true
image:
   src: https://user-images.githubusercontent.com/82709090/224716014-c92c853a-bfb9-4117-b27b-0b844538f55c.png
---

공룡책(10판) Chapter 17 요약

보안은 컴퓨터 자원을 보호하는 것에 중점을, 보호는 프로세스와 사용자의 접근을 제어하는데 중점을 맞춘다.

## 보호의 목표 _ Goals of Protection

컴퓨터 시스템의 응용이 더욱 복잡, 광범위해짐으로, 시스템의 무결성을 보호할 필요성이 증가하였다. 신뢰가 없는 사용자도 이용할 수 있는 공동의 논리적 공간, 물리적 공간을 안전하게 공유하기 위해 고안되었다. 결국 접근 제한을 악의적, 의도적으로 위반하는 것을 방지하기 위해, 자원을 정해진 **사용 정책대로 사용하도록 보장**하기 위함이다. `기법(mechanism, 어떻게 할지)`과 `정책(policy, 무엇을 할지)`의 구분이 중요하다. 

<br>

## 보호의 원칙 _ Principles of Protection

`최소한의 권한 원칙 (principle of least privilege)`을 준수한다. 딱 태스크를 수행할 만큼만 권한을 부여한다는 것이다. 우발적인 실수를 막기 위해서도, **root로 수행하는 것은 지양**하며, 필요한 곳마다 필요한 만큼의 `권한(permission`)을 부여하여 이용할 수 있게 한다. 제한을 통해 각종 구성요소들의 권한을 `구획화(compartmentalization)` (DMZ,가상화 등), 접근에 대한 `감사 추적(audit trail)`을 병행한다.

<br>

## 접근 행렬 _ Access Matrix

행렬을 이용하여 ACL(access control list, 권한의 집합)를 만들어 관리한다. 행은 영역(domain)을, 열은 객체(object)를 나타낸다. 각 항은 접근 권한의 집합으로 구성된다. 접근 행렬을 이용해 각 도메인이 어떤 자원에 접근할 수 있을지에 대한 정책(policy)을 관리할 수 있다.

![accessmatrix](https://user-images.githubusercontent.com/82709090/224716084-beb1b7ba-9800-4603-9f7c-bc4effaff60b.png)

> ex) D<sub>3</sub>은 F<sub>2</sub>에 대해 읽기 권한과 F<sub>3</sub>에 대한 실행 권한을 가진다.

<br>

## 기타 보호 개선 방법 _ Other Protection Improvement Methods

1. 샌드박싱(Sandboxing): 할 수 있는 것을 제한하는 환경에서 프로세스를 실행하는 작업이 포함된다. 기본적으로 프로세스는 어떤 `자격 증명, 서명(credentials)`을 받아서 엑세스 가능한 범위의 행위만 수행할 수 있다.
2. 코드 서명(Code Signing): 프로그램이나 스크립트의 신뢰를 보장하기 위하여, 프로그램과 실행 파일의 디지털 서명으로 프로그램을 만들고 변경되지 않았음을 확인한다.

<br>

후우..한숨 돌리고 남은 부분을 쉬엄쉬엄 메꾸도록 합시다. 고생 많았어요 멤버님들 모두