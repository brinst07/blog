---
title: "HierarchicalQuery"
date: 2021-11-14T22:00:29+09:00
draft: false
---

# MariaDB 계층형 쿼리짜기
mariadb에는 oralce과 다르게 계층형 쿼리를 작성할 수 있는 connect by 가 존재하지 않는다.
기존에는 지원하지 않았지만, mariadb 10버전부터 계층형쿼리를 구현할 수 있는 rescursive가 지원된다.

```sql
WITH RECURSIVE CTE AS (
	select *
	from user
	where parent_user is null
	# 1
	union

	select *
	from user as a, CTE b
	where a.parent_user = b.id
	# 2
)
select * from user;
```
이 쿼리를 실행하면 다음과 같이 동작한다.
1. 1번 쿼리가 실행되어 parent_user가 null인 user를 찾는다.
2. 2번 쿼리가 실행되어 1번쿼리의 조건에 맞는 결과를 찾는다.
3. 1번 쿼리와 2번쿼리를 union한다.
4. 2번쿼리가 실행되어 이전에 2번쿼리에서 나온 결과에 대하여 조건에 맞는 결과를 찾는다.
5. union한다.
6. 결과가 없을 때까지 위 작업을 반복한다.

만약 계층에 따른 level을 출력하고 싶다면 다음과 같이 작성하면된다.
```sql
WITH RECURSIVE CTE AS (
    SELECT id,group_name, parent_group, 1 as level
    FROM user_group
    WHERE group_name = 'system'

    UNION ALL

    SELECT a.id, a.group_name,a.parent_group, b.level +1
    FROM user_group a , CTE b
    where a.parent_group = b.id
)
select * from cte;
```