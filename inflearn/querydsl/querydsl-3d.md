### 2023.10.04

이 내용은 김영한님의 "실전! Querydsl" 강의를 기반으로 정리하였습니다.

----
### 중급문법
#### 프로젝션과 결과 반환 - 기본

예시 코드
```
@Test
void tuple() {
    List<Tuple> result = query
            .select(member.username, member.age)
            .from(member)
            .fetch();
    
    for (Tuple tuple : result) {
        String username = tuple.get(member.username);
        Integer age = tuple.get(member.age);
        System.out.println("username = " + username);
        System.out.println("age = " + age);
    }
}
```
- 프로젝션 대상이 하나인 경우 타입을 명확하게 지정할 수 있다.
- 프로젝션 대상이 여러개인 경우 DTO 또는  튜플을 사용한다.
- 튜플을 레포지토리 계층에서 사용하는 것은 괜찮지만 서비스 계층이나 컨트롤러 계층에서는 사용하지 않는 것을 추천

---
#### 프로젝션과 결과 반환 - DTO 조회
- 순수 JPA에서 DTO를 조회할 때는 new operation 연산자를 사용하여 패키지명을 다 적어야 하기 때문에 코드가 지저분하다.(순수 JPA는 생성자 방식만 지원한다)
- Projections를 사용해서 DTO를 조회할 수 있다.
- DTO 조회 방식에는 프로퍼티, 생성자, 필드 직접 접근방식이 있다.

**프로퍼티 접근 예시**
```
List<MemberDto> result = query.select(Projections.bean(MemberDto.class, member.username, member.age))
		.from(member)
		.fetch();
```
- Entity와 DTO class의 필드명이 다른 경우 as(alias)를 사용하여 필드명을 매핑 해줄 수 있다.
```
// member.entity필드명.as(DTO필드명)
member.username.as("name")
```

```
List<UserDto> fetch = query.select(
                        Projections.fields(
                                UserDto.class
                                , member.username.as("name")
                                , ExpressionUtils.as(select(memberSub.age.max())
                                        .from(memberSub), "age"
                                )))
                .from(member)
                .fetch();
```
- 만약 존재하지 않는 필드를 select하고자 하는 경우 ExpressionUtils를 사용하면 된다.(서브쿼리 등)

**생성자 사용 예시**
```
List<MemberDto> result = query.select(
                Projections.constructor(MemberDto.class
                        , member.username.as("name")
                        , member.age))
                .from(member)
                .fetch();
```
- 생성자 방식을 사용하는 경우는 필드명이 아닌 파라미터의 타입을 보고 매핑되기 때문에 필드명이 일치하지 않아도 된다.

---
#### 프로젝션과 결과 반환 - @QueryProjection
- 생성자에 @QueryProjection 어노테이션을 붙이고 compileQuerydsl을 실행하면 DTO QType class가 생성된다.

**@QueryProjection 활용**
```
// 생성된 DTO클래스를 활용한다.
List<MemberDto> result = query
                .select(new QMemberDto(member.username, member.age))
                .from(member)
                .fetch();
```
- 이 방식을 사용하는 것의 장점은 Compile 시점에 에러를 발견할 수 있다.

단점
- Querydsl 어노테이션을 사용한 DTO가 Queydsl에 종속적이게 된다.(만약 Queydsl을 사용하지 않게 된다면 DTO에도 영향이 미친다)
- DTO클래스는 service 계층과 presentation 계층에서도 사용하게 되는데, 계층간에 클래스가 깨끗하게 분리되지 않을 수 있다.

---
#### 동적쿼리
동적쿼리를 해결하는 방식에는 두가지 방법이 있다.
- BooleanBuilder를 사용하는 방법
- Where 다중 파라미터를 사용하는 방법

#### 동적쿼리 - BooleanBuilder 사용
**사용 예시**
```
@Test
void 동적쿼리_BooleanBuilder() {
    String usernameParam = "member1";
    int ageParam = 10;
    
    List<Member> result = searchMember1(usernameParam, ageParam);
    
    assertThat(result.size()).isEqualTo(1);
}

private List<Member> searchMember1(String usernameCond, Integer ageCond) {
    BooleanBuilder booleanBuilder = new BooleanBuilder();
    
    if (usernameCond != null) {
        booleanBuilder.and(member.username.eq(usernameCond));
    }
    
    if (ageCond != null) {
        booleanBuilder.and(member.age.eq(ageCond));
    }
    
    return query
            .selectFrom(member)
            .where(booleanBuilder)
            .fetch();
}
```
- BooleanBuilder를 사용하여 조건문에 사용될 값을 and 메서드를 사용하여 where절의 쿼리를 생성하면 된다.

---
#### 동적쿼리 - Where 다중 파라미터 사용
**사용예시**
```
@Test
void 동적쿼리_WhereParam() {
    String usernameParam = "member1";
    Integer ageParam = 10;
    
    List<Member> result = searchMember2(usernameParam, ageParam);
    
    assertThat(result.size()).isEqualTo(1);
}

private List<Member> searchMember2(String usernameCond, Integer ageCond) {
    return query
            .selectFrom(member)
            .where(usernameEq(usernameCond), ageEq(ageCond))
            .fetch();
}

private BooleanExpression ageEq(Integer ageCond) {
    return ageCond == null ? null : member.age.eq(ageCond);
}

private BooleanExpression usernameEq(String usernameCond) {
    return usernameCond == null ? null : member.username.eq(usernameCond);
}
```
- where절에 null 파라미터를 넘기는 경우 해당 값은 무시된다.
- 메서드는 BooleanExpression으로 하는게 좋다.(조합하여 사용하기 위해서)

**조합 예시**
```
private BooleanExpression allEq(String usernameCond, Integer ageCond) {
    return usernameEq(usernameCond).and(ageEq(ageCOnd));
}
```
- 메서드로 조건을 생성하는 것은 다른 곳에서 조합하여 재사용할 수 있다는 장점이 있다.
- 쿼리 자체의 가독성이 높아진다.

---
#### 수정,삭제 벌크 연산
수정,삭제 벌크 연산은 여러건의 변경을 한번에 처리하고자 할때 사용한다.(JPA의 변경감지 기능은 건건이 처리한다.)

**벌크 수정예시**
```
@Commit
@Test
void bulk() {
	long count = query
			.update(member)
			.set(member.username, "비회원")
			.where(member.age.lt(28))
			.execute();
}
```
- H2 Database에 변경결과를 확인하기 위해 @Commit 어노테이션을 사용했다.
- 나이가 28보다 작은 회원의 이름을 비회원으로 변경한다.(member1, member2)
- 쿼리 실행의 결과로 반환되는 값은 변경된 데이터 개수를 반환한다.

**모든 행의 기존 나이에 1을 더하는 예시**
```
long count = query
                .update(member)
                .set(member.age, member.age.add(1))
                .execute();
```
- minus 메소드는 없으므로 add 메서드에 파라미터로 음수 값을 넘기면 뺀 것과 같은 결과를 실행할 수 있다.

**데이터 삭제 예시**
```
long count = query
		.delete(member)
		.where(member.age.gt(18))
		.execute();
```

**주의사항**
- 주의해야 할 점은 DML 쿼리를 실행한 후 변경된 데이터가 영속성 컨텍스트에 들어있는 데이터와 DB 데이터가 다를 수 있다.
  (DB에서 데이터를 조회해 오는 값과 영속성 컨텍스트의 값을 갱신할 때에는 영속성 컨텍스트의 데이터가 우선권을 가진다.)
- 따라서 DML 연산을 한 이 후 영속성 컨텍스트를 초기화 하는 것을 권장한다.(flush() 및 clear(), @Modifying 등..)

---
#### SQL function 호출하기

**replace 함수 사용 예시**
```
String result = query
                .select(Expressions.stringTemplate("function('replace', {0}, {1}, {2})", member.username, "member", "M"))
                .from(member)
                .fetchFirst();
```
- Dialect(방언)에 등록된 function만 호출가능하다.

**내장 함수 사용 예시(upper)**
```
String result = query
                .select(member.username.upper())
                .from(member)
                .fetchFirst();
```
- 대부분의 DB에서 사용하는 Function은 Queydsl에서 제공한다.