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
  - 가장 많이 사용됨
  - name : 필드와 매핑할 테이블의 컬럼 이름
  - insertable, updateable : 읽기 전용
  - nullable : null 허용여부 결정, DDL 생성시 사용
  - unique : 유니크 제약 조건, DDL 생성시 사용
  - columnDefinition, length, precision, scale (DDL)
- @Enumerated()
  - 열거형 매핑
  - EnumType.ORDINAL : 순서를 저장(기본값)
  - EnumType.STRING : 열거형 이름을 그대로 저장, 가급적 이것을 사용해야한다.
  - Original을 사용하게 되면 Enum 클래스 안의 순서대로 숫자가 매핑 되는데 만약 중간에 다른 값이 삽입될 경우 마구잡이로 꼬이게 된다.
  - String으로 하게 될 경우는 값 그대로가 insert 되게 된다.
    ```java
    @Enumerated(EnumType.STRING)
    private MemberType memberType;
    ```
  - [우아한 형제들 기술블로그 enum](https://woowabros.github.io/tools/2017/07/10/java-enum-uses.html)

* @Temporal

  - 날짜 타입 매핑

    ```java
    @Temporal(TemporalType.DATE)
    private Date date; //날짜

    @Temporal(TemporalType.TIME)
    private Date time; //시간

    @Temporal(TemporalType.TIMESTAMP)
    private Date timestamp; //날짜와 시간
    ```

* @Lob

  - 컨텐츠 길이가 너무 길때 Binary 파일로 바로 밀어 넣을 때 사용한다.
  - CLOB, BLOB 매핑
  - CLOB : String, char[], java, sql.CLOB
  - BLOB : byte[], java.sql.BLOB

    ```java
    @Lob
    private String lobString;

    @Lob
    private byte[] lobByte;
    ```

* @Transient
  - 이 필드는 매핑하지 않는다.
  - DB와 관계없이 객체에서만 다루기 위한 필드
  - 애플리케이션에서 DB에 저장하지 않는 필드

## 식별자 매핑 방법

- 식별자 매핑 어노테이션
  - @Id
  - @GeneratedValue
    ```java
    @Id @GeneratedValue(strategy = GeneratioinType.AUTO)
    private Long id;
    ```
    - IDENTITY : 데이터베이스에 위임, MYSQL
    - SEQUENCE : 데이터베이스 시퀀스 오브젝트 사용, ORACLE
      - @SequenceGenerator 필요
    - TABLE : 키 생성용 테이블 사용, 모든 DB에서 사용
      - @TableGenerator 필요
    - AUTO : 방언에 따라 자동 지정, 기본값

### 권장하는 식별자 전략

- 기본 키 제약 조건 : null아님, 유일, 변하면 안된다.
- 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다. 대리키(대체키)를 사용하자.
- 예를 들어 주민등록번호도 기본 키로 적절하지 않다.
  - 이유 과거에는 주민등록번호를 보관하고 할수 있었지만 이제는 그렇게 할 수 없음....
- 권장사항
  1. DB에서 제공해주는 AutoIncreament, Sequence 등을 사용
  - 사용시 int 타입을 사용하면 억단위에서 끝남 -> 따라서 Long 타입을 사용해야한다.
  2. Long + 대체키 + 키 생성전략 사용
  3. UUID(성능이슈 있을 수 있음)
