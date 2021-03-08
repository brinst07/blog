---
author: "brinst"
title: "Spring Data JPA와 QueryDSL의 이해"
date: 2021-02-01T22:39:24+09:00
draft: false
---

### JPA 기반 프로젝트

- Spring Data JPA
- QueryDSL

### 스프링 데이터 JPA 소개

- 지루하게 반복되는 CRUD 문제를 세련된 방법으로 해결
- 개발자는 인터페이스만 작성
- 스프링 데이터 JPA가 구현 객체를 동적으로 생성해서 주입
  ![image](https://user-images.githubusercontent.com/60083557/106463255-8875c600-64da-11eb-98cd-51a04981944e.png)

#### 공통 인터페이스 기능

- JpaRepository 인터페이스 : 공통 CRUD 제공
- 제너릭은 <엔티티, 식별자>로 설정
