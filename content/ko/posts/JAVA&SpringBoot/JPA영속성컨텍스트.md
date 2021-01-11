---
author: "brinst"
title: "JPA 영속성 컨텍스트와 플러시"
date: 2021-01-11T22:39:24+09:00
draft: false
---
## 영속성 관리
* JPA에서 가장 중요한 2가지
    * 객체와 관계형 DB 매핑하기 - 설계 관련
    * 영속성 컨텍스트 - JPA 내부동작

## 엔티티 매니저 팩토리와 엔티티 매니저
* JPA는 스레드가 하나 생성될 때 마다 EntityManagerFactory에서 EntityManager를 생성한다.
* EntityManager는 내부적으로 DB 커넥션 풀을 사용해서 DB에 붙인다.

## 영속성 컨텍스트
* <b>엔티티를 영구저장하는 환경</b>이라는 뜻
* EntityManger.presist(entity)
    * persist()를 하면 실제로는 DB에 저장하는 것이 아니라, 영속성 컨텍스트를 통해서 엔티티를 영속화 한다.
    * persist() 시점에는 영속성 컨텍스트에 저장한다.
    * DB저장은 나중임....

## 엔티티의 생명주기
* 비영속
    * 영속성 컨텍스트와 전혀 관계가 없는 상태
    ```
        //객체를 생성한 상태 (비영속)
        Member member = new Member();
        member.setId("member1);
        member.setUsername("회원1")
    ```
* 영속
    * 영속성 컨텍스트에 저장된 상태
    * 엔티티가 영속성 컨텍스트에 의해 관리된다.
    * 이때 DB에 저장 되지 않는다. 영속 상태가 된다고 DB에 쿼리가 날아가지 않는다.
    * 트랜잭션의 커밋 시점에 영속성 컨텍스트에 있는 정보들이 DB에 쿼리로 날아간다.
    ```
      // 객체를 생성한 상태 (비영속)
        Member member = new Member();
        member.setId("member1");
        member.setUsername("회원1");
        EntityManager entityManager = entityManagerFactory.createEntityManager();
        entityManager.getTransaction().begin();
        // 객체를 저장한 상태 (영속)
        entityManager.persist(member);
    ```
    * EntityManager.persist(entity)
        * 영속상태가 된다고 바로 DB에 쿼리가 날아가지 않는다. (즉, DB 저장 X)
    * transaction.commit()
        * 트랜잭션의 commit 시점에 영속성 컨텍스트에 있는 정보들이 DB에 쿼리로 날아간다.
* 준영속
    * 영속성 컨텍스트에 저장되었다가 분리된 상태
    * 영속성 컨텍스트에서 지운 상태
    ```
        //회원 엔티티를 영속성 컨텍스트에서 분리, 준영속 상태
        entityManger.detach(member);
    ```
* 삭제
    * 삭제된 상태. DB에서도 날린다.
    ```
        //객체를 삭제한 상태
        entityManager.remove(member);
    ```

## 영속성 컨텍스트의 이점
* Application과 DB 사이의 중간 계층의 영속성 컨텍스트가 존재하는 이유
    * 버퍼링, 캐싱 등의 이점

### 1. 1차 캐시

* 영속성 컨텍스트 내부에는 1차 캐시가 존재한다.
    * 1차 캐시를 영속성 컨텍스트라고 이해해도 됨
* Map<Key,Value> 로 1차 캐시에 저장된다.
    * key : @Id로 선언한 필드값 (DB pk)
    * value : 해당 객체

    ```
        Member member = new Member();
        member.setId("brinst");
        member.setUsername("brinst1")
        // 영속상태 (Persistence Context에 의해 Entity가 관리되는 상태)
        // DB 저장 X, 1차 캐시에 저장됨
        entityManager.persist(member);
        // 1차 캐시에서 조회
        Member brinst = entityManager.find(Member.class,"brinst");
    ```
    * 1차 캐시에 Entity가 있을 때의 이점
        * 조회
        * entityManager.find()를 하면 DB보다 먼저, 1차 캐시를 조회한다.
        * 1차 캐시에 해당 Entity가 존재하면 바로 반환한다.
        * 1차 캐시에 조회하고자 하는 Entity가 없다면?
            1. DB에서 조회한다.
            2. 해당 Entity를 DB에서 꺼내와 1차 캐시에 저장한다.
            3. Entity를 다시 반환한다.
            4. 이후에 다시 해당 Entity를 조회하면 1차 캐시에 있는 Entity를 반환한다.
        * 그러나, 사실 1차 캐시는 큰 성능 이점을 가지고 있지 않다.
            * EntityManager는 Transaction 단위로 만들고, 해당 DB Transaction이 끝날 때(사용자의 대한 요청에 대한 비지니스가 끝날 때) 같이 종료된다.
            * 즉, 1차 캐시도 모두 날아가기 때문에 굉장히 짧은 찰나의 순간에만 이득이 있다.(DB의 한 Transaction 안에만 효과가 있다.)
            * 하지만, 비지니스 로직이 굉장히 복잡한 경우에는 효과가 있다.

### 2. 동일성 보장(Identity)
```
Member a = entityManager.find(Member.class, "member1");
Member b = entityManager.find(Member.class, "member1");
a == b -> true
```
* 영속 Entity의 동일성(== 비교)을 보장한다.
    * 즉, "==" 비교가 true임을 보장한다.
    * member1에 해당하는 Entity를 2번 조회하면 1차 캐시에 의해 같은 Reference로 인식된다.
    * 하나의 Transaction안에서 같은 Entity 비교시 true

### 3. 엔티티 "등록" 시 트랜잭션을 지원하는 쓰기 지연(Transactional Write-Behind)
```
EntityManager entityManager = emf.createEntityManager();
EntityTransaction transaction = entityManager.getTransaction();
// EntityManager는 데이터 변경 시 트랜잭션을 시작해야 한다.
transaction.begin(); // Transaction 시작

entityManager.persist(memberA);
entityManager.persist(memberB);
//이때 까지 Insert SQL을 DB에 보내지 않는다.

//커밋하는 순간 DB에 INSERT SQL을 보낸다.
transaction.commit();
```
* entityManager.persist()
    * JPA가 insert SQL을 계속 쌓고 있는 상태
* transaction.commit()
    * 커밋하는 시점에 insert SQL을 동시에 DB에 보낸다.
    * 동시에 쿼리들을 보냄(쿼리를 보내는 방식은 동시 or 하나씩 옵션에 따라 다름)
* entityManager.persist(memberA)
    1. memberA가 1차 캐시에 저장된다.
    2. 1과 동시에 JPA가 Entity를 분석하여 insert Query를 만든다.
    3. insert Query를 <b>쓰기 지연 SQL 저장소</b>라는 곳에 쌓는다.
    4. DB에 바로 넣지 않고 기다린다.
* transaction.commit()
    1. **쓰기 지연 SQL 저장소**에 쌓여 있는 Query들을 DB로 날린다. (**flush**)
        * **flush()**는 1차캐시를 지우지는 않는다. 쿼리들을 DB에 날려서 DB와 싱크를 맞추는 역할을 한다.
    2. **flush()** 후에 실제 DB Transaction이 커밋된다.
### 4. 엔티티 "수정"시 변경 감지(Dirty Checking)
```
EntityManager em = emf.createEntityManaer();
EntityTransaction transaction = em.getTransaction();
transaction.begin(); //트랜잭션 시작

//영속 엔티티 조회
Member memberA = em.find(Member.class, "memberA");

//영속 엔티티 데이터 수정
memberA.setUsername("hi");
memberA.setAge(10);

transaction.commit();
```
* Entity 데이터 수정 시 update()나 persist()로 영속성 컨텍스트에 해당 데이터를 업데이트 해달라고 알려줘야 할 필요가 없다.
* Entity 데이터만 수정하고 commit 하면 알아서 DB에 반영됨
* 즉, 데이터를 set하면 해당 데이터의 변경을 알아서 감지하여 자동으로 UPDATE Query가 나가는 것이다.   
    
#### 변경 감지(Dirty Checking)
* 1차 캐시
    * @Id, Entity, Snapshot (값을 읽어온 최초의 상태)
    * Snapshot : 영속성 컨텍스트에 최초로 값이 들어왔을 때의 상태값을 저장한다.
* 변경 감지 매커니즘
    * transaction.commit()을 하면
        1. flush()가 일어날 때 엔티티와 스냅샷을 일일이 비교한다.
        2. 변경사항이 있으면 UPDATE Query를 만든다.
        3. 해당 UPDATE Query를 쓰기 지연 SQL 저장소에 넣는다.
        4. UPDATE Query를 DB에 반영한 후 commit()한다.

### 5. 엔티티 삭제
```
    Member memberA = em.find(Member.class,"memberA");

    em.remove(memberA); //엔티티 삭제
```
* 위의 Entity 수정에서의 매커니즘과 동일
* Transaction의 commit 시점에 DELETE Query가 나간다.

---
## 플러시(flush)
* 영속성 컨텍스트의 변경 내용을 DB에 반영하는 것을 말한다.
* Transaction commit이 일어날 때 flush가 동작하는데, 이때 쓰기 지연 저장소에 쌓아놨던 INSERT, UPDATE, DELETE SQL 들이 DB에 날아간다. -> **영속성 컨텍스트를 비우는 것이 아님**
* 영속성 컨텍스트의 변경사항들과 DB의 상태를 맞추는 작업
    * 플러시는 영속성 컨텍스트의 변경 내용을 DB에 동기화 한다.

### 플러시의 동작 과정
1. 변경을 감지한다. (Dirty Checking)
2. 수정된 Entity를 쓰기 지연 SQL 저장소에 등록한다.
3. 쓰기 지연 SQL 저장소의 Query를 DB에 전송한다.
    * flush가 발생한다고 해서 commit이 이루어지는 것이 아니고 flush 다음에 실제 commit이 일어난다.
    * flush가 동작할 수 있는 이뉴는 데이터베이스 트랜잭션(작업 단위)이라는 개념이 있기 때문이다.
        * 트랜잭션이 시작되고 해당, 트랜잭션이 commit 되는 시점 직전에만 동기화(변경 내용을 날림) 해주면 되기 때문에, 그 사이에서 플러시 매커니즘의 동작이 가능한 것이다.

#### 영속성 컨텍스트를 플러시 하는 방법
##### 1. em.flush()을 통한 직접 호출
```
// 영속 상태(Persistence Context에 의해 Entity가 관리되는 상태)
Member member = new Member(200L,"A");
entityManager.persist(member);

entityManager.flush(); // 강제 호출(쿼리가 DB에 반영됨)

tx.commit(); //DB에 insert query가 날아가는 시점(Transaction commit)
```
* 플러시가 일어나면 1차 캐시가 모두 지워질까?
    * 그대로 남아있음
    * 쓰기 지연 SQL 저장소에 있는 Query들만 DB에 전송까지만 되는 과정일 뿐임

##### 2. 트랜잭션 커밋 시 플러시 자동 호출
##### 3. JPQL 쿼리 실행 시 플러시 자동 호출
```
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);

// 중간에 JPQL 실행
query = entityManager.createQuery("select m from Member m",Member.class);
List<Member> members = query.getResultList();
```
* member A,B,C 를 영속성 컨텍스트에 저장한 상태에서 바로 조회하면 조회가 될까?
    * 조회가 되지 않는다.
    * DB에 Query로도 날아가야 반영이 될텐데 INSERT Query 자체가 날아가지 않은 상태이다.
    * 이 때문에 JPA의 기본모드는 JPQL 쿼리 실행시 flush()를 자동으로 날린다. -> JPQL 쿼리 실행시 플러시 자동 호출로 인해 위 코드는 조회가 가능하다.