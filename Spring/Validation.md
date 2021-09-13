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

`StringUtils.hasText()` 를 이용하면 값이 있으면 True, 없으면 False를 반환해준다.

`Map<String, String> errors = new HashMap<>();`
만약 오류가 발생하면 어떤 검증 오류가 발생했는지 정보를 담아준다.

```Java
if (!errors.isEmpty()) {
    log.info("errors = {}", errors);
    model.addAttribute("errors", errors);
    return "validation/v1/addForm";
}
```
오류 메시지가 하나라도 있으면 오류 메시지를 출력하기 위해 Model에 errors를 담아서 입력 폼이 있는 뷰 템플릿으로 보낸다.

