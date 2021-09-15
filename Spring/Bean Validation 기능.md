개발을 할때 기능을 개발하는것 만큼 중요한게 validation 입니다.

그중에서 클라이언트로부터 입력받은 값의 오류로 발생하는 장애가 꽤 많습니다.

입력받은 값이 잘못되면 전혀 예상치 못한 곳에서 에러가 발생하여 원인 파악이 힘든 경우가 많이 발생하게 됩니다.

그래서 클라이언트로부터 입력받은 값을 검증하는 방법중 하나인 Annotaion을 이용한 Validation의 다양한 검증 기능들에 대해서 알아보겠습니다.

# Bean Validation 시작하기

## 환경설정

build.gradle

```java
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

Spring Boot Validation Starter를 추가해줍니다.

이것만으로 Validation을 사용할 준비는 끝입니다.

<br><br>

# 제약 설정과 검사 - Null 관련

그럼 Bean Validation에서 제공하는 애노테이션들을 알아보겠습니다.

**Address**

```java
package hello.itemservice.domain;

import lombok.AllArgsConstructor;
import lombok.Data;

import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.NotNull;

@Data
@AllArgsConstructor
public class Address {

    @NotBlank // null, "", " " 을 체크
    private String city;

    @NotEmpty // null, "" 을 체크
    private String street;

    @NotNull  // null 을 체크
    private String zipcode;

}
```

**애노테이션 역할**

- `@NotBlank` : `Null`, `""`(빈값) , `" "` (공백) 을 허용하지 않습니다.
- `@NotEmpty` : `Null`, `""` (빈값) 을 허용하지 않습니다.
- `@NotNull` : `Null` 을 허용하지 않습니다.

<br>

이제 `Address` 객체를 생성해서 Validation이 잘 적용되었는지 테스트해보겠습니다.

<br>

**AddressTest**

```java
package hello.itemservice.domain;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import javax.validation.ConstraintViolation;
import javax.validation.Validation;
import javax.validation.Validator;
import javax.validation.ValidatorFactory;

import java.util.Set;

import static org.assertj.core.api.Assertions.*;

class AddressTest {

    private static Validator validator;

    /**
     * 컴퓨터 기본 설정이 한글로 되어 있어 오류메시지가 한글로 나온다.
     * 오류메시지를 다른 언어로 보고싶으면 Locale을 설정하면 된다.
     * 예를 들어 영어로 보고싶다면
     * Locale.setDefault(Locale.US);
     */
    @BeforeEach
    void setUpValidator() {
        ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
        validator = factory.getValidator();
    }

    @Test
    void cityIsNull() {
        Address address = new Address(null, "잠실", "12345");

				//검증 실행
        Set<ConstraintViolation<Address>> violations = validator.validate(address);
        
				//오류 정보
				for (ConstraintViolation<Address> violation : violations) {
						System.out.println("violation = " + violation);
            System.out.println("violation.getMessage() = " + violation.getMessage());
            System.out.println("violation.getMessageTemplate() = " + violation.getMessageTemplate());
            System.out.println("violation.getRootBean() = " + violation.getRootBean());
            System.out.println("violation.getPropertyPath() = " + violation.getPropertyPath());
            System.out.println("violation.getRootBeanClass() = " + violation.getRootBeanClass());
        }

        assertThat(violations.size()).isEqualTo(1);
        assertThat(violations).extracting("message").containsOnly("공백일 수 없습니다");
    }

}
```

**검증기 생성**

```java
@BeforeEach
void setUpValidator() {
    ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
    validator = factory.getValidator();
}
```

<br>

**검증 실행**

```java
Address address = new Address(null, "잠실", "12345");
Set<ConstraintViolation<Address>> violations = validator.validate(address);
```

`Address` 객체를 생성합니다.

검증 대상(`address`)을 직접 검증기에 넣고 그 결과를 받습니다.

<br>

**ConstraintVioloation 메서드**

- `violation.getMessage()` : 보정된 오류 메시지 출력
- `violation.getMessageTemplate()` : 보정되지 않은 오류 메시지 출력
- `violation.getRootBean()` : 검증 중인 빈을 출력
- `violation.getPropertyPath()` : 현재 검증된 값의 속성 출력
- `violation.getRootBeanClass()` : 검증 중인 빈의 클래스 출력

<br>

**실행 결과**

```java
violation = ConstraintViolationImpl{interpolatedMessage='공백일 수 없습니다', propertyPath=city, rootBeanClass=class hello.itemservice.domain.Address, messageTemplate='{javax.validation.constraints.NotBlank.message}'}
violation.getMessage() = 공백일 수 없습니다
violation.getMessageTemplate() = {javax.validation.constraints.NotBlank.message}
violation.getRootBean() = Address(city=null, street=잠실, zipcode=12345)
violation.getPropertyPath() = city
violation.getRootBeanClass() = class hello.itemservice.domain.Address
```

<br>

`city` 속성 `""`(빈칸) 과 `" "`(공백) 검증

```java
@Test
void cityIsEmpty() {
    Address address = new Address("", "잠실", "12345");
    Set<ConstraintViolation<Address>> violations = validator.validate(address);

    assertThat(violations.size()).isEqualTo(1);
    assertThat(violations).extracting("message").containsOnly("공백일 수 없습니다");
}

@Test
void cityIsBlank() {
    Address address = new Address(" ", "잠실", "12345");
    Set<ConstraintViolation<Address>> violations = validator.validate(address);

    assertThat(violations.size()).isEqualTo(1);
    assertThat(violations).extracting("message").containsOnly("공백일 수 없습니다");
}

@Test
void cityIsValid() {
    Address address = new Address("서울", "잠실", "12345");
    Set<ConstraintViolation<Address>> violations = validator.validate(address);
    
    assertThat(violations.size()).isEqualTo(0);
}
```

`""` , `" "` 모두 오류가 발생 하는걸 확인 할 수 있습니다.

`@NotEmpty`, `@NotNull` 도 이와 같이 테스트 코드를 통해 확인 해 볼 수 있습니다. 이 과정은 생략하겠습니다.

<br>

## 오류 메시지 언어 설정

현재 시스템의 기본 언어 설정이 한글로 되어 있어 한글로 오류메시지가 출력됩니다.

만약 오류 메시지 언어를 바꾸고 싶다면 Locale 의 설정을 변경하면 됩니다.

예를 들어 영어로 보고싶다면

```java
@BeforeEach
void setUpValidator() {

		//언어 설정
		Locale.setDefault(Locale.US);

    ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
    validator = factory.getValidator();
}
```

`Locale.setDefault(Locale.US);` 언어를 설정해주면 됩니다.

<br><br>

# 제약 설정과 검사 - Size

**AnnotationSize.class**

```java
package hello.itemservice.domain;

import lombok.Data;

import javax.validation.constraints.Size;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Data
public class AnnotationSize {

    @Size(min = 2, max = 5)
    private String str;

    @Size(min = 3, max = 6)
    private List<String> list = new ArrayList<>();

    @Size(min = 3, max = 7)
    private Map<Integer,String> map = new HashMap<>();
}
```

`@Size` : 길이나 크기가 지정한 크기를 넘지 않는지 판단합니다.

**String**, **Collection**, **Map**, 배열 타입 모두 지원합니다.

<br>

**테스트 코드**

```java
package hello.itemservice.domain;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import javax.validation.ConstraintViolation;
import javax.validation.Validation;
import javax.validation.Validator;
import javax.validation.ValidatorFactory;

import java.util.List;
import java.util.Map;
import java.util.Set;

import static org.assertj.core.api.Assertions.assertThat;

class AnnotationSizeTest {

    private static Validator validator;

    @BeforeEach
    void setUpValidator() {
        ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
        validator = factory.getValidator();
    }

    @Test
    void allTooShort() {
        AnnotationSize annotationSize = new AnnotationSize();
        annotationSize.setStr("1");

        Set<ConstraintViolation<AnnotationSize>> violations = validator.validate(annotationSize);
        
        assertThat(violations.size()).isEqualTo(3);
        assertThat(violations).extracting("message").
                contains("크기가 2에서 5 사이여야 합니다", 
                        "크기가 3에서 6 사이여야 합니다", 
                        "크기가 3에서 7 사이여야 합니다");
    }

}
```

**검증객체**

- str = "1";
- list → 빈 객체
- map → 빈 객체

<br>

**테스트**

`AnnotationSize` 객체의 모든 필드가 제약 조건을 충족시키지 못하기 때문에 3개의 오류가 발생합니다.

```java
assertThat(violations.size()).isEqualTo(3);
assertThat(violations).extracting("message").
        contains("크기가 2에서 5 사이여야 합니다", 
                "크기가 3에서 6 사이여야 합니다", 
                "크기가 3에서 7 사이여야 합니다");
```

`@Size(min = 2, max = 5)` 인경우 오류메시지는 `"크기가 2에서 5 사이여야 합니다"` 로 생성됩니다.

<br><br>

# 제약 설정과 검사 - Number

**AnnotationNumber.class**

```java
package hello.itemservice.domain;

import lombok.AllArgsConstructor;
import lombok.Data;

import javax.validation.constraints.*;

@Data
@AllArgsConstructor
public class AnnotationNumber {

    @Min(2)
    @Max(5)    // @Range(min = 2, max=5) 도 같은 역할을 한다.
    private int minmax;

    @DecimalMin(value = "2")  // 지정된 최소값보다 큰지 확인, 기본값 inclusive=true이면 같은경우도 체크(>=) false이면(>)
    private int decimalMin;

    @DecimalMax(value = "5", inclusive = false)
    private int decimalMax;

    @Digits(integer = 3, fraction = 0)  // integer = 허용 가능한 정수 자릿수, fraction = 허용 가능한 소숫점 이하 자릿수
    private int digit;

    @Positive
    private int positive;

    @NegativeOrZero
    private int negativeOrZero;

}
```

**애노테이션 역할**

- `@Min()` : 지정한 최소값보다 큰지 검사합니다.
- `@Max()` : 지정한 최대값보다 작은지 검사합니다.
- `@Range(min = 2, max = 5)` : 값의 범위를 지정합니다.
- `@DecimalMin()` : 지정한 최소값보다 크거나 같은지 확인합니다.
    - 주요 속성
    - `value` : 지정 값
    - `inclusive` : 지정값 포함 여부
    - 기본값은 **true**로 **true**이면 지정한 값을 포함, **false**이면 지정한 값을 포함하지 않습니다.
    - null은 유효하다고 판단합니다.
- `@DecimalMax` : 지정한 최대값보다 작거나 같은지 확인합니다.
    - 위의 DecimalMin 과 동일한 속성을 갖고 있습니다.
- `@Digits` : 자릿수가 지정한 크기를 넘지 않는지 검사합니다.
    - 주요 속성
    - `integer` : 정수 자릿수를 지정합니다.
    - `fraction` : 소수점 이하 자릿수를 지정합니다.
    - null은 유효하다고 판단합니다.
- `@Positive` : 양수 인지 검사합니다.
    - OrZero가 붙은 것은 0또는 양수인지 검사합니다.
    - null은 유효하다고 판단합니다.
- `@NegativeOrZero` : 0또는 음수 인지 검사합니다.
    - OrZero 가 안붙은 것은 음수인지검사합니다.
    - null은 유효하다고 판단합니다.

<br>

**테스트 코드 - 일부**

```java
package hello.itemservice.domain;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import javax.validation.ConstraintViolation;
import javax.validation.Validation;
import javax.validation.Validator;
import javax.validation.ValidatorFactory;

import java.util.Set;

import static org.assertj.core.api.Assertions.*;

class AnnotationNumberTest {

    private static Validator validator;

    @BeforeEach
    void setUpValidator() {
        ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
        validator = factory.getValidator();
    }

    @Test
    void minTooLow() {
				//minmax값이 1로 Min(2)를 만족하지 못한다
        AnnotationNumber annotationNumber = new AnnotationNumber(1, 3, 3, 12, 10, -5);

        Set<ConstraintViolation<AnnotationNumber>> violations = validator.validate(annotationNumber);
        
        assertThat(violations.size()).isEqualTo(1);
        assertThat(violations).extracting("message").containsOnly("2 이상이어야 합니다");
    }

    @Test
    void decimalMinTooLow() {
				//decimalMin값이 1로 DecimalMin(2)를 만족하지 못한다.
        AnnotationNumber annotationNumber = new AnnotationNumber(4, 1, 3, 12, 10, -5);

        Set<ConstraintViolation<AnnotationNumber>> violations = validator.validate(annotationNumber);
        
        assertThat(violations.size()).isEqualTo(1);
        assertThat(violations).extracting("message").containsOnly("다음 값 이상이어야 합니다 2");
    }

    @Test
    void decimalMaxTooHigh() {
				//decimalMax값이 5로 @DecimalMax(value = "5", inclusive = false)를 만족하지 못한다.
        AnnotationNumber annotationNumber = new AnnotationNumber(4, 2, 5, 12, 10, -5);

        Set<ConstraintViolation<AnnotationNumber>> violations = validator.validate(annotationNumber);
        for (ConstraintViolation<AnnotationNumber> violation : violations) {
            System.out.println("violation = " + violation);
            System.out.println("violation.getMessage() = " + violation.getMessage());
        }

        assertThat(violations.size()).isEqualTo(1);
        assertThat(violations).extracting("message").containsOnly("다음 값 이하여야 합니다5");
    }

    @Test
    void digitTooLarge() {
				//digit 값이 "1234"로 @Digits(integer = 3, fraction = 0)를 만족하지 못한다.
        AnnotationNumber annotationNumber = new AnnotationNumber(4, 2, 4, 1234, 10, -5);

        Set<ConstraintViolation<AnnotationNumber>> violations = validator.validate(annotationNumber);
        for (ConstraintViolation<AnnotationNumber> violation : violations) {
            System.out.println("violation = " + violation);
            System.out.println("violation.getMessage() = " + violation.getMessage());
        }

        assertThat(violations.size()).isEqualTo(1);
        assertThat(violations).extracting("message").containsOnly("숫자 값이 한계를 초과합니다(<3 자리>.<0 자리> 예상)");
    }

    
    @Test
    void positiveIsZero() {
				//positive값이 0으로 @Positive 를 만족하지 못한다.
        AnnotationNumber annotationNumber = new AnnotationNumber(4, 2, 4, 123, 0, -5);

        Set<ConstraintViolation<AnnotationNumber>> violations = validator.validate(annotationNumber);
        for (ConstraintViolation<AnnotationNumber> violation : violations) {
            System.out.println("violation = " + violation);
            System.out.println("violation.getMessage() = " + violation.getMessage());
        }

        assertThat(violations.size()).isEqualTo(1);
        assertThat(violations).extracting("message").containsOnly("0보다 커야 합니다");
    }

}
```

테스트 결과 오류가 정상 출력되는것 확인해보았습니다.

<br><br>

# 제약 설정과 검사 - 기타

**User.class**

```java
package hello.itemservice.domain;

import lombok.AllArgsConstructor;
import lombok.Data;
import org.hibernate.validator.constraints.Length;

import javax.validation.constraints.*;
import java.time.LocalDateTime;

@Data
@AllArgsConstructor
public class User {

    @NotBlank
    private String userName;

    @Length(min = 3, max = 8)
    @Pattern(regexp = "[a-z0-9!@]+")
    private String password;

    @Email
    private String email;

    @AssertTrue // true 인지 검사한다. null은 유효하다고 판단
    private Boolean isVisited;

    @PastOrPresent
    private LocalDateTime createdDate;

    @Future
    private LocalDateTime lastModifiedDate;

}
```

**애노테이션 역할**

- `@Length` : 길이가 지정한 범위에 있는지 검사한다.
- `@Pattern` : 값이 정규표현식에 일치하는지 검사한다.
    - null은 유효하다고 판단한다.
- `@Email` : 이메일 주소가 유효한지 검사한다.
    - null은 유효하다고 판단한다.
- `@AssertTrue` : 값이 true인지 검사한다.
    - null은 유효하다고 판단한다.
    - @AssertFalse 는 값이 false인지 검사한다.
- `@PastOrPresent` : 해당시간이 현재 또는 과거 시간인지 검사한다.
    - OrPresent 가 붙지 않으면 과거 시간인지 검사한다.
- `@Future` : 해당 시간이 미래 시간인지 검사한다.
    - OrPresent가 붙으면 현재 또는 미래 시간인지 검사한다.

<br>

**테스트 코드 -일부**

```java
package hello.itemservice.domain;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import javax.validation.ConstraintViolation;
import javax.validation.Validation;
import javax.validation.Validator;
import javax.validation.ValidatorFactory;

import java.time.LocalDateTime;
import java.util.Set;

import static org.assertj.core.api.Assertions.*;

class UserTest {

    private static Validator validator;

    @BeforeEach
    void setUpValidator() {
        ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
        validator = factory.getValidator();
    }

    @Test
    void passwordIsUpper() {
        User user = new User(
                "UserA",
                "AAA",
                "abc@gmail.com",
                true,
                LocalDateTime.now(),
                LocalDateTime.MAX);

        Set<ConstraintViolation<User>> violations = validator.validate(user);
        
        assertThat(violations.size()).isEqualTo(1);
        assertThat(violations).extracting("message").containsOnly("\"[a-z0-9!@]+\"와 일치해야 합니다");
    }

    @Test
    void passwordIsNotMatch() {
        User user = new User(
                "UserA",
                "a$",
                "abc@gmail.com",
                true,
                LocalDateTime.now(),
                LocalDateTime.MAX);

        Set<ConstraintViolation<User>> violations = validator.validate(user);
        
        assertThat(violations.size()).isEqualTo(2);
        assertThat(violations).extracting("message").contains("길이가 3에서 8 사이여야 합니다", "\"[a-z0-9!@]+\"와 일치해야 합니다");
    }

    @Test
    void emailIsNull() {
        User user = new User(
                "UserA",
                "aa12",
                null,
                true,
                LocalDateTime.now(),
                LocalDateTime.MAX);

        Set<ConstraintViolation<User>> violations = validator.validate(user);
        
        assertThat(violations.size()).isEqualTo(0);
    }

    @Test
    void visitIsFalse() {
        User user = new User(
                "UserA",
                "aa12",
                "abc@gmail.com",
                false,
                LocalDateTime.now(),
                LocalDateTime.MAX);

        Set<ConstraintViolation<User>> violations = validator.validate(user);
        
        assertThat(violations.size()).isEqualTo(1);
        assertThat(violations).extracting("message").containsOnly("true여야 합니다");
    }

    @Test
    void lastModifiedDateIsPast() {
        User user = new User(
                "UserA",
                "aa12",
                "abc@gmail.com",
                true,
                LocalDateTime.now(),
                LocalDateTime.MIN);

        Set<ConstraintViolation<User>> violations = validator.validate(user);
        
        assertThat(violations.size()).isEqualTo(1);
        assertThat(violations).extracting("message").containsOnly("미래 날짜여야 합니다");
    }

    @Test
    void lastModifiedDateIsNow() {
        User user = new User(
                "UserA",
                "aa12",
                "abc@gmail.com",
                true,
                LocalDateTime.now(),
                LocalDateTime.now());

        Set<ConstraintViolation<User>> violations = validator.validate(user);
        
        assertThat(violations.size()).isEqualTo(1);
        assertThat(violations).extracting("message").containsOnly("미래 날짜여야 합니다");
    }

}
```

- `void passwordIsUpper()` : password에 대문자가 입력되어 오류 발생
- `void passwordIsNotMatch()` : password에 $ 같은 허용되지 않은 특수문자가 입력되어 오류 발생
- `void emailIsNull()` : email은 null 값을 허용한다. `@Email` 을 사용할때 주의하자
- `void visitIsFalse()` : false가 입력되어 오류 발생
- `void lastModifiedDateIsPast()` : 과거 시간이 입력 되어 오류 발생
- `void lastModifiedDateIsNow()` : `@Future` 에 OrPresent가 붙지 않았으므로 현재시간을 허용하지 않는다.