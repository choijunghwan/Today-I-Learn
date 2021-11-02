# Builder Annotation
개발하면서 우리는 객체 생성을 하기 위해 Builder 패턴을 많이 사용하게 된다.

직접 Builder를 만드는것은 매우 귀찮은 일이지만 우리는 @Builder 어노테이션을 통해 간단하게 Builder를 사용할 수 있습니다.

오늘은 @Builder의 다양한 사용 방법에 대해 공부해보고자 합니다.

<br><br>

# @Builder 사용하기
빌더를 사용하는 방법은 클래스 선언에 어노테이션을 추가하는것입니다.

바로 예시 코드로 보겠습니다.


```Java
public enum Gender {
    MAN, FEMALE;
}
```
Enum 값에대해서도 테스트하기 위해 Gender를 추가하였습니다.

<br><br>

```Java
@Builder
@Getter
@ToString
public class Custom {

    private String name;
    private Integer age;
    private LocalDateTime time;
    private Boolean check;
    private List<String> jobs;
    private Gender gender;
}
```
위와 같이 **@Builder** 어노테이션만 추가해주면 사용가능합니다.

<br>

**적용**
```Java
@Test
public void builderTest() {
    Custom custom = Custom.builder()
            .name("Custom")
            .age(20)
            .time(LocalDateTime.now())
            .check(true)
            .jobs(Arrays.asList("직업"))
            .gender(Gender.MAN)
            .build();

    System.out.println("custom = " + custom);
    assertThat(custom.getName()).isEqualTo("Custom");
    assertThat(custom.getAge()).isEqualTo(20);
    assertThat(custom.getCheck()).isEqualTo(true);
}
```

**실행결과**  
`custom = Custom(name=Custom, age=20, time=2021-10-31T23:17:14.533483500, check=true, jobs=[직업], gender=MAN)`

빌더를 통해 객체를 생성해 보았다. 다만 이렇게보면 빌더의 큰 장점이 와닿지 않는다. 빌더의 진짜 장점을 좀 더 알아보자.

<br><br>

# Builder.Default
빌더는 왜 좋은걸까??

빌더는 개발자가 좀 더 능동적이게 객체를 생성할 수 있게 도와준다.

다음 코드를 보며 좀 더 생각해보자.

```Java
@Test
public void builder() {
    Custom custom = Custom.builder()
            .age(20)
            .jobs(Arrays.asList("직업"))
            .gender(Gender.MAN)
            .time(LocalDateTime.now())
            .build();

    System.out.println("custom = " + custom);
    assertThat(custom.getAge()).isEqualTo(20);
}
```
객체를 생성할때 `name`과 `check` 필드를 제외하고 생성을 하였다. 게다가 순서도 바뀌어 `time`필드가 제일 마지막에 추가하여 생성했다.

하지만 이렇게 해도 `Builder`는 아무 문제 없이 객체를 생성해준다. 

Builder의 장점
* 파라미터의 순서가 상관없다.
* 필요한 파라미터만 넘기면 된다.

만약 우리가 생성자를 통해 객체를 생성한다면 파라미터 순서도 지켜야하고 `name`과 `check`같은 팔드에는 null을 채워 넘겨주어야 한다.

이제 개발자들은 좀 더 편하게 객체를 생성 할 수 있다.

<br>

그러면 우리가 추가하지 않은 `name`과 `check`필드에는 어떤 값이 들어가게 될까??

바로 **null**값이 들어가게 된다.

이게 위의 코드의 실행결과이다.  
`custom = Custom(name=null, age=20, time=2021-10-31T23:35:04.204326400, check=null, jobs=[직업], gender=MAN)`

<br><br>

그러면 다른 필드는 어떻게 되는지 한번 테스트 해보자.

```Java
@Test
public void builderNull() {
    Custom custom = Custom.builder()
            .build();

    System.out.println("custom = " + custom);
}
```

실행결과  
`custom = Custom(name=null, age=null, time=null, check=null, jobs=null, gender=null)`

<br>

Builder는 값을 설정하지 않으면 자동으로 null을 채워주는것을 확인할 수 있다.

여기서 저는 Integer, Boolean Wrapper 타입으로 선언하여 모두 null이 채워졌지만 만약 int, boolean 으로 Primitive 타입으로 선언한다면 int는 0을 boolean는 false를 기본으로 채워줍니다.

즉, Builder는 0/null/false 3가지 값중 하나로 값을 채워줍니다.

만약 우리가 직접 기본값을 설정해주고 싶다면 Lombok 1.16.16 이후의 버전이라면 `@Builder.Default`를 사용하여 기본값을 정해줄 수 있습니다.

```Java
@Builder
@Getter
@ToString
public class Custom {

    private String name;
    @Builder.Default
    private Integer age = 0;

    @Builder.Default
    private LocalDateTime time = LocalDateTime.now();

    @Builder.Default
    private Boolean check = false;
    private List<String> jobs;

    @Builder.Default
    private Gender gender = Gender.MAN;
}
```

<br>

`@Builder.Default`를 사용하는 방법은 필드에 어노테이션을 추가한뒤 필드에 기본값을 직접 지정해주면 됩니다.
```Java
@Builder.Default
private Integer age = 0;
```

`time`, `check`, `gender` 필드에도 추가로 기본값을 추가해줬습니다.

<br>

```Java
@Test
public void builderDefault() {
    Custom custom = Custom.builder()
            .name("Custom")
            .jobs(Arrays.asList("직업"))
            .build();

    System.out.println("custom = " + custom);
    assertThat(custom.getName()).isEqualTo("Custom");
    assertThat(custom.getAge()).isEqualTo(0);
    assertThat(custom.getCheck()).isEqualTo(false);
    assertThat(custom.getGender()).isEqualTo(Gender.MAN);
}
```
**실행결과**  
`custom = Custom(name=Custom, age=0, time=2021-10-31T23:59:39.079739800, check=false, jobs=[직업], gender=MAN)`

위와같이 `@Builder.Default`로 설정해준 기본값이 모두 잘 입력되어 있는것을 확인할 수 있습니다.

<br><br>

# Builder Singular
우리가 생성한 `Custom` 객체의 `job` 필드를 보면 컬렉션으로 생성되어 있습니다.  
`private List<String> jobs;`

그리고 `.jobs(Arrays.asList("직업1", "직업2"))` 라는 방법을 통해 값을 설정할수있었습니다.

이것을 `@Singular`를 사용하면 한번에 하나씩 값 목록을 작성할 수 있습니다.

**Singular 추가**
```Java
@Singular
private List<String> jobs;
```

<br>

**객체 생성**
```Java
@Test
public void builderSingular() {
    Custom custom = Custom.builder()
            .name("Custom")
            .job("직업1")
            .job("직업2")
            .build();

    System.out.println("custom = " + custom);
    assertThat(custom.getName()).isEqualTo("Custom");
    assertThat(custom.getJobs().size()).isEqualTo(2);
}
```

<br>

**실행결과**  
`custom = Custom(name=Custom, age=0, time=2021-11-01T01:04:05.271057900, check=false, jobs=[직업1, 직업2], gender=MAN)`

여기서는 @Singular가 java.util.List로 작업을 했지만 Set,Map 자료구조로도 가능합니다. 

여기서 주의할점은 `@Singular`를 사용하면 인수가 단수형식으로 전달된다는 점입니다. 위의경우 `jobs`이므로 `job`의 형태로 전달됩니다. 만약 단수형이 애매한경우 직접 이름을 지정해 줄 수도 있습니다. `@Singular("job") List<String> jobs` 라고 명시적으로 지정해주면 됩니다.

<br><br>

# 마무리
오늘 Builder 어노테이션의 여러 기능들을 공부해보았습니다.

먼저 `Default` 기능의 경우 null이라는 기본값이외에도 지정된 값을 넘길수 있어 매우 유용한것 같습니다.

그러나 `Singular`의 경우 실제 객체를 생성할때 컬렉션의 값을 일일이 넣어주는 경우는 드물고 컬렉션 객체를 넘겨받아 넣어주기때문에 사용빈도가 적을것 같은 기능입니다.

`Singular`를 사용하지 않으면 객체를 생성할때 `.jobs(Arrays.asList("직업1", "직업2"))` 와 같은 코드가 발생해 빌더 문법이 어설프고, 투박해 보이지만 훨씬 실용적인 방법인것 같습니다.

<br>

참고자료
* https://www.baeldung.com/lombok-builder-singular
* https://projectlombok.org/features/Builder
* https://umbum.dev/1016
