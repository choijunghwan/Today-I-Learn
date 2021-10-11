우리가 보통 프로젝트를 진행할때 데이터 이력관리를 위해 해당 데이터를 누가, 언제 생성 또는 수정했는지 기록하기 위해 수정일, 생성일, 수정자, 생성자등을 DB 테이블 필드를 추가합니다.

이 데이터들은 많은 테이블에 사용되기 때문에 Entity에도 필드가 중복되어 들어가고 Entity를 생성하고, 수정할때도 매번 데이터를 입력해줘야 하는 번거로움이 있습니다.

이걸 좀더 편하고 자동화 해주는 기술이 Spring Data에서 제공하는 Auditing입니다.

먼저 사용방법을 보겠습니다.

# Auditing 활성화 하기
가장 먼저 SpringBootApplication에 @EnableJpaAuditing 어노테이션을 추가해줍니다.
```Java
@EnableJpaAuditing
@SpringBootApplication
public class CharlieZipApplication {

	public static void main(String[] args) {
		SpringApplication.run(CharlieZipApplication.class, args);
	}

}
```

<br>

# BaseTimeEntity 생성하기
Auditing이 필요한 Entity에서 상속받을 BaseTimeEntity를 생성합니다.

```Java
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
@Getter
public class BaseTimeEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
}
```
@MappedSuperclass
* Entity에서 Table에 대한 공통 매핑 정보가 필요할 때 부모 클래스에 정의하고 상속받아 해당 필드를 사용합니다.

@EntityListeners(AuditingEntityListener.class)
* EntityListeners는 Entity를 DB에 적용하기 이전, 이후에 커스텀 콜백을 요청할 수 있는 애노테이션입니다.
* AuditingEntityListener는 Entity 영속성 및 업데이트에 대한 Auditing 정보를 캡처하는 JPA Entity Listener
  
@CreatedDate
* 데이터 생성 날짜 자동 저장
  
@LastModifiedDate
* 데이터 수정 날짜 자동 저장

<br>

# Entity에 적용하기
UserEntity에 BaseTimeEntity를 상속받습니다.
```Java
@Getter
@Entity
@NoArgsConstructor(access = PROTECTED)
public class User extends BaseTimeEntity {

    @Id
    @GeneratedValue
    @Column(name = "user_id")
    private Long id;

    private String name;

    public User(String name) {
        this.name = name;
    }
    
    public void changeName(String name) {
        this.name = name;
    }

}
```
이렇게만 하면 생성일과 수정일이 createdDate, lastModifiedDate 컬럼에 insert 됩니다.

그럼 생성자와 수정자도 해보겠습니다.

# @CreatedBy, @ModifiedBy
**BaseEntity 생성**
```Java
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
@Getter
public class BaseEntity extends BaseTimeEntity {

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;
}

```

BaseEntity가 BaseTimeEntity를 상속한걸 볼 수 있는데 이 부분은 뒤에가서 자세히 설명하겠습니다.

**UserEntity에 BaseEntity 적용**
```Java
@Getter
@Entity
@NoArgsConstructor(access = PROTECTED)
public class User extends BaseEntity {

    @Id
    @GeneratedValue
    @Column(name = "user_id")
    private Long id;

    private String name;

    public User(String name) {
        this.name = name;
    }
    
    public void changeName(String name) {
        this.name = name;
    }

}
```
그런데 BaseEntity는 위으 BaseTimeEntity랑은 다른점이 있습니다.
BaseEntity는 생성자, 수정자를 넣어주는건데 JPA에서 생성자와 수정자를 알 수 없기 때문에 개발자가 기준을 정해줘야 합니다.

**AuditorAware 구현체 만들기**
```Java
public class AuditorAwareImpl implements AuditorAware<String> {


    @Override
    public Optional<String> getCurrentAuditor() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

    if (authentication == null || !authentication.isAuthenticated()) {
      return null;
    }

    return ((MyUserDetails) authentication.getPrincipal()).getUser();
    }
}
```
Spring Security의 Authentication을 이용해 값을 지정해주었습니다.

구현체를 만들었으니 @EnableJpaAuditing에 등록해 줍니다.

```Java
@EnableJpaAuditing
@SpringBootApplication
public class CharlieZipApplication {

	public static void main(String[] args) {
		SpringApplication.run(CharlieZipApplication.class, args);
	}

	@Bean
	public AuditorAware<String> auditorProvider() {
		return new AuditorAwareImpl();
	}
}
```

createdBy, lastModifiedBy 칼럼에 생성자와 수정자가 등록됩니다.

# AuditorAware 구현체 - Session 사용
Auditing 기술을 내가 진행하는 프로젝트에 적용을 할려 했더니 기존 AuditorAware 구현체는 Spring Security를 사용하여 사용자 정보를 가져오고 있었다.

그런데 나는 Spring Security를 사용하지 않는 프로젝트를 진행중이었고 그러면 Session에 있는 사용자의 정보를 이용해볼 수는 없을까라는 생각이 들었다.

조금 찾아보니 Spring은 Controller가 아닌 다른 부분에서도 Session 같은 http 헤더정보를 쉽게 접근할수 있게 `RequestContextHolder`라는 것을 제공해주고 있었다.

그럼 `RequestContextHolder`를 이용해 구현해보자.
```Java
public class AuditorAwareImpl implements AuditorAware<String> {

    @Override
    public Optional<String> getCurrentAuditor() {
        ServletRequestAttributes servletRequestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        Member findMember = (Member) servletRequestAttributes.getRequest().getSession().getAttribute(GlobalConst.LOGIN_MEMBER);

        return Optional.ofNullable(findMember.getUsername());
    }
}
```
`RequestContextHolder.getRequestAttributes()`를 이용해 서블릿 Request의 값들을 불러오고 
`servletRequestAttributes.getRequest().getSession().getAttribute(GlobalConst.LOGIN_MEMBER)`을 통해 session에 있는 사용자 정보를 가져왔다.


<br>

# 실제 환경
대부분의 테이블에서 생성일, 수정일은 필요하지만 생성자, 수정자는 필요하지 않은 경우도 있습니다. 그래서 `BaseTimeEntity`와 `BaseEntity`로 나누었습니다.

생성일와 수정일만 사용할때는 `BaseTimeEntity`를 생성자와 수정자까지 모두 필요할때는 `BaseEntity`를 상속받아 사용하면 됩니다.

