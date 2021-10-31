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

