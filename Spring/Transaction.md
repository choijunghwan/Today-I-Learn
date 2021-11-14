# 트랜잭션 처리 방법
스프링에서는 간단하게 어노테이션 방식으로 `@Transactional`을 메소드, 클래스, 인터페이스 위에 추가하여 사용할 수 있습니다.

이 방식을 선언전 트랜잭션이라 부르는데, 적용된 범위에서는 자동으로 트랜잭션 기능이 적용됩니다.

<br><br>

# @Transactional 옵션

- propagation : 전파 속성을 정의합니다.
- isolation : 격리수준을 지정합니다.
- timeout : 지정된 시간안에 수행이 완료되지 못하면 rollback한다.
- readonly : 트랜잭션을 읽기 전용으로 설정한다.

<br><br>

# propagaion(전파속성)
전파속성이란 트랜잭션 동작 도중 다른 트랜잭션을 호출할때, 트랜잭션을 범위를 어떻게 처리할지에 대한 옵션입니다.

```java
public class ServiceA {
		private ServiceB serviceB;

		...

		@Transactional
		public void a() {
				serviceB.b();
		}
}

public class ServiceB {
		@Transactional
		public void b() {...}
}
```

위의 코드의 경우 ServiceA의 a()메소드에 트랜잭션이 설정되어 있는데 a()메소드가 동작할때 ServiceB의 b()메소드에 추가로 트랜잭션이 설정되어 있습니다. 이럴경우 트랜잭션이 중첩되어 설정되어있는데 이럴때 어떤 트랜잭션의 범위를 사용할 것인지에 대한 전파 타입을 정해주어야 합니다.

<br>

- REQUIRED(Default)
    - 이미 진행중인 트랜잭션이 있다면 해당 트랜잭션 속성을 따르고, 진행중이 아니라면 새로운 트랜잭션을 생성한다.
- REQUIRES_NEW
    - 이미 진행중인 트랜잭션이 있어도 항상 새로운 트랜잭션을 생성한다. 새로운 트랜잭션이 생성된다면 이미 진행중인 트랜잭션은 잠깐 보류된다.
- SUPPORT
    - 이미 진행중인 트랜잭션이 있다면 해당 트랜잭션 속성을 따르고, 없다면 트랜잭션을 설정하지 않는다.
- NOT_SUPPORT
    - 이미 진행중인 트랜잭션이 있다면 보류하고, 트랜잭션 없이 작업을 수행한다.
- MANDATORY
    - 이미 진행중인 트랜잭션이 있어야만, 작업을 수행한다. 없다면 Exception을 발생시킨다.
- NEVER
    - 트랜잭션이 진행중이지 않을 때 작업을 수행한다. 트랜잭션이 있다면 Exception을 발생시킨다.
- NESTED
    - 진행중인 트랜잭션이 있다면 중첩된 트랜잭션이 실행되며, 존재하지 않으면 REQUIRED와 동일하게 실행된다.
  
<br>

### REQUIRES_NEW 와 NESTED의 차이

중첩 트랜잭션(NESTED)는 트랜잭션 안에 다시 트랜잭션을 만드는 것이지만 REQUIRES_NEW는 독립적인 트랜잭션 만드는것이다.

중첩된 트랜잭션은 먼저 시작된 부모 트랜잭션의 커밋과 롤백에는 영향을 받지만 자신의 커밋과 롤백은 부모 트랜잭션에게 영향을 주지 않는다. 예를 들어 어떤 중요한 작업을 진행하는 중에 작업 로그를 DB에 저장해야 한다고 해보자. 그런데 로그를 저장하는 작업이 실패하더라도 메인 작업의 트랜잭션까지는 롤백해서는 안되는 경우가 있다. 힘들게 처리한 시급한 작업을 단지 로그를 남기는 작업에 문제가 있다고 모두 실패로 만들 수는 없기 때문이다. 반면에 로그를 남긴 후에 핵심 작업에서 예외가 발생한다면 이때는 저장한 로그도 제거해야 한다. 바로 이럴 때 로그 작업을 메인 트랜잭션에서 분리해서 중첩 트랜잭션으로 만들어 두면 된다. 메인 트랜잭션이 롤백되면 중첩된 로그 트랜잭션도 같이 롤백되지만, 반대로 중첩된 로그 트랜잭션이 롤백되도 메인 작업에 이상이 없다면 메인 트랜잭션은 정상적으로 커밋된다.

<br><br>

# isolation(격리레벨)

트랜잭션에서 일관성없는 데이터 허용 수준을 설정합니다.

- DEFAULT
    - 사용하는 DB 드라이버의 디폴트 설정을 따른다. 대부분 READ_COMMITED를 기본 격리수준으로 설정한다.
- READ_UNCOMMITED (level 0)
    - 가장 낮은 격리 수준이다. 트랜잭션이 커밋되기 전에 그 변화가 다른 트랜잭션에 그대로 노출된다.
    - 하지만 속도가 빠르기 때문에 데이터의 일관성이 떨어지더라도, 성능 극대화를 위해 의도적으로 사용하기도 한다.
    - ![image2](/Img/Transaction/transaction2.png)
- READ_COMMITED (level 1)
    - 트랜잭션이 커밋하지 않은 정보는 읽을 수 없다. 하지만 트랜잭션이 읽은 로우를 다른 트랜잭션에서 수정 할 수 있다. 그래서 트랜잭션이 같은 로우를 읽었어도 시간에 따라서 다른 내용이 발견될 수 있다.
    - ![image3](/Img/Transaction/transaction3.png)
- REPEATABLE_READ (level 2)
    - 트랜잭션이 읽은 로우를 다른 트랜잭션에서 수정되는 것을 막아준다. 하지만 새로운 로우를 추가하는 것은 제한하지 않는다.
    - ![image4](/Img/Transaction/transaction4.png)
- SERIALIZABLE (level 3)
    - 가장 강력한 트랜잭션 격리수준이다. 여러 트랜잭션이 동시에 같은 테이블 로우에 액세스하지 못하게 한다.
    - 가장 안전하지만 가장 성능이 떨어진다.

<br>

### 격리수준에 따른 문제

![image1](/Img/Transaction/transaction1.png)

- Dirty Read
    - 트랜잭션 1이 수정중인 데이터를 트랜잭션2가 읽을 수 있다. 만약 트랜잭션 1의 작업이 정상 커밋되지 않아 롤백되면, 트랜잭션 2가 읽었던 데이터는 잘못된 데이터가 되는것이다.
- Non-repeatable Read
    - 트랜잭션 1이 회원 A를 조회중에 트랜잭션 2가 회원 A의 정보를 수정하고 커밋한다면, 트랜잭션 1이 다시 회원 A를 조회했을 때는 수정된 데이터가 조회된다. 이처럼 반복해서같은 데이터를 읽을 수 없는 경우이다.
- Phantom Read
    - 트랜잭션 1이 10살 이하의 회원을 조회했는데 트랜잭션 2가 5살 회원을 추가하고 커밋하면 트랜잭션 1이 다시 10살 이하 회원을 조회했을 때 회원 한명이 추가된 상태로 조회된다. 이처럼 반복 조회시 결과집합이 달라지는 경우이다.

<br><br>  

# 다양한 설정

- rollbackFor
    - 트랜잭션 작업 중 런타임 예외가 발생하면 롤백한다. 반면에 예외가 발생하지 않거나 체크 예외가 발생하면 커밋한다.
    - 체크 예외를 커밋 대상으로 삼는 이유는 체크 예외가 예외적인 상황에서 사용되기 보다는 리턴 값을 대신해서 비즈니스 적인 의미를 담은 결과로 돌려주는 용도로 사용되기 때문이다.
    - 스프링에서는 데이터 엑세스 기술의 예외를 런타임 예외로 전환해서 던지므로 런타임 예외만 롤백대상으로 삼는다.
    - 하지만 원한다면 체크예외지만 롤백 대상으로 삼을 수 있다. `rollbackFor` or `rollbackForClassName` 속성을 이용해서 예외를 지정한다.
- noRollbackFor
    - rollbackFor 속성과는 반대로 런타임 예외가 발생해도 지정한 런타임 예외면 커밋을 진행한다.
- timeout
    - 트랜잭션에 제한시간을 지정한다. 초 단위로 지정하고, 디폴트 설정으로 트랜잭션 시스템의 제한시간을 따른다.
- readOnly
    - 트랜잭션을 읽기 전용으로 설정한다. 특정 트랜잭션 안에서쓰기 작업이 일어나는 것을 의도적으로 방지하기 위해 사용된다. `insert`,`update`,`delete` 작업이 진행되면 예외가 발생한다.


### **주의사항**

**rollbackFor** 설정을 보면 `@Transactional`은 기본적으로 **Unchecked Exception**, **Error** 만을 rollback하고 있다. 즉 **Checked Exception**이 발생하면 rollback을 처리하지 않느나. 그 이유는 **Checked Exception** 같은 경우는 예상된 에러이며 **Unchecked Exception**, **Error** 같은 경우는 예상치 못한 에러이기 때문이다. 그렇기 때문에 모든 예외에 대해서 **rollback**을 진행하고 싶을 경우 `(rollbackFor = Exception.class)`를 붙여야 한다.

<br><br>

---

**참고자료**

[https://springsource.tistory.com/136](https://springsource.tistory.com/136)

[https://velog.io/@kdhyo/JavaTransactional-Annotation-알고-쓰자-26her30h](https://velog.io/@kdhyo/JavaTransactional-Annotation-%EC%95%8C%EA%B3%A0-%EC%93%B0%EC%9E%90-26her30h)
https://www.youtube.com/watch?v=e9PC0sroCzc