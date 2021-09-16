Bean Validation은 많은 편리한 기능을 제공해 주지만 그럼에도 추가적인 기능이나 필요한 제약을 직접 정의해서 사용하고 싶을 때가 있습니다.

프로젝트를 진행하다 Enum 타입을 String으로 입력받는 경우에는 어떻게 검증할 수 있을까라는 궁금증이 들어서 Enum 타입을 직접 정의해서 검증해보도록 하겠습니다.

<br><br>

# Validation

**Phone.class**

```java
@Data
@AllArgsConstructor
public class Phone {

    @NotBlank
    private String phone1;

    @NotBlank
    @Length(min = 3, max = 4)
    private String phone2;

    @NotBlank
    @Length(min = 3, max = 4)
    private String phone3;
}
```

**Role.ENUM**

```java
@Getter
public enum Role {
    ADMIN("관리자"),
    USER("유저"),
    GUEST("손님");

    String value;

    Role(String value) {
        this.value = value;
    }
}
```

**Member.class**

```java
@Data
@AllArgsConstructor
public class Member {

    @NotBlank
    private String name;

    @???
    private Phone phone;

    @???
    private String roleType;

}
```

<br>

**Nested**

먼저 `Member` 객체에 `Phone` 객체가 nested로 구성되어 있습니다.

이러한 경우 `@Valid` 애노테이션을 설정해야 `Phone`객체에 설정한 제약조건이 정상적으로 작동합니다.

```java
@Valid
private Phone phone;
```

<br><br>

# Custom Validator

먼저 Enum annotation을 작성해줍니다.

```java
package hello.itemservice.domain.validation;

import javax.validation.Constraint;
import javax.validation.Payload;
import javax.validation.constraints.NotBlank;
import java.lang.annotation.Documented;
import java.lang.annotation.Repeatable;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;

import static java.lang.annotation.ElementType.*;
import static java.lang.annotation.ElementType.TYPE_USE;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

@Documented
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE })
@Retention(RUNTIME)
@Constraint(validatedBy = {EnumValidator.class})
public @interface Enum {

    String message() default "Enum에 없는 값입니다.";

    Class<?>[] groups() default { };

    Class<? extends Payload>[] payload() default { };

    Class<? extends java.lang.Enum<?>> enumClass();

    boolean ignoreCase() default false;
}
```

Custom Constraint Annotation을 만들때는 message, groups, payload 3개는 꼭 정의해야 합니다.

- `message()` : 메시지 관리를 위해 사용됩니다.
- `group()` : 상황별 validation 제어를 위해 사용됩니다.
- `payload()` : 심각도를 나타낸다고 하는데 거의 사용할 일이 없습니다.

<br>

**추가**

- `enumClass()` : 제약할 Enum 클래스를 지정
- `ignoreCase()` : 대소문자 구분 여부를 결정합니다.

<br><br>

자세한건 다음 EnumValidator를 보며 추가 설명하겠습니다.

**EnumValidator**

```java
package hello.itemservice.domain.validation;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

public class EnumValidator implements ConstraintValidator<Enum, String> {

    private Enum annotation;

    @Override
    public void initialize(Enum constraintAnnotation) {
        this.annotation = constraintAnnotation;
    }

    //equalsIgnoreCase -> 대소문자를 구분하지 않고 비교
    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        Object[] enumValues = this.annotation.enumClass().getEnumConstants();
        if (enumValues != null) {
            for (Object enumValue : enumValues) {
                if (value.equals(enumValue.toString())
                        || (this.annotation.ignoreCase() && value.equalsIgnoreCase(enumValue.toString()))) {
                    return true;
                }
            }
        }
        return false;
    }
}
```

`initialize()` 는 필수는 아닙니다.

하지만 `constraintAnntation`에 있는 정보를 멤버변수로 저장해서 `isValid()`에서 사용할 수 있습니다.

지금은 Enum의 정보가 필요하므로 initialize()를 정의하였습니다.

<br>

`isValid()` 제약 조건을 설정합니다.

```java
if (value.equals(enumValue.toString())
   || (this.annotation.ignoreCase() && value.equalsIgnoreCase(enumValue.toString())))
```

`equals` 메서드는 대소문자를 구분하며 비교하고

`equalsIgnoreCase` 메서드는 대소문자를 구분하지 않고 비교합니다.

ignoreCase 가 true 인경우 대소문자를 구분하지 않고, false 인경우 대소문자를 구분해서 검증합니다.

Validation에 대한 설정은 모두 끝났습니다. 

<br>

이제 Member 객체에 적용해 주면 됩니다.

```java
@Enum(enumClass = Role.class, ignoreCase = true)
private String roleType;
```

enumClass로 Role 객체를 지정해주고, ignoreCase=true로 대소문자 구분을 하지 않습니다.

<br><br>

# 테스트 코드

```java
private static Validator validator;

@BeforeEach
void setUpValidator() {
    ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
    validator = factory.getValidator();
}

@Test
void phone1IsBlank() {
    Phone phone = new Phone(" ", "1234", "5678");

    Member member = new Member(
            "memberA",
            phone,
            Role.ADMIN,
            "ADMIN");

    Set<ConstraintViolation<Member>> violations = validator.validate(member);
    
    assertThat(violations.size()).isEqualTo(1);
    assertThat(violations).extracting("message").containsOnly("공백일 수 없습니다");
}

@Test
void roleTypeNotValid() {
    Phone phone = new Phone("010", "1234", "5678");

    Member member = new Member(
            "memberA",
            phone,
            Role.ADMIN,
            "VIP");

    Set<ConstraintViolation<Member>> violations = validator.validate(member);
   
    assertThat(violations.size()).isEqualTo(1);
    assertThat(violations).extracting("message").containsOnly("Enum에 없는 값입니다.");

}
```

`void phone1IsBlank()` 

- phone1 이 빈칸이므로 `@NotBlank` 제약에 검증되었습니다.
- Nested 검증이 정상 작동하였습니다.

`void roleTypeNotValid()`

- Role 에 없는 "VIP"객체를 입력
- Role에 없는 객체이므로 오류 발생
- Enum 애노테이션을 생성할때 `message()`를 "Enum에 없는 값입니다." 로 설정했기 때문에 에러메시지가 `"Enum에 없는 값입니다."` 라고 출력됩니다.

<br><br>

# Message

Validation 실행결과를 보면 roleType의 오류 메시지가가 미리 설정한 `"Enum에 없는 값입니다."` 로 나오는걸 알수 있습니다.

<br>

Validation Annotation에 있는 message는 재정의 할 수 있습니다.

```java
@Enum(enumClass = Role.class, ignoreCase = true, message ="잘못된 Enum 값 입니다.")
private String roleType;
```

이 방법은 소스코드에서 바로 메시지 내용을 알 수 있는 것이 장점이지만, 다국어 처리가 어렵고 메시지를 수정하기 위해서는 Java 코드를 수정해야 합니다.

Custom Message Template을 지정하는 방법입니다.

기본으로 저장되어 있는 메시지 말고 새롭게 정의하고 싶다면 [ValidationMessages.properties](http://validationmessages.properties) 를 생성하여 새로 추가한 항목만 재정의하면 됩니다. ConstraintViolation 의 오류 메시지는 ValidationMessages.properties에 정의되어 있는걸 우선적으로 사용합니다.

<br>

**ValidationMessages.properties**

```java
validation.constraints.Enum.message=Enum에 없는 값입니다.
```

<br>

```java
@Enum(enumClass = Role.class, ignoreCase = true, message ="{validation.constraints.Enum.message}")
private String roleType;
```

Message Property를 통해 오류 메시지를 재정의하면 추후 메시지 변경시 Java코드를 수정할 필요 없이 `ValidationMessages.properties`의 내용만 수정하면 됩니다.