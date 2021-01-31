---
author: "brinst"
title: "JPA 기본기 다지기(5)"
date: 2021-01-27T22:39:24+09:00
draft: false
---

## 양방향 매핑

![image](https://user-images.githubusercontent.com/60083557/106384193-f81d7f80-640c-11eb-8280-e8dd4d88adbc.png)

### 객체와 테이블이 관계를 맺는 차이

- 객체 연관관계
  - 회원 -> 팀 연관관계 1개(단방향)
  - 팀 -> 회원 연관관계 1개(단방향)
- 테이블 연관관계
  - 회원 <-> 팀의 연관관계 1개(양방향)

### 객체의 양방향 관계

- 객체의 양방향 관계는 사실 양방향 관계가 아니라 서로 다른 단방향 관계 두개이다.
- 객체를 양방향으로 참조하려면 단방향 연관관게를 2개 만들어야 한다.

### 테이블의 양방향 관계

- 테이블은 외래 키 하나로 두 테이블의 연관관계를 관리
- Member.Team_ID 외래키 하나로 양방향 연관관계 가짐(양쪽으로 조인 할 수 있다.)

  ```sql
      SELECT *
      FROM MEMBER M
      JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID

      SELECT *
      FROM MEMBER M
      JOIN TEAM T ON T.TEAM_ID = M.TEAM_ID

  ```

### 아이러니.......

하지만 이렇게 될 경우 아이러니에 빠지게 된다. Member 객체가 진짜인지 Team 객체 안의 memebers안의 member가 진짜인지...
따라서 둘 중 하나로 관리를 해야하는 것이다.

#### 연관관계의 주인

##### 양방향 매핑 규칙

- 객체의 두 관계중 하나를 연관관계의 주인으로 지정
- 연관관계의 주인만이 외래 키를 관리(등록,수정)
- 주인이 아닌쪽은 읽기만 가능
- 주인은 mappedBy 속성 사용X
- 주인이 아니면 mappedBy 속성으로 주인 지정

> 외래키가 있는 곳을 주인으로 정해라. 즉 다인 쪽....

```java
    @Entity
    public class Team{
        @Id @GeneratedValue
        private Long id;

        private String name;

        @OneToMany(mappedBy = "team")
        List<Member> members = new ArrayList<Member>();
    }
```

따라서 N:1중 1인 쪽인 Team에 mappedBy 어노테이션을 붙여 읽기만 가능하게 지정해주었다.

#### 양방향 매핑시 가장 많이 하는 실수

```java
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);

    Member member = new Member();
    member.setName("member1");

    //역방향(주인이 아닌 방향)만 연관관계 설정
    team.getMembers().add(member);
    //연관관계의 주인에 값 설정
    member.setTeam(team);

    em.persist(member);
```

위 처럼 역방향 읽기전용 객체에 값을 넣게 되면 값이 들어가질 않는다.

값의 주인에 하단 처럼 값을 넣어주고, 객체관계를 고려하여 항상 양쪽다 값을 입력해주도록 한다.

### 양방향 매핑의 장점

- 단방향 매핑만으로 이미 연관관계 매핑은 완료
- 양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가된 것 뿐
- JPQL에서 역방향으로 탐색할 일이 많음
- 단방향 매핑을 잘 하고 양방향은 필요할 때 추가해도됨(테이블에 영향을 주지 않음)
  - 배달의 민족에서는 모두다 단방향 매핑으로 설계를 하고 양방향이 필요할때마다 추가하는 식으로 개발을 진행한다고 한다. 어쩌피 양방향은 테이블에 영향을 미치지 않기 때문에 자유롭게 바꿔도 되는듯....
