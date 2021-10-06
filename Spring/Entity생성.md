웹 개발을 하게되면 수많은 엔티티 개발을 하게 됩니다.

주로 생성자를 이용하거나 빌더 패턴을 사용하였는데 modelMapper와 같은 다른 방식이 있는것을 알고나서 각 방법의 장단점이 무엇이고 어느것이 더 좋을까의 궁금증이 들어 공부를 시작하였습니다.

제가 공부 해볼 엔티티 생성 방법은 아래와 같습니다.

### 엔티티를 생성하는 방법
1. 생성자 사용
2. 정적 팩토리 메서드 사용
3. 빌더 패턴 사용
4. modelMapper 사용

<br><br>

# 생성자 
생성자는 new 연산자와 같이 사용되어 클래스로부터 객체를 생성할 때 호출되어 객체의 초기화를 담당하는 역할을 해줍니다.

생성자는 많은 분들이 아시는 부분이니 바로 코드를 보겠습니다.

**Student 객체**
```Java
@Getter
public class Student {

    private Long id;

    private String name;

    private Integer age;
}
```

<br>

**생성자 생성**
```Java
public Student(Long id, String name, Integer age) {
    this.id = id;
    this.name = name;
    this.age = age;
}
```
생성자 생성 규칙
* 생성자의 이름은 클래스의 이름과 같아야 한다.
* 생성자는 리턴 값이 없다.
* 생성자도 오버로딩이 가능하므로 하나의 클래스에 여러 개의 생성자가 있을 수 있다.
* 생성자도 메서드이기 때문에 리턴값이 없다는 의미로 void를 적어야 하지만, 모든 생성자가 리턴값이 없으므로 관례로 void를 생략합니다.

<br>

**객체 생성**
```Java
Student student = new Student(1L, "학생A", 10);
```


<br><br>

# 정적 팩토리 메서드
정적 팩토리 메서드는 엔티티 생성에 의미를 부여하고 무분별한 `Setter`의 사용을 제한하기 위해 도입되기 시작하였습니다.

### Setter의 제한
엔티티의 모든 필드에 `Setter` 메서드를 생성하게 되면 객체의 값의 변경을 열어두기 때문에 객체의 일관성을 보장하기 어렵고 무분별한 값의 변경을 막을수 없게 됩니다.

하지만 만약 학생의 나이를 변경할 수 있게 기능을 열어두어야 한다면 정적 팩토리 메서드를 이용해 의도적으로 값의 변경을 허용해줄 수 있습니다.

```Java
// Setter
public void setAge(Integer age) {
    this.age = age;
}

// 정적 팩토리 메서드
public void changeAge(Integer age) {
    this.age = age;
}
```
만약 `setAge()`처럼 Setter 메서드를 생성한다면 같이 개발하는 개발자의 입장에서는 학생의 나이를 변경을 해도 되는건지 안되는건지를 알 수 없습니다.

허나 `changeAge()` 메서드를 생성한다면 메서드의 이름을 보고 학생의 나이를 변경하는 메서드구나를 알 수 있고 나이 필드 값을 변경해도 된다는 정보도 알 수 있습니다.

객체 생성도 정적 팩토리 메서드를 통해 만들어 줄 수 있습니다.
```Java
public Student(Long id, String name, Integer age) {
    this.id = id;
    this.name = name;
    this.age = age;
}

public Student createStudent(Long id, String name, Integer age) {
    return new Student(id, name, age);
}
```
정적 팩토리 메서드 장점
* 이름을 가질 수 있다.(의미를 부여할 수 있다)
* 호출 될 때마다 인스턴스를 새로 생성하지 않아도 된다.
* 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
* 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
* 정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

<br><br>

# 빌더 패턴
빌더 패턴은 복잡한 객체를 생성하는 방법을 정의하는 클래스와 표현하는 방법을 정의하는 클래스를 별도로 분리하여, 서로 다른 표현이라도 이를 생성할 수 있는 동일한 절차를 제공하는 패턴입니다.

빌더 패턴은 생성 패턴으로 인스턴스를 만드는 절차를 추상화하는 패턴입니다. 특히 빌더 패턴은 많은 Optional한 멤버 변수(혹은 파라미터)나 지속성 없는 상태 값들에 대해 처리해야 할 때 큰 장점을 가지고 있습니다.

**빌더 패턴 구현 방법**
1. 빌더 클래스를 Static Nested Class로 생성합니다. 이때, 관례적으로 생성하고자 하는 클래스 이름 뒤에 Builder를 붙입니다.
2. 빌더 클래스의 생성자는 public으로 하며, 필수 값들에 대해 생성자의 파라미터로 받습니다.
3. Optional한 값들에 대해서는 각각의 속성마다 메소드로 제공하며, 이때 리턴 값이 빌더 객체 자신이어야 합니다.
4. 빌더 클래스 내에 build() 메서드를 정의하여 클라이언트 프로그램에게 최종 생성된 결과물을 제공합니다.

<br>

글로만 보면 이해가 어렵기 때문에 바로 코드를 보도록 하겠습니다.

```Java
@Getter
public class Student {

    private Long id;
    private String name;
    private Integer age;

    private Student(StudentBuilder builder) {
        this.id = builder.id;
        this.name = builder.name;
        this.age = builder.age;
    }
    
    //Nested Builder Class
    public static class StudentBuilder {

        private Long id;
        private String name;
        private Integer age;

        //필수값들은 생성자의 파라미터
        public StudentBuilder(Long id) {
            this.id = id;
        }

        public StudentBuilder setName(String name) {
            this.name = name;
            return this;
        }

        public StudentBuilder setAge(Integer age) {
            this.age = age;
            return this;
        }

        public Student build() {
            return new Student(this);
        }

    }
}
```
`StudentBuilder` 클래스를 보면 `id`값은 필수 값으로 생성자의 파라미터를 통해 받고 나머지 필드값들은 각각의 속성마다 메서드를 생성하였습니다.

빌더 패턴을 통해 많은 **장점**을 얻을 수 있습니다.
1. 경우에 따라 필요 없는 파라미터들에 대해 일일이 null 값을 넘겨주지 않아도 됩니다.
2. 객체를 생성할때 파라미터의 순서를 신경쓰지 않아도 됩니다.

추가로 Student 클래스를 보면 public 생성자가 없는것을 볼수있습니다. **Student 객체를 생성하기 위해서는 오직 StudentBuilder 클래스를 통해서만 가능**합니다.

<br>

**Student 객체 생성**
```Java
// new Student(1L, "학생A", 10);
Student student = new Student.StudentBuilder(1L)
            .setName("학생A")
            .setAge(10)
            .build();
```
* StudentBuilder를 통해 Student 객체를 생성할때 Optional한 Name,Age에 대해서는 필요없는 값이면 메서드를 사용하지 않으면 됩니다.
* Name, Age 메서드의 순서가 바뀌어도 가능합니다.

빌더 패턴은 많은 장점이 있는데 위와같이 추가적인 많은 코드가 필요합니다. 위의 예시는 필드가 3개뿐이지만 만약 필드가 많아지면 그만큼 코드의 양도 증가하게 되고 객체가 student 뿐만아니라 여러개의 객체가 있다고 생각하면 필요한 코드의 양은 생각만 해도 아찔합니다. 

<br>

그래서 이걸 해결하기 위해 `@Builder`라는 Lombok을 사용합니다.
```Java
@Getter
@Builder
public class Student {

    private Long id;
    private String name;
    private Integer age;
}
```
이와같이 `@Builder` 애노테이션을 사용하면 위의 `Nested Builder Class`를 생성한것과 똑같습니다.

<br>

**Student 객체 생성 - @Builder 사용**
```Java
Student student = Student.builder()
            .id(1L)
            .name("학생A")
            .age(10)
            .build();
```
Lombok `@Builder`는 builder 라는 클래스 명을 사용하며 메서드의 이름은 기존의 필드명을 그대로 사용합니다.

사실 `@Builder` 애노테이션을 class 레벨에 사용하는 것은 추천드리지 않습니다.

* `@Builder`를 Class에 적용시키면 생성자의 접근 레벨이 default이기 때문에, 동일 패키지 내에서 해당 생성자를 호출 할 수 있는 문제
* 모든 멤버 필드에 대해서 매개변수를 받는 기본 생성자를 만듭니다.
  * 마치 `@AllArgsConstructor` 와 같은 효과를 발생시킵니다.
  * Student 객체에서 id 값이 데이터베이스 PK 생성전략에 의존하고 있다고 가정한다면 객체를 생성할 때 id값을 넘겨받지 않아야 합니다.
  * Class 레벨에 적용하면 객체생성에 제한을 두기가 어렵습니다.

따라서 private 생성자를 구현해서 `@Builder`를 지정하는 방법을 추천드립니다.
```Java
@Getter
public class Student {

    private Long id;

    private String name;

    private Integer age;

    @Builder
    private Student(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
}
```

**Student 객체 생성**
```Java
Student student = Student.builder()
            //.id(1L)  error 발생
            .name("학생A")
            .age(10)
            .build();
```
`@Builder` 생성자가 name, age 파라미터만 가지고 있으므로 Student 객체를 생성할 때 id값을 넘겨받지 못하게 제한할 수 있습니다.

<br><br>

