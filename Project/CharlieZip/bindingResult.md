# 문제
기존에 프로젝트를 진행할때 클라이언트로 부터 오는 요청데이터를 Bean Validation한후 Error가 있다면 bindingResult에 에러 내용을 담아 다시 넘겨주었다.

그런데 댓글 작성을 구현할때 요청데이터를 validation(유효성 검사)를 한뒤 bindingResult에 에러내용을 담아 Json응답으로 반환하니 에러메시지가 의도와 다르게 전달되는것을 확인할 수 있었다.

<br>

### 댓글 작성 소스

**ReplyRequest 요청 DTO**
```Java
@Getter
@Setter
public class ReplyRequest {

    @NotNull
    private Long bno;

    @NotEmpty
    @Length(min = 4)
    private String comment_content;

}
```

<br>

**error.properties**
```
Length=입력 문자의 길이가 부족합니다.
```
`Length`의 오류 메시지 내용을 MessageSource를 이용하여 정해두었다.

<br>

**ReplyController**

```Java
@PostMapping("/new")
public void addComment(@ModelAttribute("insertData") @Validated ReplyRequest replyReq, BindingResult bindingResult,
                        @SessionAttribute(name = GlobalConst.LOGIN_MEMBER, required = false) Member loginMember) throws BindException{
    if (bindingResult.hasErrors()) {
        log.info("댓글 에러 {}", bindingResult);
        throw new BindException(bindingResult);
    }
    Long replyId = replyService.saveReply(replyReq.getBno(), replyReq.getComment_content(), loginMember.getUsername());
}
```

<br>

### 예상 시나리오
요청 `/new?bno=4&comment_content=11`라고 데이터가 들어온다면

`comment_content`필드가 최소길이 4를 만족하지 못하기 때문에 에러 메시지로 `"입력 문자의 길이가 부족합니다"` 가 응답데이터로 넘어가는것을 예상하였습니다.

<br>

### 실제
Postman을 이용하여 데이터를 전송해본 결과
``` Json
{
    "timestamp": "2021-11-20T09:50:24.985+00:00",
    "status": 400,
    "error": "Bad Request",
    "trace": "org.springframework.validation.BindException: org.springframework.validation.BeanPropertyBindingResult: 1 errors\nField error in object 'insertData' on field 'comment_content': rejected value [11]; codes [Length.insertData.comment_content,Length.comment_content,Length.java.lang.String,Length]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [insertData.comment_content,comment_content]; arguments []; default message [comment_content],2147483647,4]; default message [길이가 4에서 2147483647 사이여야 합니다]\r\n\tat study.charlieZip.domain.reply.controller.ReplyController.addComment(ReplyController.java:47)
    .
    .
    .,
    "message": "Validation failed for object='insertData'. Error count: 1",
    "errors": [
        {
            "codes": [
                "Length.insertData.comment_content",
                "Length.comment_content",
                "Length.java.lang.String",
                "Length"
            ],
            "arguments": [
                {
                    "codes": [
                        "insertData.comment_content",
                        "comment_content"
                    ],
                    "arguments": null,
                    "defaultMessage": "comment_content",
                    "code": "comment_content"
                },
                2147483647,
                4
            ],
            "defaultMessage": "길이가 4에서 2147483647 사이여야 합니다",
            "objectName": "insertData",
            "field": "comment_content",
            "rejectedValue": "11",
            "bindingFailure": false,
            "code": "Length"
        }
    ],
    "path": "/comments/new"
}
```

그러나 실제로 Json응답된 내용을 보면 

`defaultMessage": "길이가 4에서 2147483647 사이여야 합니다`
로 직접 설정해둔 오류메시지가 넘어가는 것이 아닌 기본 메시지가 넘어가는 것을 확인할 수 있었다.

<br><br>

# 해결과정
왜 기존의 프로젝트를 진행할때는 잘 되던 MessageSource 기능이 안될까 생각해보았다. 

생각을 해보니 기존에는 api를 통한 Json으로 값을 넘겨주는 것이 아닌 view(뷰)를 반환해준다는 차이가 있었다. 또한 저는 뷰 템플릿을 `타임리프(thymeleaf)`를 사용하는데 타임리프는 내부에서 스프링의 MessageSource와 비교해서 오류 메시지를 찾아 값을 반환해주고 있었다.

내가 생각지도 못한 부분을 이미 자동으로 지원해주고 있었던 것이었다.

그래서 Json으로 응답할때는 타임리프가 자동으로 해주던 MessageSource와의 매핑을 내가 직접해주어 하는것이다.

그럼 한번 직접 매핑을 해주어 Json으로 내가 의도한 `Length`의 오류 메시지를 반환해보겠다.

<br>

### Api 오류 메시지 반환 객체 생성
먼저 오류가 발생했을때 오류메시지를 담아 반환해줄 객체를 생성한다.

**FieldErrorDetail**
```Java
@Getter
@Setter
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class FieldErrorDetail {

    private String field;
    private String code;
    private Object rejectedValue;
    private String message;

    public static FieldErrorDetail create(FieldError fieldError, MessageSource messageSource, Locale locale) {
        return new FieldErrorDetail(
                fieldError.getField(),
                fieldError.getCode(),
                fieldError.getRejectedValue(),
                messageSource.getMessage(fieldError, locale));
    }

}
```
내가 의도한대로 오류메시지가 잘 생성되었는지를 확인하기 위해 총 4개의 정보를 출력해보기 했다.
* field -> 에러가 발생한 필드이름
* code -> 에러 코드 (ex. Length)
* rejectedValue -> 에러가 발생하게 된 Value값
* message -> 에러 메시지

여기서 중요한 부분이 `messageSource.getMessage(fieldError, locale)`이다.

기존에 타임리프가 자동으로 해주던 오류메시지 매핑을 이제 우리가 직접 매핑을 해주는것이다. 아까 `error.properties`에서 정의해놓은 오류메시지를 `messageSource`를 통해 찾을 수 있다.

messageSource가 fieldError를 갖고 메시지를 찾을 수 있는 이유는 FieldError는 `DefaultMessageSourceResolvable`를 구현하고 있다.

![image1](/Img/bindingResult.png)

위의 그림을 보면 `DefaultMessageSourceResolvable`를 구현하고 있는것을 확인할 수 있다.

**DefaultMessageSourceResolvable**
```Java
public class DefaultMessageSourceResolvable implements MessageSourceResolvable, Serializable {

	@Nullable
	private final String[] codes;

	@Nullable
	private final Object[] arguments;

	@Nullable
	private final String defaultMessage;
}
```
그리고 `DefaultMessageSourceResolvable`를 보면 오류메시지를 찾기 위한 `codes`,`arguments`,`defaultMessage` 정보를 갖고있는것을 확인해 볼 수 있다.

<br>

에러가 하나가 아닌 여러개가 발생할 수 있으므로 `FieldErrorDetail` 여러개를 저장할 List 형태로 만들어준다.

<br>

**ValidationResult**
```Java
@Getter
@Setter
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class ValidationResult {
    private List<FieldErrorDetail> errors;

    public static ValidationResult create(Errors errors, MessageSource messageSource, Locale locale) {
        List<FieldErrorDetail> details = errors.getFieldErrors()
                .stream()
                .map(error -> FieldErrorDetail.create(error, messageSource, locale))
                .collect(Collectors.toList());
        return new ValidationResult(details);
    }
}
```

<br>

### 오류메시지 출력
ExceptionHandler를 통해 우리가 생성한 오류 객체를 반환하도록 설정해준다.

```Java
@ExceptionHandler(BindException.class)
public ValidationResult handleBindException(BindException bindException, Locale locale) {
    return ValidationResult.create(bindException, messageSource, locale);
}
```

<br>

요청 : `/new?bno=4&comment_content=11`

결과
```Json
{
    "errors": [
        {
            "field": "comment_content",
            "code": "Length",
            "rejectedValue": "11",
            "message": "입력 문자의 길이가 부족합니다."
        }
    ]
}
```

이제 내가 원하는 오류메시지가 잘 출력되는것을 확인 할 수 있다.

<br><br>

# 정리
타임리프라는 뷰템플릿을 사용하지 않은 Api통신에서는 내가 설정해둔 오류메시지를 출력해주기 위해서는 내가 직접 오류메시지를 매핑해줘야 한다는 것을 알 수 있었다.

그리고 `ValidationResult`와 같이 공통된 객체를 만들어 사용하면 의도한 오류메시지를 쉽게 전달 할 수 있을거 같다.

또한 타임리프를 사용할때는 당연하게 사용하던 기능이라 내부에서 MessageSource를 찾아주는 처리를 하는지 몰랐다. 앞으로는 기능을 당연하다고 사용하는 것이 아닌 내부에서 어떤 처리를 하는지 살펴보는 습관을 들여야겠다.

