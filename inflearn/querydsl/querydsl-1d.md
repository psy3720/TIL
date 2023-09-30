### 2023.09.30

이 내용은 김영한님의 "실전! Querydsl" 강의를 기반으로 정리하였습니다.

----

#### QueryDSL
> QueryDSL은 자바 기반의 데이터베이스 쿼리 작성을 위한 도메인 특화 언어(DSL)중 하나이다.   
QueryDSL은 SQL 쿼리를 프로그래밍적으로 작성하고 관리하기 위한 강력한 도구로 사용된다.   
이를 통해 데이터베이스와 상호 작용할 때 생기는 일반적인 문제를 해결하고, 더 안전하고 유지보수 가능한 코드를 작성하는데 도움이 된다.

주요 특징은 다음과 같다.
1. 타입안정성(Type Safety)
: QueryDSL은 자바 언어를 기반으로 하므로 컴파일 타임에 타입 안정성을 보장한다.  
이는 코드를 작성할 때 발생할 수 있는 오타나 오류를 줄여준다.

2. SQL 문법 추상화
: QueryDSL은 SQL을 추상화하고 자바 코드로 표현할 수 있도록 도와준다.   
이로써 SQL 쿼리 작성 시 발생할 수 있는 문법 오류나 데이터 타입오류를 방지할 수 있다.

3. 동적쿼리 작성
: QueryDSL을 사용하면 동적으로 쿼리를 생성할 수 있다.   
예를 들어 사용자가 선택한 필터 조건에 따라 쿼리르 생성하거나 정렬 순서를 바꿀 수 있다.

4. 다양한 데이터 베이스 지원
: QueryDSL은 다양한 데이터 베이스 시스템을 지원하며, 데이터베이스에 종속되지 않는 코드를 작성할 수 있다.

5. JPA 및 Hibernate 통합
: QueryDSL은 Java Persistence API(JPA)와 Hibernate와 같은 ORM(Object-Relational Mapping)프레임워크와 통합되어 사용될 수 있어, 객체 지향 데이터베이스 엑세스를 더 쉽게 구현할 수 있다.

6. 코드 가독성과 유지 보수성
: QueryDSL을 사용하면 SQL 문자열을 작성하는 대신 자바코드로 쿼리를 작성할 수 있으므로 코드 가독성이 향상되고 유지보수가 쉬워진다.

---
#### 프로젝트 생성
- spring web
- spring data jpa
- h2 database
- lombok

*Querydsl은 start.spring.io의 Dependencies에는 없기 때문에 별도로 추가해야 한다.(버전관리는 해준다)

[gralde 8.x, spring 2.x 버전 gradle 설정방법 참고](https://data-make.tistory.com/728)

**Querydsl은 Qtype이라는 클래스를 생성한다.**  
intelliJ에서 gradle > other > compileQuerydsl 명령을 실행하면 gralde.build에 설정된 위치에 QType class가 생성됨.

Entity class로 QType Class을 만든다. 
```
@Entity
class Member { ... }

// compileQuerydsl 명령으로 생성된 QType class => QMember.class
```
- 생성된 QType Class는 버전관리를 하지 않는 것을 권장(컴파일 시점에 자동 생성되므로)

테스트 코드 작성
- JPAQueryFactory 사용

---
#### JPQL vs Querydsl
- Querydsl은 JPAQueryFactory를 사용한다.
- 파라미터 바인딩시 PrepareStatement를 활용하여 파라미터를 바인딩하는 방식이기 떄문에 sql injection등에 안전하다.
- JPQL은 컬럼명을 잘못적으면 실행시점에 발견할 수 있다. 반면에 Querydsl은 컴파일시점에 발견할 수 있다.

Member 테이블에서 username이 "member1"인 쿼리 조회 예시
```
@Test
public void startQuerydsl() {
    JPAQueryFactory queryFactory = new JPAQueryFactory(em);
    QMember m = new QMember("m");
    
    Member findMember = queryFactory
                            .select(m)
                            .from(m)
                            .where(m.username.eq("member1"))
                            .fetchOne();
                            
    assertThat(findMember.getUsername()).isEqualTo("member1");
}
```
- JPAQueryFactory는 인스턴스 변수로 생성하여 사용하는 것을 권장  
- 스프링 프레임워크는 여러 쓰레드에서 동시에 같은 EntityManager에 접근해도, 트랜잭션마다 별도의 영속성 컨텍스트를 제공하기 때문에, 동시성 문제는 걱정하지 않아도 된다.

---
#### 기본 Q-Type 활용
- QType class는 static import로 사용하면 코드를 줄일 수 있다.  
- Querydsl로 작성한 코드는 내부적으로 JPQL로 변경되어 실행된다.  
- QTypeclass 생성시 생성자에 넘기는 파라미터 값은 쿼리 실행 시 엔티티의 alias 값이다. (셀프 조인할 때 사용하고 나머지는 기본으로 생성된 값을 사용하면 된다.)

생성된 JPQL을 보려면 application.yml 설정에 다음을 추가하면 된다.
```
spring:
  jpa:
    properties:
      hibernate:
        use_sql_comments: true
```

---
#### 검색조건 쿼리
- select(), from()을 한번에 사용하는 selectFrom()도 가능하다.
- Querydsl 조건 검색 시 JPQL에서 제공하는 모든 검색 조건을 제공한다.(eq, like, ne, isNotNull 등..)
- 검색조건을 메서드 체이닝 방식으로 연결할수 있다.
- where절에 메서드를 파라미터로 넘기는 방식도 있다.(and만 있는 경우) 파라미터로 검색조건을 추가하면 AND 조건이 추가된다.

---
#### 결과조회
- fetchResults 메서드는 deprecated 되었으므로 content 데이터는 fetch(), totalCount는 fetchOne()을 사용 해야한다.
```
List<MemberTeamDto> content = queryFactory
        .select(
                new QMemberTeamDto(
                        member.id.as("memberId"),
                        member.username,
                        member.age,
                        team.id.as("teamId"),
                        team.name.as("teamName")))
        .from(member)
        .leftJoin(member.team, team)
        .where(
                usernameEq(condition.getUsername()),
                teamNameEq(condition.getTeamName()),
                ageGoe(condition.getAgeGoe()),
                ageLoe(condition.getAgeLoe())
        )
        .offset(pageable.getOffset())
        .limit(pageable.getPageSize())
        .fetch();

Long countQuery = queryFactory
        .select(member.count())
        .from(member)
        .leftJoin(member.team, team)
        .where(
                usernameEq(condition.getUsername()),
                teamNameEq(condition.getTeamName()),
                ageGoe(condition.getAgeGoe()),
                ageLoe(condition.getAgeLoe())
        )
        .fetchOne();
```

---
#### 정렬
- 테스트 코드 작성

회원 나이 내림차순(desc), 이름 올림차순(asc)이고 만약 null인 경우 마지막에 출력하는 테스트 코드 예시
```
@Test
void sort() {
    em.persist(new Member(null, 100));
    em.persist(new Member("member5", 100));
    em.persist(new Member("member6", 100));
    
    List<Member> result = queryFactory
                            .selectFrom(member)
                            .where(member.age.eq(100))
                            .orderBy(member.age.desc(), member.username.asc().nullsLast())
                            .fetch();
    
    Member member5 = result.get(0);
    Member member6 = result.get(1);
    Member memberNull = result.get(2);
    
    assertThat(member5.getUsername()).isEqualTo("member5");
    assertThat(member6.getUsername()).isEqualTo("member6");
    assertThat(memberNull.getUsername()).isNull();
}
```

---
#### 집합
- avg, count, max, min 등의 함수를 select로 조회하면 Tuple로 반환된다.(Querydsl에서 제공하는 객체)
- 단일타입이 아닌 여러가지 타입으로 조회하는 경우 Tuple을 사용하면 된다.(자주 사용하지는 않는다)

집합함수 예시
```
@Test
void aggregation() throws Exception {
    List<Tuple> result = queryFactory
                            .select(member.count(),
                            member.age.sum(),
                            member.age.avg(),
                            member.age.max(),
                            member.age.min())
                            .from(member)
                            .fetch();
    Tuple tuple = result.get(0);
    
    assertThat(tuple.get(member.count())).isEqualTo(4);
    assertThat(tuple.get(member.age.sum())).isEqualTo(100);
    assertThat(tuple.get(member.age.avg())).isEqualTo(25);
    assertThat(tuple.get(member.age.max())).isEqualTo(40);
    assertThat(tuple.get(member.age.min())).isEqualTo(10);
}
```
- 차례대로 회원수(count), 회원나이의 합(sum), 회원나이의 평균(avg), 회원나이의 최대값(max), 회원나이의 최소값(min)을 구한다.

GroupBy 사용 예시
```
@Test
void group() throws Exception {
    List<Tuple> result = queryFactory
                            .select(team.name, member.age.avg())
                            .from(member)
                            .join(member.team, team)
                            .groupBy(team.name)
                            .fetch();
    Tuple teamA = result.get(0);
    Tuple teamB = result.get(1);
    
    assertThat(teamA.get(team.name)).isEqualTo("teamA");
    assertThat(teamA.get(member.age.avg())).isEqualTo(15);
    assertThat(teamB.get(team.name)).isEqualTo("teamB");
    assertThat(teamB.get(member.age.avg())).isEqualTo(35);
}
```
- Team의 이름으로 group by를 하고 각 팀별 회원의 평균 나이를 구했다.