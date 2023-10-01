### 2023.10.01

이 내용은 김영한님의 "실전! Querydsl" 강의를 기반으로 정리하였습니다.

----

#### 조인-기본조인
- 기본 조인의 문법은 첫번째 파라미터에 조인 대상을 지정하고, 두번째 파라미터에 별칭으로 사용할 Q타입을 지정하면 된다.

```
join(조인대상, 별칭으로 사용할 Q타입)
```

**기본조인의 예시**
```
@Test
void join() {
    QMember member = QMember.member;
    QTeam team = QTeam.team;
    
    List<Member> result = query.selectFrom(member)
            .join(member.team, team)
            .where(team.name.eq("teamA"))
            .fetch();
    
    assertThat(result)
            .extracting("username")
            .containsExactly("member1", "member2");
}
```
- 기본 inner join 뿐만아니라 left join, right join도 가능하다.
- 엔티티간의 연관 관계가 없어도 조인이 가능하다.(theta join)

**세타조인 예시 - 연관관계가 없는 필드로 조인**
```
/**
 * 세타 조인(연관관계가 없는 필드로 조인)
 * 회원의 이름이 팀 이름과 같은 회원 조회
 */
@Test
void theta_join() {
    em.persist(new Member("teamA"));
    em.persist(new Member("teamB"));
    
    List<Member> result = query.select(member)
            .from(member, team)
            .where(member.username.eq(team.name))
            .fetch();
    
    assertThat(result)
            .extracting("username")
            .containsExactly("teamA", "teamB");
}
```
- from 절에 조인할 엔티티를 파라미터로 넘긴다.

---
#### 조인 - on절
- on절을 활용한 조인은 JPA 2.1부터 지원한다. 
- 조인 대상을 필터링할 때 사용한다.


**조인대상 필터링 예시**
```
/**
 * 회원과 팀을 조인하면서, 팀 이름이 teamA인 팀만 조인, 회원은 모두 조회(left join)
 */
@Test
void join_on_filtering() {
    List<Tuple> result = query.select(member, team)
            .from(member)
            .leftJoin(member.team, team).on(team.name.eq("teamA"))
            .fetch();
    
    for (Tuple tuple : result) {
        System.out.println("tuple = " + tuple);
    }
}
``` 
- 출력값을 보면 team 이름이 teamA인 회원만 팀정보가 조회되고 나머지는 team정보가 null로 출력된다.
- 내부조인을 사용하는 경우 where절을 사용하는 것과 동일하기 때문에 내부조인을 사용하는 경우 where절로 풀면 된다.


**연관관계 없는 엔티티를 외부 조인하는 예시**
```
/**
 * 연관관계 없는 엔티티 외부 조인
 * */
@Test
void join_on_no_relation() {
    em.persist(new Member("teamA"));
    em.persist(new Member("teamB"));
    
    List<Tuple> result = query.select(member, team)
            .from(member)
            .leftJoin(team).on(member.username.eq(team.name))
            .fetch();
    
    for (Tuple tuple : result) {
        System.out.println("tuple = " + tuple);
    }
}
``` 
- 연관관계 없는 엔티티 외부조인시 연관관계 엔티티 조인 방법과 문법이 살짝 다르다. (일반 조인과 다르게 엔티티 한 개만 들어간다.)

---
#### 조인 - 페치조인
- 페치조인은 SQL에서 제공하는 기능은 아니다. SQL조인을 활용해서 연관된 엔티티를 SQL한번에 조회하는 기능으로 성능 최적화에 사용하는 방법이다.
- EntityManagerFactory를 사용하면 로딩여부를 판단할수 있는 메서드를 제공한다(isLoaded)

**페치조인 적용 예시**
```
@Test
void fetchJoin() {
    em.flush();
    em.clear();
    
    Member findMember = query
            .selectFrom(member)
            .join(member.team, team).fetchJoin()
            .where(member.username.eq("member1"))
            .fetchOne();
    
    boolean loaded = emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
    
    assertThat(loaded).as("페치 조인 적용")
            .isTrue();
}
```

---
#### 서브쿼리
- 서브쿼리는 "com.querydsl.jpa.JPAExpressions"를 사용한다.

**서브쿼리 사용예시**
```
/**
 * 나이가 가장 많은 회원 조회
 */
@Test
void subQuery() {
    QMember memberSub = new QMember("memberSub");
    
    List<Member> result = query
            .selectFrom(member)
            .where(member.age.eq(
                    select(memberSub.age.max())
                            .from(memberSub))
            ).fetch();
    
    assertThat(result.size()).isEqualTo(1);
    assertThat(result).extracting("username").containsExactly("member4");
    assertThat(result).extracting("age").containsExactly(40);
}
```

- 서브쿼리절의 QType class alias는 달라야하기 때문에 새로 생성해서 사용해야 한다.
- select, where절에 가능하지만 from절에는 서브쿼리(인라인 뷰)가 안된다.
  이를 해결하기 위해서는 1. 서브쿼리를 join으로 변경하거나, 2. 애플리케이션에서 쿼리를 2번 분리해서 실행, 3. nativeSQL을 사용하는 방법이 있다.
- from절에 서브쿼리를 사용하는 경우에 화면에 맞추기 위한 쿼리를 작성하는 경우가 있는데 화면에 데이터를 맞추는 것은 프레젠테이션계층에서 하는 것을 추천한다.
- 쿼리를 여러번 나눠서 호출하는 것이 낫다.

---
#### Case문
**간단한 case문 예시**
```
List<String> result = query
                .select(member.age
                        .when(10).then("열살")
                        .when(20).then("스무살")
                        .otherwise("기타")
                )
                .from(member)
                .fetch();
```

**복잡한 case문 예시**
```
List<String> result = query
                .select(new CaseBuilder()
                        .when(member.age.between(0, 20)).then("0~20살")
                        .when(member.age.between(21, 30)).then("21~30살")
                        .otherwise("기타"))
                .from(member)
                .fetch();
```

**orderBy에서 Case문과 함께 사용 예시**
```
NumberExpression<Integer> rankPath = new CaseBuilder()
                .when(member.age.between(0, 20)).then(2)
                .when(member.age.between(21, 30)).then(1)
                .otherwise(3);

List<Tuple> result = query
		.select(member.username, member.age, rankPath)
		.from(member)
		.orderBy(rankPath.desc())
		.fetch();
```
- case문을 변수로 빼고 해당 case문 변수를 orderBy절에서 사용할 수 있다.


---
#### 상수, 문자 더하기
- 상수가 필요한 경우 Expressions.constant(xxx)를 사용하면된다.

**상수 사용 예시**
```
Tuple result = query
                .select(member.username, Expressions.constant("A"))
                .from(member)
                .fetchFirst();
```
- JPQL 쿼리 로그에는 나오지 않는다.

```
select member1.username
from Member member1 */ select member0_.username as col_0_0_ from member member0_ limit 1;
```

**문자 더하기 - stringValue() 사용 예시**
```
String result = query
                .select(member.username.concat("_").concat(member.age.stringValue()))
                .from(member)
                .where(member.username.eq("member1"))
                .fetchOne();
```

**실행 결과값**
```
result = member1_10
```
- 문자가 아닌 다른 타입들을 처리할 때 stringValue() 메서드를 사용하는데 Enum같은 데이터를 처리할 때 자주 사용한다.
