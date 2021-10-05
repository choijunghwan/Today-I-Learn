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

생성자는 엔티티를 생성하는 가장 간단한 방법입니다.  
만약 객체가 간단하다면 생성자를 사용하는게 가장 좋은 방법처럼 보입니다.

<br><br>

# 정적 팩토리 메서드
정적 팩토리 메
