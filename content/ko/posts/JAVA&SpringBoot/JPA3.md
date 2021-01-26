---
author: "brinst"
title: "JPA 기본기 다지기(3)"
date: 2021-01-26T22:39:24+09:00
draft: false
---

## 데이터베이스 스키마 자동 생성하기

- DDL을 애플리케이션 실행 시점에 자동 생성
- 테이블 중심 -> 객체중심
- 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 생성
- 이렇게 생성된 <b>DDL은 개발 장비에서만 사용</b>
- 생성된 DDL은 운영서버에서는 사용하지 않거나, 적절히 다듬은 후 사용
- hibernate.hbm2ddl.auto
  - create : 기존 테이블 삭제 후 다시생성(DROP + CREATE)
  - create-drop : create와 같으나 종료시점에 테이블 DROP
  - update : 변경분만 반영(운영DB에는 사용하면 안됨)
    - 보통 DB에는 수천 수만건이 있는에 여기에 alter문을 때리게 되면 보통 DB에 락이 걸리게 되고 그것은 곧 지옥.....
  - validate : 엔티티와 테이블이 정상 매핑되었는지만 확인
  - none : 사용하지 않음

## 데이터베이스 스키마 자동 생성하기 주의!!!

- 운영장비에는 절대 create, create-drop, update 사용하면 안된다.
- 개발 초기 단게는 create 또는 update
- 테스트 서버는 update 또는 validate
- 스테이징과 운영 서버는 validate 또는 none

## 매핑 어노테이션(철저하게 DB 와 매핑)

- @Column
  - 자바 객체는 name이지만 DB 컬럼 이름은 USERNAME이라고 하고 싶을때 사용한다.
    ```java
    @Column(name="USERNANE")
    private String name;
    ```
- @Enumerated()
  - 반드시 String을 사용해야한다.
  - Original을 사용하게 되면 Enum 클래스 안의 순서대로 숫자가 매핑 되는데 만약 중간에 다른 값이 삽입될 경우 마구잡이로 꼬이게 된다.
  - String으로 하게 될 경우는 값 그대로가 insert 되게 된다.
    ```java
    @Enumerated(EnumType.STRING)
    private MemberType memberType;
    ```
  - [우아한 형제들 기술블로그 enum](https://woowabros.github.io/tools/2017/07/10/java-enum-uses.html)
