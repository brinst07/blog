---
title: "JPA 정리"
date: 2021-01-12T22:41:24+09:00
draft: false
---

## JPA란??

Java 언어를 통해서 데이터베이스와 같은 영속 계층을 처리하고자 하는 스펙

## ORM과 JPA

### ORM

- 객체지향과 관련이 있음
- '객체지향 패러다임을 관계형 데이터베이스에 보존하는 기술'
- '객체지향 패러다임을 관계형 패러다임으로 매핑해주는 개념'
- 관계형 데이터베이스의 테이블을 설계하여 데이터를 보관하는 틀을 만든다는 의미에서 클래스와 상당히 유사함
  {{< codes java sql >}}
  {{< code >}}

  ```java
    public class Member{
        private String id;
        private String pw;
        private String name;
    }
  ```

  {{< /code >}}

  {{< code >}}

  ```sql
    Member
    id -> varchar2(50)
    pw -> varchar2(50)
    name -> varchar2(100)
  ```

  {{< /code >}}
  {{< /codes >}}

* 클래스와 테이블이 유사하듯이 '인스턴스'와 'Row'도 상당히 유사함
  - 객체지향에서는 클래스에서 인스턴스를 생성하여 인스턴스에 보관
  - 테이블에서는 하나의 'Row'에 데이터를 저장
* 관계와 참조라는 의미도 매우 유사함
  - 관계형 데이터베이스는 테이블 사이의 관계를 통하여 구조적인 데이터를 표현
  - 객체지향에서는 참조를 통하여 어떤 객체가 다른 객체들과 어떤 관계를 맺고 있는지를 표현

### JPA

- 'Java Persistence API"의 약어
- ORM을 Java언어에 맞게 사용하는 스펙
- ORM이 더 상위의 개념이다.
- JPA는 단순한 스펙이기 때문에 해당 스펙을 구현하는 구현체마다 회사의 이름이나 프레임워크의 이름이 다르게 된다.
- 그중에서 가장 유명한 것이 Hibernate

## 엔티티 클래스와 JpaRepository

Spring Data JPA가 개발에 필요한 것은 단지 두 종류의 코드이다.

- JPA를 통해서 관리하게 되는 객체(이하 엔티티 객체)를 위한 엔티티 클래스
- 엔티티 객체들을 처리하는 기능을 가진 Repository
  - repository는 Spring Data JPA에서 제공하는 인터페이스로 설계하는데 스프링 내부에서 자동으로 객체를 생성하고 실행하는 구조이다.

### 엔티티 클래스

```Java
    @Entity
    @Table(name = "tbl_memo")
    @ToString
    @Getter
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public class Memo{
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long mno;

        @Coulmn(length = 200, nullable = false)
        private String memoText;
    }
```

- @Entity

  - 엔티티 클래스는 반드시 @Entity라는 어노테이션을 추가해야한다.
  - 해당 클래스가 엔티티를 위한 클래스라는 것을 의미
  - 해당 클래스의 인스턴스들이 JPA로 관리되는 엔티티 객체라는 것을 의미

- @Table

  - 데이터베이스상에서 엔티티 클래스를 어떠한 테이블로 생성할 것인지에 대한 정보를 담기 위한 어노테이션
  - @Table(name = "") 이런 형식으로 이름도 정할 수 있다.
  - 단순히 테이블의 이름 뿐만 아니라 인덱스 등을 생성하는 설정도 가능하다.

- @Id, @GeneratedValue

  - @Entity가 붙은 클래스는 PK에 해당하는 특정 필드를 @Id로 지정해야만 한다.
  - @Id가 사용자가 입력하는 값을 사용하는 경우가 아니면 자동으로 생성되는 번호를 사용하기 위해 @GeneratedValue라는 어노테이션을 사용한다.
  - (strategy = GenerationType.IDENTITY) 키 생성 전략이라고도 한다.
    - AUTO - JPA 구현체가 생성 방식을 겨렂
    - IDENTITY - 사용하는 데이터베이스가 키 생성을 결정
    - SEQUENCE - 데이터베이스의 sequence를 이용하여 키를 생성, @SequenceGenerator와 같이 사용
    - TABLE - 키 생성 전용 테이블을 생성해서 키 생성, @TableGenerator와 함께 사용

- @Coulmn

  - 추가적인 필드가 필요한 경우에도 마찬가지로 어노테이션을 활용한다.
  - nullable, name, length 등을 이용해서 데이터베이스의 칼럼에 필요한 정보를 제공
  - columnDefinition을 이용하여 기본값도 지정이 가능하다.

- @Builder
  - 객체 생성하기 위한 어노테이션
  - @AllArgsConstructor와 @NoArgsConstructor를 같이 사용해야 컴파일 에러가 발생하지 않음

### JpaRepository 인터페이스

Sprin Date JPA에는 여러 종류의 인터페이스의 기능을 통해서 JPA 관련 작업을 별도의 코드 없이 처리할수 있게 지원한다. 그 중의 하나가 JpaRepository 인터페이스이다.
일반적인 기능을 사용할 때는 CrudRepository를 사용하면 되고, 모든 기능을 사용하고 싶다면 JpaRepository를 이용하면 된다. 일반적으로는 JpaRepository를 사용한다.

```Java
    public interface SampleRepository extends JpaRepository<Memo,Long> {

    }
```

- JpaRepository를 사용할 때는 엔티티의 타입정보(Memo 클래스 타입)와 @Id의 타입을 지정한다.
- 위 처럼 인터페이스의 선언만으로도 자동으로 스프링의 bean으로 등록된다.
- CRUD
  - insert 작업 : save(엔티티 객체)
  - select 작업 : findById(키 타입), getOne(키 타입)
  - update 작업 : save(엔티티 객체)
  - delete 작업 : deleteById(키 타입), delete(엔티티 객체)
  - save -> JPA의 구현체가 메모리상에서 객체를 비교하고 없다면 insert, 존재한다면 update를 동작시키는 방식으로 동작

#### 데이터베이스 방언(dialect)

- JPA는 특정 데이터베이스에 종속적이지 않은 기술
- 각각의 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다르디
  - 가변 문자 : MySQL은 VARCHAR, ORACLE은 VARCHAR2
  - 문자열을 자르는 함수 : SQL표준은 SUBSTRING(), ORACLE은 SUBSTR()
  - 페이징 : MySQL은 LIMIT, ORACLE은 ROWNUM
- 방언 : SQL 표준을 지키지 않거나 특정 데이터베이스만의 고유한 기능

* JPA를 사용하게 되면 방언이 존재하기 때문에 ORACLE -> MYSQL 이나 그반대로 전환이 엄청 빠르다...
