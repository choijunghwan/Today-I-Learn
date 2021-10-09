# Entity Listener란?
엔티티를 DB에 적용하기 이전 이후에 커스텀 콜백을 요청할 수 있는 어노테이션입니다. 

Entity Listener를 이용해서 Entity에 자동으로 값을 넣어줄 수 있습니다.

주로 등록시간, 수정시간 같은 값을 자동으로 넣어줄 때 사용합니다.

### Entity Listener / 콜백옵션
* @PrePersist
  * manager persist 의해 처음 호출될 때 실행됩니다.
* @PostPersist
  * manager persist 에 의해 실행되고 불립니다.
  * SQL INSERT 이후에 대응될 수 있습니다.
* @PostLoad
  * 로드 이후에 불립니다.
  * SQL SELECT 이후에 대응될 수 있습니다.
* @PreUpdate
  * SQL UPDATE 이전에 불립니다.
* @PostUpdate
  * SQL UPDATE 이후에 불립니다.
* @PreRemove
  * SQL DELETE 이전에 불립니다.
* @PostRemove
  * SQL DELETE 이후에 불립니다.

간단한 예시로 값을 수정할때 Update 쿼리가 나갈때 값을 삽입하는것을 보겠습니다.

**Account객체**
```Java
@Entity
@Getter @Setter
public class Account {

    @Id @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Long id;

    private LocalDateTime testRandom;
}
```
업데이트를 하게되면 testRandom에 자동으로 수정시간이 등록되게 하겠습니다.

**AccountListener**
```Java
public class AccountListener {

    @PreUpdate
    void onUpdate(Account account) {
        account.setTestRandom(LocalDateTime.now());
    }
}
```
@PreUpdate - 업데이트 쿼리가 나가기 이전에 값을 삽입해줍니다.

Account 객체에 AccountListener를 등록해줍니다.
**Account 객체 - AccountListener 추가**
```Java
@Entity
@EntityListeners(AccountListener.class)
@Getter @Setter
public class Account {

    @Id @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Long id;

    private LocalDateTime testRandom;
}
```
`@EntityListeners(AccountListener.class)`
* @EntityListeners 를 통해 AccountListener클래스를 추가해줍니다.


