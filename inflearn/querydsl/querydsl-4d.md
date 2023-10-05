### 2023.10.05

이 내용은 김영한님의 "실전! Querydsl" 강의를 기반으로 정리하였습니다.

----
#### 순수 JPA 리포지토리와 Querydsl

```
@Bean
JPAQueryFactory jpaQueryFactory(EntityManager em) {
    return new JPAQueryFactory(em);
}
```
- JPAQueryFactory 객체는 빈으로 등록해서 사용하거나 생성자에서 객체를 생성해도 된다.
  (각각의 장단점 존재, @RequiredArgsConstructor, 테스트 코드 등)
- 참고로 멀티쓰레드 환경에서 EntityManager와 JPAQueryFactory 동시성 문제는 걱정하지 않아도 된다. 왜냐하면 스프링이 주입해주는 엔티티 매니저는 시작 시점에 진짜 엔티티 매니저를 찾아주는 프록시용 가짜 엔티티매니저이다.
- 가짜 엔티티 매니저는 사용 시점에 트랜잭션 단위로 실제 엔티티 매니저를 할당해준다.
  (자세한 내용은 자바 ORM 표준 JPA 책 13.1 트랜잭션 범위의 영속성 컨텍스트를 참고하면 된다.)

---
#### 동적쿼리와 성능최적화 조회 Builder 사용
- string value 검증시 StringUtils 사용
- 동적쿼리 조회시 페이징 쿼리와 같이 사용하는 것을 권장(조건이 없는 경우 전체 데이터를 가져오기 때문에 limit을 걸어 놓는게 좋다)

---
#### 동적쿼리와 성능 최적화 조회 - Where절 파라미터 사용
- 메서드로 구현하는 경우 조합(Composition)이 가능하다는 장점이 있다.

---
#### 조회 API 컨트롤러 개발
- 테스트 데이터를 구분하기 위해 profiles의 active를 test와 local로 구분해서 테스트한다.
```
# main\resources\application.yml
spring:
  profiles:
    active: test
	
---
# test\resources\application.yml
spring:
  profiles:
    active: local	
```

---
#### 스프링 데이터 JPA 리포지토리로 변경
- 기본 작업인 CRUD와 간단한 정적쿼리 메서드등은 spring data jpa의 이름으로 메서드 생성등으로 사용하면 된다.

---
#### 사용자 정의 리포지토리
- 복잡한 기능이나 커스텀한 쿼리를 사용해야 하는 경우 사용한다.
- repository명+Impl은 명명규칙을 따른다.

사용자 정의 리포지토리 구현 순서
1. 사용자 정의 인터페이스(커스텀 인터페이스) 생성한다.
```
public interface MemberRepositoryCustom {

    List<MemberTeamDto> search(MemberSearchCondition condition);
}
```
2. 사용자 정의 인터페이스를 구현한다.(커스텀 인터페이스 구현체 생성, 예제에서는 Querydsl을 사용한 구현체이다.)
```
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    private final EntityManager em;
    private final JPAQueryFactory queryFactory;

    public MemberRepositoryImpl(EntityManager em) {
        this.em = em;
        this.queryFactory = new JPAQueryFactory(em);
    }
	
    ...
}	
```

3. Spring Data JPA repository에 사용자 정의 인터페이스를 상속받는다.(extends)
```
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
    List<Member> findByUsername(String username);
}
```

* 참고로 특화된 조회(특정화면에서만 사용하는 등)인 경우 별도의 조회용 class를 생성해서 구현해도 된다.
---
#### 스프링 데이터 페이징 활용
- fetchResults, fetchCount는 deprecated 되어 별도로 쿼리를 두번 날리면 된다.(content 쿼리, totalCount 쿼리.)

```
@Override
public Page<MemberTeamDto> searchComplex(MemberSearchCondition condition, Pageable pageable) {
    List<MemberTeamDto> content = getContent(condition, pageable);
    JPAQuery<Long> totalCount = getTotalCount(condition);
    
    return PageableExecutionUtils.getPage(content, pageable, totalCount::fetchOne);
}

private List<MemberTeamDto> getContent(MemberSearchCondition condition, Pageable pageable) {
	return queryFactory
			.select(new QMemberTeamDto(
					member.id,
					member.username,
					member.age,
					team.id,
					team.name))
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
}

private JPAQuery<Long> getTotalCount(MemberSearchCondition condition) {
    return queryFactory
            .select(member.count())
            .from(member)
            .leftJoin(member.team, team)
            .where(
                    usernameEq(condition.getUsername()),
                    teamNameEq(condition.getTeamName()),
                    ageGoe(condition.getAgeGoe()),
                    ageLoe(condition.getAgeLoe())
            );
}
```
- PageableExecutionUtils의 getPage 메서드를 사용하면 최적화하여 실행한다.
  (내부 로직에서 count 쿼리가 생략 가능한 경우 생략해서 처리한다.)

---
#### 스프링 데이터 페이징 활용 - 컨트롤러 개발
(localhost:8080/v2/members?**page=0&size=200**)
(localhost:8080/v2/members?**page=0&size=10**)

- 다음과 같이 총 데이터 개수가 100개인 상태에서 총 데이터 개수보다 많이 size를 요청해보면 내부적으로 count query를 생략하는 것을 확인할 수 있다.

```
Hibernate: 
    /* select
        count(member1) 
    from
        Member member1   
    left join
        member1.team as team */ select
            count(member0_.member_id) as col_0_0_ 
        from
            member member0_ 
        left outer join
            team team1_ 
                on member0_.team_id=team1_.team_id
```
---
#### 리포지토리 지원 - QuerydslRepositorySupport

특징
- getQuerydsl().applyPagination()을 사용하면 스프링 데이터가 제공하는 페이징을 Querydsl로 편리하게 반환 가능하다.(하지만 Sort는 오류가 발생하는 치명적인 단점이 존재한다.)
- 메소드 체이닝을 시작할때는 select가 아니라 from부터 시작한다.(3.x 버전 대상으로 만들어져 from부터 시작한다)
- 사용자 정의 리포지토리 구현체에서 상속받아 사용한다.
- 스프링 데이터가 제공하는 QuerydslRepositorySupport가 지닌 한계를 보완한 클래스를 직접 작성할 수 있다.
