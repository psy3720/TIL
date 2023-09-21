### 2023.09.21

이 내용은 김영한님의 "실전! 스프링 데이터 JPA" 강의를 보고 정리하였습니다.

---

#### 확장기능
##### 사용자 정의 리포지토리 구현
- 스프링 데이터 JPA 레포지토리는 인터페이스만 정의하고 구현체는 스프링이 자동으로 생성해준다.
- 스프링 데이터 JPA가 제공하는 인터페이스를 직접 구현하면 구현해야하는 기능이 너무 많다.
- 만약 다양한 이유로 인터페이스의 메서드를 직접 구현하고자 한다면?
  - JPA 직접 사용(EntityManage)
  - 스프링 JDBC Template 사용
  - Mybatis 사용
  - 데이터베이스 커넥션 직접 사용등..
  - Querydsl 사용

별도의 커스텀 인터페이스를 만들고, 구현체를 만들고, jpa repository에서 커스텀인터페이스를 상속받도록 한다.  
따라서 커스텀 인터페이스의 구현체는 별도의 클래스에서 구현하게 된다.

**사용자 정의 인터페이스 예시**
```
public interface MemberRepositoryCustom {
    List<Member> findMemberCustom();
}
```

**사용자 정의 인터페이스 구현 클래스 예시**
```
@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom {
    private final EntityManager em;
    
    @Override
    public List<Member> findMemberCustom() {
        return em.createQuery("select m from Member m")
        .getResultList();
    }
}
```

**사용자 정의 인터페이스 상속**
```
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
    ... 
}
```

* xxRepositoryImpl 이름 규칙 관례만 맞추면 된다.
* spring data 2.x부터는 사용자 정의 구현 클래스명을 커스텀 인터페이스명 + Impl을 지원한다.
* 메서드명만으로 해결되지 않는 경우 커스텀 리포지토리를 구현한다.


[참고]  
화면에 맞춘 쿼리와 핵심비즈니스 로직 인터페이스를 쪼개는것이 복잡성을 줄여준다.

---
#### Auditing
- 엔티티를 생성, 변경할때 변경한 사람과 추적하고 싶은 경우(ex. 등록일,수정일,등록자,수정자)
- 순수 JPA를 사용할때 (@PrePersist, @PreUpdate, @PostPersist, @PostUpdate 등 사용)

**스프링 데이터 JPA 사용시**  
@EnableJpaAuditing -> 스프링 부트 설정 클래스에 적용
@EntityListeners(AuditingEntityListener.class) -> 엔티티에 적용

사용 어노테이션
- @CreatedDate
- @LastModifiedDate
- @CreatedBy
- @LastModifiedBy

**스프링 데이터 Auditing 적용 예시**
```
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
@Getter
public class BaseEntity {
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;
    
    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
}
```

**AuditorAware 스프링 빈 등록 예시**
```
@EnableJpaAuditing
@SpringBootApplication
public class DataJpaApplication {
    public static void main(String[] args) {
        SpringApplication.run(DataJpaApplication.class, args);
    }
    
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.of(UUID.randomUUID().toString());
    }
}
```
* DataJpaApplication에 @EnableJpaAuditing도 함께 등록해야 한다.
* 실무에서는 세션 정보나 스프링 시큐리티 로그인 정보에서 ID를 받는다.
* 등록일, 수정일은 필요하지만 등록자, 수정자가 필요하지 않은 경우에는 별도의 클래스로 구현한 후 상속받는 형태로 분리하여 사용한다.

---
#### Web확장 - 페이징과 정렬
스프링 데이터가 제공하는 페이징과 정렬 기능을 스프링 MVC에서 편리하게 사용할 수 있다.

```
@GetMapping("/members")
public Page<Member> list(Pageable pageable) {
    Page<Member> page = memberRepository.findAll(pageable);
    return page;
}
```

* global 설정은 yml 파일에 지정 가능
* 파라미터로 Pageable을 받는 경우 스프링부트에서 내부적으로 PageRequest 객체를 생성한다.
```
@RequestMapping(value ="/members_page", method=RequestMethod.GET)
public String list(@PageableDefault(size = 12, sort = "username", direction = Sort.Direction.DESC) Pageable pageable) {
    ...
}
```
메서드에만 개별설정을 적용하고자할때는 @PageableDefault 어노테이션을 사용한다.

---
#### Page 내용을 DTO로 변환하기
엔티티를 API로 노출하면 다양한 문제가 발생한다.
- 내부설계노출
- 엔티티클래스 수정시 api 스펙이 바뀐다.

따라서 엔티티를 dto클래스로 변환해서 반환해야 한다.
Page에 map()메서드를 사용하여 dto클래스로 변환하면 된다.

Page.map() 사용 예시
```
@GetMapping("/members")
public Page<MemberDto> list(Pageable pageable) {
    return memberRepository.findAll(pageable).map(MemberDto::new);
}
```
---
#### 스프링 데이터 JPA 구현체 분석
- 스프링 데이터 JPA가 제공하는 공통 인터페이스의 구현체 org.springframework.data.jpa.repository.support.SimpleJpaRepository
- 하부 구현기술을 바꿔도 비즈니스로직에 영향이 가지 않도록 설계되어 있다.
- 스프링 데이터 JPA는 변경(등록, 수정, 삭제) 메서드를 트랜잭션 처리한다.
- 따라서 스프링 데이터 JPA를 사용할때 트랜잭션이 없어도 리포지토리 계층에 트랜잭션이 전파되어 트랜잭션 어노테이션을 따로 사용하지 않아도 데이터 등록,변경이 가능했다.
- merge는 db에서 select 쿼리를 한번 조회한 후 동작한다.
  (merge 메소드는 가급적 사용하지 않는 것이 좋다. 엔티티의 변경은 변경감지(더티체킹)로 동작하게 하는게 정석이다.)
- merge를 사용하는 경우는 영속상태를 벗어난 후 다시 영속상태에 넣을때 사용한다.(detected 상태에서 다시 영속상태로 관리하고자 할 때 등..)

@Transactional(readOnly = true)
- 데이터를 단순히 조회만 하고 변경하지 않는 트랜잭션에서는 readonly 옵션을 true로 사용하면 flush() 메서드를 생략하기 때문에 약간의 성능향상을 얻을 수 있다.

#### 새로운 엔티티를 구별하는 방법
- save() 메서드는 새로운 엔티티면 저장(persist), 새로운 엔티티가 아니면 병합(merge)이 동작한다.
- 새로운 엔티티를 구별하는 전략은 식별자가 객체일때는 null로 판단하고,
  기본타입(primitive type)인 경우 0으로 판단한다.
- 만약 생성자로 id값을 주입하는 경우, (@GeneratedValue를 사용하지 않고) persist가 호출되지 않고, merge가 호출된다.
- 만약 ID값을 임의로 만들어야 하는 경우 Persistable이라는 인터페이스를 상속받아 사용하면 된다.(LocalDateTime 필드로 판단할 수 있다.)

Persistable 구현 예시
```
import jakarta.persistence.Entity;
import jakarta.persistence.EntityListeners;
import jakarta.persistence.Id;
import lombok.AccessLevel;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.domain.Persistable;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.LocalDateTime;

@Entity
@EntityListeners(AuditingEntityListener.class)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Item implements Persistable<String> {

    @Id
    private String id;

    @CreatedDate
    private LocalDateTime createdDate;

    public Item(String id) {
        this.id = id;
    }

    @Override
    public String getId() {
        return id;
    }

    @Override
    public boolean isNew() {
        return createdDate == null;
    }
}
```
- 등록시간 필드를 조합해서 사용하면 이 필드로 새로운 엔티티 여부를 편리하게 확인할 수 있다.(@CreatedDate에 값이 없으면 새로운 엔티티로 판단하면 된다.)
---
##### Proejections
- 엔티티 대신에 DTO를 편리하게 조회할 때 사용한다.
- 전체 엔티티가 아닌 회원 이름 같은 특정필드 한개만 조회하고자 할 때

```
public interface UsernameOnly {
	String getUsername();
}
```
조회할 엔티티의 필드를 getter 형식으로 지정하면 해당 필드만 선택해서 조회한다.(Projection)

```
public interface MemberRepository ... {
	List<UsernameOnly> findProjectionsByUsername(String username);
}
```
- 메서드 이름은 자유로 작성하면 되고, 반환타입으로 인지한다.
- 프로퍼티 형식(getter)의 인터페이스를 제공하면, 구현체는 스프링 데이터 JPA가 제공한다.
- 인터페이스 기반 뿐만 아니라 클래스 기반 Projection도 가능하다.
- Generic type을 사용하여 동적 Projection으로 데이터를 변경할 수 있다.

클래스 기반 Projection 예시
```
public class UsernameOnlyDto {
    private final String username;
    
    public UsernameOnlyDto(String username) {
        this.username = username;
    }
    
    public String getUsername() {
        return username;
    }
}	
```

**동적 Projections 예시**
```
<T> List<T> findProjectionsByUsername(String username, Class<T> type);
```

```
List<UsernameOnly> result = memberRepository.findProjectionsByUsername("m1", UsernameOnly.class);
```

- 프로젝션 대상이 root 엔티티면 jpql select 절 최적화가 가능하지만 root 엔티티가 아니라면 모든 필드를 select해서 엔티티로 조회한 다음에 계산한다.
- 따라서 복잡한 쿼리를 해결하기에는 한계가 있다.

---
#### 네이티브 쿼리
- 가급적 네이티브 쿼리는 사용하지 않는것이 좋다.
- 타입 및 제약이 있어 비추천
- DTO로 조회할땐느 JdbcTemplate 또는 myBatis를 사용하는것을 권장한다.
- Projection과 함께 사용하면 네이티브 쿼리 타입 한계를 풀 수 있다.




 