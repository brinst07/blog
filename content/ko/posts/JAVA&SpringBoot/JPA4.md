---
author: "brinst"
title: "JPA 기본기 다지기(4)"
date: 2021-01-27T22:39:24+09:00
draft: false
---

## 연관관계 매핑

### 객체를 테이블에 맞추어 모델링

> 식별자로 다시 조회, 객체지향적인 방법은 아니다.
> 연관관계가 없기 때문에, 직접 하나하나 다 조회해야 한다.

```java
            Team team = new Team();
            team.setName("teamA");
            em.persist(team);

            Member member = new Member();
//            member.setId(100L);
            member.setName("안녕하세요");
            member.setMemberType(MemberType.USER);
            member.setTeamId(team.getId());
            //persist -> 영구 저장하다.
            em.persist(member);


            //조회
            Member findeMember = em.find(Member.class, member.getId());

            //연관관꼐가 없음
            Team findTeam = em.find(Team.class, team.getId());

```

![image](https://user-images.githubusercontent.com/60083557/106375783-9b04d800-63d2-11eb-8149-585000d73775.png)

#### 객체를 테이블에 맞추어 데이터 중심으로 모델링하면, 협력관계를 만들 수 없다.

- 테이블은 외래키로 조인을 사용해서 연관된 테이블을 찾는다.
- 객체는 참조를 사용해서 연관된 객체를 찾는다.
- 테이블과 객체 사이에는 이런 큰 간격이 있다.
- 이런방식은 객체를 객체 답게 사용하는게 아니라, DB에 끼워 맞춘 방식이다.

### 단방향 매핑

> 객체의 참조와 테이블의 외래 키를 매핑

![image](https://user-images.githubusercontent.com/60083557/106375939-f97e8600-63d3-11eb-974c-5bc5cccc8792.png)

다음과 같이 연관관계를 지정하는 것이 단방향 매핑이다.
Member에서는 Team을 가져올수 있지만, Team에서는 Member를 가져오지 못한다.

- 객체 생성

  ```Java
  public class Member{
  ....
  @ManyToOne
      @JoinColumn(name = "TEAM_ID")
      private Team team;
  }
  ```

  - 다음과 같이 컬럼을 생성해준다.
  - ManyToOne은 Member입장에서 N:1이기 때문에 붙여준다.
  - JoinColumn은 어느 컬럼으로 조인할 것인지 정해주는 것인데, 생략하면 Default 값으로 들어가게 된다. 왠만하면 작성해준다.

- 개선된 조회 방식

  ```Java
      //조회
      Member member = e.find(Member.class, member.getId());

      //참조를 사용한 연관관계 조회
      Team findTeam = member.getTeam();
  ```

  위와 같이 여러번 조회하는 방식이 아닌 연관관계를 사용하여 한번에 객체를 가져오는 것이 가능해진다.

### 지연로딩

> 현업에서는 지연로딩을 권장한다.

위에서 실습을 진행하면 아래처럼 한방쿼리가 작성되어 실행되게 된다.

```sql
  select
        member0_.id as id1_0_0_,
        member0_.age as age2_0_0_,
        member0_.memberType as memberty3_0_0_,
        member0_.USERNAME as username4_0_0_,
        member0_.regDate as regdate5_0_0_,
        member0_.TEAM_ID as team_id6_0_0_,
        team1_.id as id1_1_1_,
        team1_.name as name2_1_1_
    from
        Member member0_
    left outer join
        Team team1_
            on member0_.TEAM_ID=team1_.id
    where
        member0_.id=?
```

이게 싫다면 지연로딩을 사용하면 된다.

```Java
public class Member{
    ...
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;

}
```

지연로딩을 사용하는 방법은 간단하다. ManyToOne 어노테이션에 Lazy라고 선언해주면 된다.

이렇게 하고 다시 실행하면 다음과 같은 쿼리가 실행되게 된다.

```sql
    select
        member0_.id as id1_0_0_,
        member0_.age as age2_0_0_,
        member0_.memberType as memberty3_0_0_,
        member0_.USERNAME as username4_0_0_,
        member0_.regDate as regdate5_0_0_,
        member0_.TEAM_ID as team_id6_0_0_
    from
        Member member0_
    where
        member0_.id=?
```

이때 team을 가져오지 않고 member만 가져온것을 볼수가있는데, 지연로딩은 객체를 직접적으로 사용할때 값을 가져온다.
![image](https://user-images.githubusercontent.com/60083557/106376312-79f2b600-63d7-11eb-98f7-54b457b1c2b4.png)
