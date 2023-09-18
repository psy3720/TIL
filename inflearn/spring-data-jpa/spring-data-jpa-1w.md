### 2023.09.18

이 내용은 김영한님의 "실전! 스프링 데이터 JPA" 강의를 보고 정리하였습니다.

---

#### 순수 JPA 기반 리포지토리 만들기

- 순수 jpa 기반으로 코드를 작성한 후 spring data jpa를 사용하면 어떤 장점이 있는지 확인하는 방식으로 실습 진행
- JPQL은 객체를 대상으로 적용하는 쿼리로 다른 SQL과 조금 다르다.

JPA에는 update가 왜 없는가?

- JPA는 변경감지라는 기능으로 데이터를 바꾼다.
- 자바 컬렉션에서 값을 꺼내와 해당 값을 바꾸면 해당 값이 바뀌어 있는 것처럼 JPA의 엔티티도 마찬가지로 생각하면 된다.
- 엔티티에 대한 변경은 더티체킹(변경감지)를 통해 값이 바뀌게 된다.

---

#### 공통 인터페이스 설정 및 적용

SpringBoot를 사용하면 spring data jpa Configuration 어노테이션을 사용하지 않아도 자동으로 설정된다.

JPA를 구현한 인터페이스는 구현체가 없다?  
개발자가 구현하지 않아도 Spring Data JPA가 구현체를 만들어서 주입해준다.

엔티티 타입과 엔티티의 pk로 지정된 값의 데이터타입으로 구현한다.

```
public interface MemberRepository extends JpaRepository<Member, Long> { 
    ... 
}
```

---

#### 쿼리 메소드 기능

스프링데이터 JPA는 다음과 같은 기능을 제공한다.

- 메소드 이름으로 쿼리 생성
- NamedQuery
- @Query- 레포지토리 메소드에 쿼리 정의
- 파라미터 바인딩
- 반환타입
- 페이징과 정렬
- 벌크성 수정 쿼리
- @EntityGraph

---

#### 1. 메소드이름으로 쿼리 생성 
메서드 이름을 분석해서 JPQL 쿼리를 실행하는 기능을 제공한다.

이름과 나이를 기준으로 회원 조회하는 메서드 작성 예시

 ```
 List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
 ```

* 메서드명이 필드명과 일치하지 않는 경우 에러가 발생하므로 정확히 명시해야 한다.
* 메서드명 작성법은 스프링 공식 사이트에 접속해서 확인할 수
있다.(https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)
* 그러나 두개이상 넘어가는 경우 메소드명이 너무 길어지게되는 문제점이 있다.
* 장점: 필드명이 바뀌는 경우 애플리케이션 로딩 시점에 에러를 발견할 수 있다.
---
#### 2. 메서드이름으로 JPA NamedQuery
@NamedQuery 어노테이션을 사용하여 Entity클래스에 name과 query를 작성한 후 레포지토리의 메서드에 Named 쿼리를 지정하여 호출할 수 있다.

작성 예시  
2.1 entity 클래스에 NamedQuery 어노테이션 명시

```
@Entity
@NamedQuery(
 name="Member.findByUsername",
 query="select m from Member m where m.username = :username")
public class Member { ...}
```

2.2 스프링데이터 JPA를 사용하여 Named 쿼리 호출

```
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByUsername(@Param("username") String username);
}
```

* 실무에서 거의 사용하지 않는다.
* 우선순위는 named query를 찾은 후 없으면 쿼리메서드를 생성한다.
* named query의 장점은 문법 오류를 애플리케이션 실행 시점에 검증해준다.
---
#### 3. @Query, 레포지토리 메소드에 쿼리를 정의
이 어노테이션을 사용하면 리포지토리 메서드에 바로 query를 작성할 수 있다.

```
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query("select m from Member m where m.username = :username and m.age = :age")
    List<Member> findUser(@Param("username") String username, @Param("age") int age);
}
```
* query문법을 잘못 작성한 경우 애플리케이션 실행시점에 에러가 발생하는 장점이 있다.
* 실행할 메서드에 정적 쿼리를 직접 작성하기 때문에 이름없는 Named 쿼리라 할수 있다.
---
#### 4. @Query 값, DTO 조회
단순히 값 하나를 조회하는 경우 다음과 같이 작성하면 된다.

```
@Query("select m.username from Member m")
List<String> findUsernameList();
```

* JPA 값 타입 @Embedded도 이방식으로 조회할 수 있다.

DTO로 직접 조회하는 경우는 다음과 같이 작성하면 된다.
```
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
    List<MemberDto> findMemberDto();
```
* dto조회시 AllArgumentConstructor가 필요하다.
* dto조회시 new operation을 다 적어야 한다.(패키지 위치, 속성..)
---
#### 5. 파라미터 바인딩
* 파라미터 바인딩은 위치기반과 이름기반이 존재한다.
* 위치가 바뀌는 경우 파라미터가 일치하지 않을 수 있으므로 위치기반보다는 이름기반으로 작성하는것을 추천한다.

```
select m from Member m where m.username = ?0 // 위치기반
select m from Member m where m.username = :username // 이름기반
```

컬렉션타입으로 파라미터 바인딩도 가능하다.

```
@Query("select m from Member m where m.username in :names")
List<Member> findByNames(@Param("names") List<String> names);
```
---
#### 6. 반환타입
스프링 데이터 JPA는 유연한 반환타입을 지원한다.

```
List<Member> findByUsername(String name); // 컬렉션
Member findByUsername(String name); // 단건
Optional<Member> findByUsername(String name); // 단건 Optional
```
* 반환타입이 컬렉션인 경우 empty collection을 반환해주지만, 객체인 경우 null을 반환한다.

---
#### 7. JPA페이징과 정렬
7.1 순수 JPA  
페이징 구현  
offset(몇번째부터), limit(몇개를 가져올것인지) 파라미터로 받은 후 JPQL 쿼리를 구현하여 반환하도록 구현  

DB가 바뀌면 어떡하나요?  
jpa는 방언을 기반으로 동작하기 때문에 db가 바뀌어도 유연하게 변경할 수 있다.  

7.2 스프링 데이터 JPA 페이징과 정렬
Page 사용 예제 정의 코드

```
public interface MemberRepository extends Repository<Member, Long> {
	Page<Member> findByAge(int age, Pageable pageable);
}
```

Pageable을 파라미터로 받고, PageRequest.of 메서드를 사용하여 파라미터로 넘긴다.  
* 페이지는 0번째부터 시작한다.

반환타입에는 Page와 Slice가 존재한다.
- Page는 추가 count 쿼리 결과를 포함하는 페이징 객체이다.
- Slice는 추가 count 쿼리(totalCount)를 가져오지 않고,다음 페이지만 확인 가능하다.(내부적으로 limit+1을 조회하여 확인한다.)
* spring data jpa를 사용함으로써 반복적인 작업을 줄여주기 때문에 개발 생산성이 증가하게 된다.
(paging -> slice 방식으로 변경시 등..)

```
@Query(value = "select m from Member m",
	countQuery = "select count(m.username) from Member m")
Page<Member> findMemberAllCountBy(Pageable pageable);
```

* totalCount 쿼리는 조인이 필요하지 않은 경우도 있기 때문에, content query와 totalcount query를 분리할 수 있다.

```
Page<Member> page = memberRepository.findByAge(10, pageRequest);
Page<MemberDto> dtoPage = page.map(m -> new MemberDto());
```
* page객체를 Controller에서 바로 반환하면 안된다. (엔티티를 그대로 노출하는 것은 보안상 위험, dto 클래스로 변환한 후 반환하면 된다.)  
map 메서드를 사용하여 객체르 변환하면 된다.

---
#### 벌크성 수정쿼리

모든직원의 연봉을 10%인상하기는 쿼리1번으로 날리는게 효율적이다.
(jpa는 더티체킹시 commit시점에 1건씩 처리한다.)

벌크성 수정쿼리는 여러개의 엔티티를 한번에 수정하는 쿼리를 말한다.

스프링 데이터 JPA를 사용한 벌크성 수정쿼리 예시

```
@Modifying
@Query("update Member m set m.age = m.age + 1 where m.age >= :age")
int bulkAgePlus(@Param("age") int age);
```

- 벌크성 수정,삭제 쿼리는 @Modifying 어노테이션을 사용한다.

*벌크연산시 주의사항  
- 영속성컨텍스트에는 남기지 않고 db에 날리기 때문에 영속성컨텍스트는 변경된 내용을 알지 못한다.따라서 영속성컨텍스트를 비워줘야 한다.
- 벌크연산 이후에 flush메서드를 사용하여 db에 반여되지 않는 반영한 후 clear 메서드를 사용하여 비운다
- 만약 api가 끝나지 않고 같은 트랜잭션에 다른 작업을 수행하는 경우 반영되지 않은 값을 사용할수있다.
- @Modifying의 clearAutomatically를 사용하면 쿼리가 나간후 clear를 자동으로 실행해준다.
- mybatis같은 SQL Mapper같은 기술과 함께 사용하는 경우에 mybatis에서 날린 쿼리 내용이 영속성 컨텍스트에 반영되지 않을 수 있기 때문에 clear작업등을 고려해야 한다.
---
#### @EntityGraph
연관된 엔티티들을 SQL 한번에 조회할때 사용할 수 있다.

사전지식
- 이걸 이해하기 위해서는 fetchJoin을 알아야 한다.
  실무에서는 데이터 로딩 전략을 사용시점에 가져오는 LAZY로 사용하게 된다.  
- N+1 문제, 연관관계에 있는 엔티티를 조회할 때 N개만큼 더 조회하는 문제

JPA에서는 Fetch Join을 통해 해결  
fetch join을 사용하면 1을 조회할때 연관된 n을 함께 조회한다.(예제에서는 Member를 조회할때, Team을 함께 조회한다.)

JPQL 페치 조인 예시
```
@Query("select m from Member m left join fetch m.team")
List<Member> findMemberFetchJoin();
```

엔티티 그래프 기능을 사용하면 JPQL 없이 페치조인을 사용할 수 있다.(JPQL + EntityGraph 조합하여 사용 가능하다.)
```
@Override
@EntityGraph(attributePaths = {"team"})
List<Member> findAll();

@EntityGraph(attributePaths = {"team"})
@Query("select m from Member m")
List<Member> findMemberEntityGraph();   

@EntityGraph(attributePaths = {"team"})
List<Member> findByUsername(String username);
```

EntityGraph는 fetch join의 간편 버전이라고 이해하면 된다.

---

#### JPA Hint & Lock

SQL Hint가 아니라, JPA 구현체에게 제공하는 힌트

쿼리힌트 사용예시

```
@QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value =
"true"))
Member findReadOnlyByUsername(String username);
```

- 변경감지 기능의 단점은 원본을 가지고 비교하기 때문에 2개가 필요하다(변경된 정보, 원본 정보)따라서 비용이 든다.
- 쿼리힌트 어노테이션을 사용하여 readonly가 true인 경우 내부적으로 snapshot을 만들지 않는다.(변경감지 체크자체를 하지 않는다.)