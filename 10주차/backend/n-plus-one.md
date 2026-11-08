# N+1

## N+1 문제란

조회 쿼리 **1번**으로 엔티티 N개를 가져왔는데, 연관 엔티티를 채우려고 **쿼리가 N번 더** 나가는 문제다.

```java
@Entity
public class Team {
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
}
```

```java
List<Team> teams = teamRepository.findAll();   // 1번: SELECT * FROM team
for (Team team : teams) {
    team.getMembers().size();                   // 팀마다 1번: SELECT * FROM member WHERE team_id = ?
}
```

팀이 100개면 쿼리가 101번 나간다.

```text
SELECT * FROM team;
SELECT * FROM member WHERE team_id = 1;
SELECT * FROM member WHERE team_id = 2;
...
SELECT * FROM member WHERE team_id = 100;
```

## 원인

### EAGER일 때

```java
em.createQuery("select t from Team t", Team.class).getResultList();
```

1. JPQL을 그대로 SQL로 바꿔서 `SELECT * FROM team`만 실행한다.
2. 결과를 받아보니 `members`가 EAGER라 바로 채워야 한다.
3. 팀마다 SELECT를 한 번씩 더 실행한다.

### LAZY일 때

LAZY로 바꾸면 조회 시점에는 쿼리가 1번만 나간다. 하지만 반복문에서 연관 엔티티에 접근하는 순간 프록시를 하나씩 초기화하면서 똑같이 N번이 나간다.

**LAZY는 N+1을 해결하지 않는다.** 쿼리가 나가는 시점을 늦출 뿐이다. 다만 LAZY여야 필요한 곳에서만 골라서 해결할 수 있다.

## 해결 방법

### 1. fetch join

```java
@Query("select t from Team t join fetch t.members")
List<Team> findAllWithMembers();
```

```sql
SELECT t.*, m.* FROM team t INNER JOIN member m ON m.team_id = t.id
```

- 연관 엔티티를 한 쿼리로 같이 가져온다. 가장 기본적인 해결책이다.
- 일반 join은 SELECT 절에 연관 엔티티를 넣지 않는다. 조인 조건으로만 쓰고 연관 엔티티는 여전히 프록시다.
- 내부 조인이라 멤버가 없는 팀은 빠진다. 필요하면 `left join fetch`를 쓴다.

**컬렉션 fetch join의 주의할 점**

- **결과가 뻥튀기된다.** 팀 하나에 멤버가 3명이면 같은 팀 행이 3개 나온다.
  - Hibernate 6부터는 같은 엔티티를 자동으로 중복 제거한다. 그 전에는 `distinct`를 붙였다.
- **페이징을 할 수 없다.** 행 수가 팀 수가 아니라 멤버 수라서 DB에서 LIMIT을 걸 수 없다.
  - Hibernate는 경고(`HHH90003004`)를 남기고 **전체를 메모리로 가져와서** 페이징한다. 데이터가 많으면 OOM이 날 수 있다.
- **컬렉션은 하나만 fetch join한다.** 둘 이상이면 곱해져서 결과가 폭발한다. `List` 둘을 fetch join하면 `MultipleBagFetchException`이 난다.

`~ToOne` 관계는 행이 늘어나지 않아서 몇 개를 fetch join해도 페이징이 된다.

### 2. @EntityGraph

```java
@EntityGraph(attributePaths = {"members"})
@Query("select t from Team t")
List<Team> findAllWithMembers();
```

- JPQL을 쓰지 않고 애노테이션으로 fetch join을 붙인다.
- 기본으로 left outer join이 나간다.
- Spring Data JPA 쿼리 메서드(`findByName`)에 바로 붙일 수 있어서 편하다.

### 3. Batch Size

```yaml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 100
```

```sql
SELECT * FROM team LIMIT 10;
SELECT * FROM member WHERE team_id IN (1, 2, 3, ..., 10);
```

- 프록시나 컬렉션을 초기화할 때 같은 종류의 다른 프록시도 모아서 **IN 절로 한 번에** 가져온다.
- 쿼리가 1 + N에서 1 + (N / batch size)로 줄어든다.
- 부모 쿼리는 그대로라 **페이징이 된다.** 컬렉션 조회 + 페이징이 필요하면 이 방법을 쓴다.
- 보통 100 ~ 1000 사이로 잡는다. DB마다 IN 절 파라미터 수 제한이 있다.
- 엔티티별로는 `@BatchSize(size = 100)`으로 설정한다.

### 4. DTO 직접 조회

```java
@Query("select new com.example.TeamDto(t.name, m.name) from Team t join t.members m")
List<TeamDto> findTeamDtos();
```

- 필요한 컬럼만 골라서 가져온다. 엔티티가 아니라서 영속성 컨텍스트가 관리하지 않는다.
- 성능은 가장 좋지만 화면에 맞춘 쿼리라 재사용이 어렵다. 별도 조회용 Repository로 분리한다.

## 정리

| 상황 | 방법 |
| --- | --- |
| `~ToOne` 연관 | fetch join (페이징 가능) |
| 컬렉션, 페이징 없음 | fetch join 또는 `@EntityGraph` |
| 컬렉션 + 페이징 | `~ToOne`만 fetch join + `default_batch_fetch_size` |
| 성능이 아주 중요한 조회 | DTO 직접 조회 |

기본 설정은 **모든 연관관계 LAZY + `default_batch_fetch_size` 전역 설정**으로 두고, 필요한 곳에서 fetch join을 쓰는 방식이 일반적이다.

## N+1 찾는 법

- `spring.jpa.show-sql=true` 또는 `logging.level.org.hibernate.SQL=debug`로 SQL 로그를 본다.
- `p6spy`를 쓰면 바인딩된 파라미터까지 볼 수 있다.
- Hibernate 통계(`hibernate.generate_statistics=true`)로 요청당 쿼리 수를 확인한다.
- 테스트에서 쿼리 수를 세서 검증하는 방법도 있다.
