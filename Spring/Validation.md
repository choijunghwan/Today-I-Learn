**컨트롤러의 중요한 역할중 하나는 HTTP 요청이 정상인지 검증하는 것이다**. 그리고 정상 로직보다 이런 검증 로직을 잘 개발하는 것이 더 중요할 수 있다.

대표적인 검증 방법은 Bean Validation 이다.
그치만 바로 Bean Validation을 알아보기 전에 Validation을 직접 처리해보고 스프링을 이용한 Validation을 처리해보면서 하나씩 발전 시켜 나가보자.

<br>

**요구사항: 검증 로직**
- 타입 검증
    - 가격, 수량에 문자가 들어가면 검증 오류 처리
- 필드검증
    - 상품명: 필수, 공백X
    - 가격: 1000원 이상, 1백만원 이하
    - 수량: 최대 9999
- 특정 필드의 범위를 넘어서는 검증
    - 가격 * 수량의 합은 10,000원 이상

<br>

# 검증 직접 개발

**ValidationItemController**
```Java
@PostMapping("/add")
public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes, Model model) {

    //검증 오류 결과를 보관
    Map<String, String> errors = new HashMap<>();

    // 검증 로직
    if (!StringUtils.hasText(item.getItemName())) {
        errors.put("itemName", "상품 이름은 필수입니다.");
    }
    if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
        errors.put("price", "가격은 1,000 ~ 1,000,000 까지 허용합니다.");
    }
    if (item.getQuantity() == null || item.getQuantity() >= 9999) {
        errors.put("quantity", "수량은 최대 9,999 까지 허용합니다.");
    }

    //특정 필드가 아닌 복합 룰 검증
    if (item.getPrice() != null && item.getQuantity() != null) {
        int resultPrice = item.getPrice() * item.getQuantity();
        if (resultPrice < 10000) {
            errors.put("globalError", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice);
        }
    }

    //검증에 실패하면 다시 입력 폼으로
    if (!errors.isEmpty()) {
        log.info("errors = {}", errors);
        model.addAttribute("errors", errors);
        return "validation/v1/addForm";
    }

    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v1/items/{itemId}";
}
```

`Map<String, String> errors = new HashMap<>();`
만약 오류가 발생하면 어떤 검증 오류가 발생했는지 정보를 담아준다.

`StringUtils.hasText()` 를 이용하면 값이 있으면 True, 없으면 False를 반환해준다.

**특정 필드의 범위를 넘어서는 검증 로직**
```Java
if (item.getPrice() != null && item.getQuantity() != null) {
    int resultPrice = item.getPrice() * item.getQuantity();
    if (resultPrice < 10000) {
        errors.put("globalError", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice);
    }
}
```
특정 필드를 넘어서는 오류를 처리해야 할 수도 있다. 이때는 필드 이름을 넣을 수 없으므로 `globalError`로 넣어준다.

<br>

**검증에 실패하면 다시 입력 폼으로**
```Java
if (!errors.isEmpty()) {
    log.info("errors = {}", errors);
    model.addAttribute("errors", errors);
    return "validation/v1/addForm";
}
```
오류 메시지가 하나라도 있으면 오류 메시지를 출력하기 위해 Model에 errors를 담아서 입력 폼이 있는 뷰 템플릿으로 보낸다.

<br>

# BindingResult
이제 스프링이 제공하는 검증 오류 처리 방법을 알아보자.

직접 검증 오류를 처리할 때는 **Map** 자료구조를 통해 오류메시지를 넘겨주었다면 스프링은 `Error`, `BindingResult` 라는 객체를 제공해준다.

`BindingResult` 객체는 검증 오류를 보관하는 객체이다. 또한 `BindingResult`가 있으면 `@ModelAttribute` 에 데이터 바인딩시 오류가 발생해도 컨트롤러가 호출된다.

예) @ModelAttribute에 바인딩시 타입 오류가 발생하면?

- `BindingResult`가 없으면 → 400 오류가 발생하면서 컨트롤러가 호출되지 않고, 오류 페이지로 이동한다.
- `BindingResult`가 있으면 → 오류 정보(`FieldError`)를 `BindingResult` 에 담아서 컨트롤러를 정상 호출한다.

<br>

위에 직접 검증 오류를 처리한 코드를 스프링이 제공하는 `BindingResult`를 이용해 변경해보자.

```Java
@PostMapping("/add")
public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

    // 검증 로직
    if (!StringUtils.hasText(item.getItemName())) {
        bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));
    }
    if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
        bindingResult.addError(new FieldError("item", "price", "가격은 1,000 ~ 1,000,000 까지 허용합니다."));
    }
    if (item.getQuantity() == null || item.getQuantity() >= 9999) {
        bindingResult.addError(new FieldError("item", "quantity", "수량은 최대 9,999 까지 허용합니다."));
    }

    //특정 필드가 아닌 복합 룰 검증
    if (item.getPrice() != null && item.getQuantity() != null) {
        int resultPrice = item.getPrice() * item.getQuantity();
        if (resultPrice < 10000) {
            bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
        }
    }

    //검증에 실패하면 다시 입력 폼으로
    if (bindingResult.hasErrors()) {
        log.info("errors = {}", bindingResult);
        return "validation/v2/addForm";
    }

    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v2/items/{itemId}";
}
```
`@ModelAttribute Item item, BindingResult bindingResult`  
`BindingResult` 객체를 선언해 주었다. 여기서 조심해야 할 점이 있다.
`BindingResult` 파라미터는 는 `@ModelAttribute` 바로 다음에 와야한다.

**필드 오류 - FieldError**
```Java
if (!StringUtils.hasText(item.getItemName())) {
    bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));
}
```

<br>

**FieldError 생성자 요약**

```java
public FieldError(String objectName, String field, String defaultMessage) {}
```

필드에 오류가 있으면 FieldError 객체를 생성해서 bindingResult 에 담아두면 된다.

- `objectName` : `@ModelAttribute` 이름
- `field` : 오류가 발생한 필드 이름
- `defaultMessage` : 오류 기본 메시지

<br>

**글로벌 오류 - ObjectError**

```java
bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
```

**ObjectError 생성자 요약**

```java
public ObjectError(String objectName, String defaultMessage) {}
```

특정 필드를 넘어서는 오류가 있으면 ObjectError 객체를 생성해서 bindingResult 에 담아두면 된다.

- `objectName` : `@ModelAttribute`의 이름
- `defaultMessage` : 오류 기본 메시지


<br><br>

# BindingResult 오류메시지 남기기
이제 오류 검증 처리를 완료했다. 근데 아직 문제가 남아있다.

예를 들어 사용자가 item을 저장할려고 할때 상품이름만 입력을 안하고 나머지 정보는 모두 입력했을 경우 정상적으로 오류를 잡아내지만 오류를 잡아낸뒤에 사용자에게 다시 화면을 보여줄때 사용자가 입력했던 모든 정보들이 남아있지 않고 날라가게 된다.

사용자에게 좋은 사용 경험을 주기위해서는 오류메시지를 보여주면서 입력한 정보들을 남겨주는것이 좋다.

<br>

오류 메시지를 남기는 법을 알아보자.

**FieldError 생성자**

```java
public FieldError(String objectName, String field, String defaultMessage);
public FieldError(String objectName, String field, @Nullable Object 
				rejectedValue, boolean bindingFailure, @Nullable String[] codes, @Nullable
				Object[] arguments, @Nullable String defaultMessage)
```

파라미터 목록

- `objectName` : 오류가 발생한 객체 이름
- `field` : 오류 필드
- `rejectedValue` : 사용자가 입력한 값(거절된 값)
- `bindingFailure` : 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분 값
- `codes` : 메시지 코드
- `arguments` : 메시지에서 사용하는 인자
- `defaultMessage` : 기본 오류 메시지

`ObjectError` 도 유사하게 두가지 생성자를 제공한다.

<br>

**오류 발생시 사용자 입력 값 유지**
```Java
new FieldError("item", "itemName", item.getItemName(), false, null, null, "상품 이름은 필수입니다.");

new FieldError("item", "price", item.getPrice(), false, null, null, "가격은 1,000 ~ 1,000,000 까지 허용합니다.");
```

<br>

# 오류 코드 메시지 처리
FieldError, ObjectError의 생성자는 errorCode, arguments 를 제공한다. 이것은 오류 발생시 오류코드로 메시지를 찾기 위해 사용된다.

<br>

스프링 부트 메시지 설정 추가
**application.properties**

```java
spring.messages.basename=messages,errors
```

<br>

**errors.properties 추가**

src/main/resources/errors.properties

```
required.item.itemName=상품 이름은 필수입니다.
range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
max.item.quantity=수량은 최대 {0} 까지 허용합니다.
totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}
```
메시지 설정을 추가하고 errors.properties를 추가하면 메시지를 등록해서 사용할 수 있다.

<br>
등록한 오류 코드 메시지 처리

```Java
new FieldError("item", "itemName", item.getItemName(), false, new String[]{"required.item.itemName"}, null, null);

new FieldError("item", "price", item.getPrice(), false, new String[]{"range.item.price"}, new Object[]{1000, 1000000}, null);
```

- `codes` : `required.item.itemName` 를 사용해서 메시지 코드를 지정한다. 메시지 코드는 하나가 아니라 배열로 여러 값을 전달할 수 있는데, 순서대로 매칭해서 처음 매칭되는 메시지가 사용된다.
- `arguments` : `Object[] {1000, 1000000}` 를 사용해서 코드의  `{0}` , `{1}` 로 치환할 값을 전달한다.


<br><br>

하지만 현재 `FieldError`, `ObjectError` 는 다루기 너무 번거롭고 코드가 길다.

컨트롤러에서 `BindingResult`는 검증해야 할 객체인 `target` 바로 다음에 온다. 따라서 `BindingResult` 는 이미 본인이 검증해야 할 객체인 `target`을 알고 있다.

`BindingResult` 가 제공하는 `rejectValue()`, `reject()` 를 사용하면 `FieldError`, `ObjectError` 를 직접 생성하지 않고, 깔끔하게 검증 오류를 다룰 수 있다.

**rejectValue, reject 적용**
```Java
if (!StringUtils.hasText(item.getItemName())) {
    bindingResult.rejectValue("itemName", "required");
}

if (item.getPrice() != null && item.getQuantity() != null) {
    int resultPrice = item.getPrice() * item.getQuantity();
    if (resultPrice < 10000) {
        bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
    }
}
```
필드오류는 `rejectValue()`를 글로벌 오류는 `reject()`를 사용한다.

rejectValue의 "required"를 통해 errors.properties에 있는 required.item.itemName 을 호출하고

reject의 "totalPriceMin"을 통해 errors.properties에 있는 totalPriceMin 을 호출해 오류 메시지를 처리해준다.

오류 메시지 코드를 처리하는 자세한 메커니즘은 `MessageCodesResolver`를 검색해서 찾아보면 자세한 내용을 알 수 있다.

<br><br>

# Bean Validation
Bean Validation은 검증 애노테이션과 여러 인터페이스의 모음이다.

Bean Validation을 통해 애노테이션으로 쉽게 검증이 가능하다.

Bean Validation을 사용하기 위해 의존관계를 추가해주자.

**Build.gradle**
```Java
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

사용할 검증 애노테이션
- `@NotBlank` : 빈값 + 공백만 있는 경우를 허용하지 않는다.
- `@NotNull` : null을 허용하지 않는다.
- `@Range(min = 1000, max = 1000000)` : 범위 안의 값이어야 한다.
- `@Max(9999)` : 최대 9999까지만 허용한다.

이것말고도 다양한 검증 애노테이션이 있다.  
다른 애노테이션은 링크를 참고하자.

검증 애노테이션 모음 : https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-defineconstraints-spec 

<br><br>

**Item - Bean Validation 애노테이션 적용**
```Java
package hello.itemservice.domain.item;
import lombok.Data;
import org.hibernate.validator.constraints.Range;
import javax.validation.constraints.Max;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotNull;
@Data
public class Item {
		private Long id;

		@NotBlank
		private String itemName;

		@NotNull
		@Range(min = 1000, max = 1000000)
		private Integer price;

		@NotNull
		@Max(9999)
		private Integer quantity;

		public Item() {
		}

		public Item(String itemName, Integer price, Integer quantity) {
				this.itemName = itemName;
				this.price = price;
				this.quantity = quantity;
		}
}
```

**ValidationItemController - Bean Validation 적용**
```Java
@PostMapping("/add")
public String addItem(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

//특정 필드가 아닌 복합 룰 검증
if (item.getPrice() != null && item.getQuantity() != null) {
    int resultPrice = item.getPrice() * item.getQuantity();
    if (resultPrice < 10000) {
        bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
    }
}

//검증에 실패하면 다시 입력 폼으로
if (bindingResult.hasErrors()) {
    log.info("errors = {}", bindingResult);
    return "validation/v3/addForm";
}

Item savedItem = itemRepository.save(item);
redirectAttributes.addAttribute("itemId", savedItem.getId());
redirectAttributes.addAttribute("status", true);
return "redirect:/validation/v3/items/{itemId}";
}
```

**스프링 MVC는 어떻게 Bean Validator를 사용?**

스프링 부트가 `spring-boot-starter-validation` 라이브러리를 넣으면 자동으로 Bean Validator를 인지하고 스프링에 통합한다.

**스프링 부트는 자동으로 글로벌 Validator로 등록한다.**

`LocalValidatorFactoryBean`을 글로벌 Validator로 등록한다. 이 Validator는 `@NotNull` 같은 애노테이션을 보고 검증을 수행한다. 이렇게 글로벌 Validator가 적용되어 있기 때문에, `@Valid`, `@Validated` 만 적용하면 된다.

검증 오류가 발생하면, `FieldError`, `ObjectError`를 생성해서 `BindingResult`에 담아준다.

**검증 순서**

1. `@ModelAttribute` 각각의 필드에 타입 변환 시도
    1. 성공하면 다음으로
    2. 실패하면 `typeMismatch`로 `FieldError` 추가
2. Validator 적용

**바인딩에 성공한 필드만 Bean Validation 적용**

BeanValidator는 바인딩에 실패한 필드는 BeanValidation을 적용하지 않는다.

생각해보면 타입 변환에 성공해서 성공한 필드여야 BeanValidation 적용이 의미 있다.

(일단 모델 객체에 바인딩 받는 값이 정상으로 들어와야 검증도 의미가 있다.)

`@ModelAttribute` → 각각의 필드 타입 변환 시도 → 변환에 성공한 필드만 BeanValidation 적용

**예)**

- `itemName` 에 문자 "A" 입력 → 타입 변환 성공 → `itemName` 필드에 BeanValidation 적용
- `price` 에 문자 "A" dlqfur → "A"를 숫자 타입 변환 시도 실패 → typeMismatch FieldError 추가 → `price` 필드는 BeanValidation 적용 X

