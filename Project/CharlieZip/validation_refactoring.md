# 기존 Validation
현재 제가 구현해놓은 validation을 살펴보고 왜 리팩토링을 할려고 하는지 설명하겠습니다.

회원가입 시 검증 요구 사항

* 필드 검증
  * 이름 : 필수
  * 비밀번호 : 필수, 공백X
  * 생년월일 : 8자리 숫자
  * 성별 : 필수

**MemberForm - 회원가입 저장 폼**
```Java
package study.charlieZip.dto;

import lombok.Getter;
import lombok.Setter;
import study.charlieZip.entity.Gender;

import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Pattern;

@Getter @Setter
public class MemberForm {

    //NotEmpty는 null과 ""을 허용하지 않는다.
    @NotEmpty(message = "회원 이름은 필수 입니다.")
    private String username;

    //NotBlank는 null과 ""와 " "(빈공백문자열)을 허용하지 않는다.
    @NotBlank(message = "비밀번호는 필수 입력 값입니다.")
    private String password;

    @Pattern(regexp = "^[a-zA-z0-9]{8}$", message = "8자리로 입력해주세요.")
    private String date;

    private Gender gender;

}
```

<br>

**MemberController**
```Java
@PostMapping(value = "/members/new")
public String addMember(@Valid MemberForm memberform, Errors errors, Model model) {
    if (errors.hasErrors()) {
        //회원가입 실패시, 입력 데이터를 유지
        model.addAttribute("memberForm", memberform);

        // 유효성 통과 못한 필드와 메시지 핸들링
        Map<String, String> validatorResult = memberService.validateHandling(errors);
        for (String key : validatorResult.keySet()) {
            model.addAttribute(key, validatorResult.get(key));
        }

        return "members/addForm";
    }

    // 비밀번호 암호화
    BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();

    Member member = Member.builder()
            .username(memberform.getUsername())
            .password(passwordEncoder.encode(memberform.getPassword()))
            .date(memberform.getDate())
            .gender(memberform.getGender())
            .build();

    memberService.join(member);
    return "redirect:/";
}
```
회원가입 실패시 발생하는 오류를 `Errors` 객체를 이용해 담고 `Errors`의 내용을 `Model`을 통해 다시 회원가입폼으로 넘겨준다.

<br>

**MemberService**
```Java
/**
* 회원가입시, 유효성 체크
*/
public Map<String, String> validateHandling(Errors errors) {
    Map<String, String> validatorResult = new HashMap<>();

    for (FieldError error : errors.getFieldErrors()) {
        String validKeyName = String.format("valid_%s", error.getField());
        validatorResult.put(validKeyName, error.getDefaultMessage());
    }

    return validatorResult;
}
```
회원가입시 발생한 오류를 **validatorResult** Map 자료구조에 넣어 넘겨주었다.

<br><br>

# 문제점
기존의 코드도 정상작동하지만 검증이 `Service` 단까지 넘어온다는것과 오류메시지를 **Map자료구조**에 담은다음에 일일이 `Model`에 담는 부분이 복잡하다고 느껴진다.

그럼 리팩토링 할 부분을 정리해보자.

### 1. Map자료구조가 아닌 BindingResult 객체를 사용해서 오류메시지를 담아서 넘겨주자.  

검증을 할때 크게 Errors와 BindingResult 객체를 사용한다.  
이 두 객체는 BindingResult는 groups 기능을 추가 제공하는것 말고는 거의 똑같은 기능을 제공하므로 둘중에 아무거나 써도 상관없지만 저는 스프링이 제공하는 BindingResult 객체를 사용하겠습니다.

Bean Validation을 사용하면 `LocalValidatorFactoryBean`을 글로벌 Validator로 등록합니다. 이 Validator는 `@NotNull` 같은 애노테이션을 보고 검증을 수행합니다.

검증 오류가 발생하면 `FieldError`, `ObjectError`를 생성해서 **BindingResult** 에 담아줍니다.

그리고 추가로 오류를 자동으로 Model에 추가해줍니다.

즉 기존에 `Errors`에 있는 오류메시지를 Map자료구조에 담아서 Model에 넣어주는 과정이 `BindingResult`를 사용하므로써 자동으로 처리되게 할 수 있습니다.

참고로 Errors도 BindingResult와 똑같이 작동하지만 기존 구현코드에서는 저의 부족함으로 복잡하게 사용되었습니다.

### 2. 