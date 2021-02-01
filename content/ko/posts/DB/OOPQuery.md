---
title: "OOPQuery"
date: 2021-02-01T22:41:24+09:00
draft: false
---

### JPA는 다양한 쿼리 방법을 지원

- JPQL
- JPA Criteria
- QueryDSL
- 네이티브 SQL
- JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용

### JPQL

- JPA를 사용하면 엔티티 객체를 중심으로 개발
- 문제는 검색 쿼리
- 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색
- 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능
- 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색조건이 포함된 SQL이 필요
- JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공
- SQL과 문법 유사, SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원
- JPQL은 엔티티 객체를 대상으로 쿼리
- SQL은 데이터베이스 테이블을 대상으로 쿼리
- 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리
- SQL을 추상화해서 특정 데이터베이스 SQL에 의존X
- JPQL을 한마디로 정의하면 객체 지향 SQL

```Java
//검색
String jpql = "select m FROM Member m where m.name like '%hello%'";

List<Member> result = em.createQuery(jpql, Member.class).getResultList();
```

### JPQL 문법

- select m from Member m where m.age > 18
- 엔티티와 속성은 대소문자 구분(Member, username)
- JPQL 키워드는 대소문자 구분 안함(SELECT, FROM, where)
- 엔티티 이름을 사용, 테이블 이름이 아님(Member)
- 별칭은 필수(m)

### 결과 조회 API

- query.getResultList() : 결과가 하나 이상, 리스트 반환
- query.getSingleResult() : 결과가 정확히 하나, 단일 객체 반환(정확히 하나가 아니면 예외 발생)

### 파라미터 바인딩 - 이름 기준, 위치 기준

```SQL
SELECT m FROM Member m where m.username=:username
query.setParameter("username",usernameParam);

SELECT m FROM Member m where m.username=?1
query.setParameter(1,usernameParam);
```

### 프로젝션

- SELECT m FROM Member m -> 엔티티 프로젝션
- SELECT m.team FROM Member m -> 엔티티 프로젝션
- SELECT username,age FROM Member m -> 단순 값 프로젝션
- new 명령어 : 단순 값을 DTO로 바로 조회<br>
  SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m
- DISTINCT는 중복 제거

### 페이징 API

- JPA는 페이징을 다음 두 API로 추상화
- setFirstResult(int startPosition) : 조회 시작 위치(0부터 시작)
- setMaxResults(int maxResult) : 조회할 데이터 수

### 조인

- 내부 조인 : SELECT m FROM Member m [INNER] JOIN m.team t
- 외부 조인 : SELECT m FROM Member m LEFT [OUTER] JOIN m.team t
- 세타 조인 : SELECT count(m) from Member m, Team t where m.username = t.name
  - 연관관계 없어도 조인할 수 있는 막 조인

### 페치 조인

- 엔티티 객체 그래프를 한번에 조회하는 방법
- 별칭을 사용할 수 없다.
- N+1를 해결하기 위한 방법이다.
- 모든 조인을 다 Lazy 로 바르고 특정 부분만 페치 조인을 사용하는게 베스트다.
- JPQL : select m from Member m join fetch m.team
  - 마치 eager를 건것처럼 한번에 가져온다.
- SQL : SELECT M._, T._ FROM MEBER M INNER JOIN TEAM T ON M.TEAM_ID=T.ID
