# 타임리프의 스프링 통합 장점
타임리프는 스프링으로 개발하는 개발자를 위해 많은 편리한 기능을 제공해준다.

스프링 통합으로 추가로 사용할 수 있는 기능들을 알아보자.

<br>

# 입력 폼 처리
웹을 개발하면서 많은 경우 폼을 이용해 데이터를 주고 받게 된다.
이때 타임리프가 지원해주는 `폼 커맨드 객체`를 사용하면 효율적인 코드를 작성할 수 있다.

* `th:object` 커맨드 객체를 지정한다.
* `*{...}` 선택 변수 식이라고 한다. `th:object` 에서 선택한 객체에 접근한다.
* `th:field` HTML 태그의 id, name, value 속성을 자동으로 처리해준다.

<br>

바로 코드를 통해 기능을 살펴보면 이해가 쉬울것이다.

```html
<form action="item.html" th:action th:object="${item}" method="post">
      <div>
          <label for="itemName">상품명</label>
          <input type="text" id="itemName" name="itemName" th:field="*{itemName}" class="form-control" placeholder="이름을 입력하세요">
      </div>
      <div>
          <label for="price">가격</label>
          <input type="text" id="price" th:field="*{price}" class="form-control" placeholder="가격을 입력하세요">
      </div>
      <div>
          <label for="quantity">수량</label>
          <input type="text" th:field="*{quantity}" class="form-control" placeholder="수량을 입력하세요">
      </div>
```

- `th:object="${item}"` : `<form>` 에서 사용할 객체를 지정한다. 선택 변수 식 `(*{...})` 을 적용할 수 있다.
- `th:field="*{itemName}"`
    - `*{itemName}` 는 선택 변수 식을 사용했는데, `${item.itemName}` 과 같다. 앞서 `th:object` 로 `item`을 선택했기 때문에 선택 변수 식을 적용할 수 있다.
    - `th:field` 는 **id, name, value** 속성을 모두 자동으로 만들어준다.
        - `id` : `th:field` 에서 지정한 변수 이름과 같다 `id="itemName"`
        - `name` : `th:field` 에서 지정한 변수 이름과 같다. `name="itemName"`
        - `value` : `th:field` 에서 지정한 변수의 값을 사용한다. `value=""`

**렌더링 전**

```html
<input type="text" id="itemName" name="itemName" th:field="*{itemName}" class="form-control" placeholder="이름을 입력하세요">
<input type="text" id="price" th:field="*{price}" class="form-control" placeholder="가격을 입력하세요">
<input type="text" th:field="*{quantity}" class="form-control" placeholder="수량을 입력하세요">
```

**렌더링 후**

```html
<input type="text" th:field="*{quantity}" class="form-control" placeholder="수량을 입력하세요">
<input type="text" id="price" class="form-control" placeholder="가격을 입력하세요" name="price" value="">
<input type="text" class="form-control" placeholder="수량을 입력하세요" id="quantity" name="quantity" value="">
```

<br>

# 체크 박스 - 단일
단순 HTML 체크 박스
```html
<div>
    <div class="form-check">
        <input type="checkbox" id="open" name="open" class="form-check-input">
        <label for="open" class="form-check-label">판매 오픈</label>
    </div>
</div>
```
기본 HTML이 제공하는 체크박스 타입을 사용하면 체크 박스를 체크했을 때 `open=on` 이라는 값이 Form을 타고 넘어온다.  
그리고 스프링이 `on` 이라는 문자를 `true` 타입으로 변환해 줍니다.

하지만 HTML에서 체크박스를 선택하지 않고 폼을 전송하게 되면 open 이라는 필드 자체가 서버로 전송이 되지 않게 됩니다.

이럴 경우 서버에서는 사용자가 의도적으로 값을 넘기지 않은것인지 실수로 아무 값도 넘어오지 않은것인지 판단을 할 수 가 없습니다.

이런 문제를 해결하기 위해 스프링은 히든 필드를 하나 만들어서 `_open` 처럼 기존 체크 박스 이름 앞에 언더스코어(_) 를 붙여서 전송합니다.
히든 필드는 항상 전송된다. 따라서 체크를 해제한 경우 `open`은 전송되지 않고, `_open`만 전송되는데, 이 경우 스프링 MVC는 체크를 해제했다고 판단한다.

<br>

**기존 코드에 히든 필드 추가**
```html
<div>
    <div class="form-check">
        <input type="checkbox" id="open" name="open" class="form-check-input">
        <input type="hidden" name="_open" value="on"/> <!-- 히든 필드 추가 -->
        <label for="open" class="form-check-label">판매 오픈</label>
    </div>
</div>
```

타임리프의 `th:field` 를 사용하면 히든 필드를 자동으로 추가해 준다.
즉, 개발자가 매번 히든 필드를 신경쓸 필요가 없게 되는 것이다.

**타임리프 - 체크 박스**
```html
<div>
    <div class="form-check">
        <input type="checkbox" id="open" th:field="*{open}" class="form-check-input">
        <label for="open" class="form-check-label">판매 오픈</label>
    </div>
</div>
```

<br>

# 체크 박스 - 멀티
단일 체크박스를 사용하는 것이 아닌 다중 선택일 경우를 살펴보자.

**Controller**
```Java
public Map<String, String> regions() {
    Map<String, String> regions = new LinkedHashMap<>();
    regions.put("SEOUL", "서울");
    regions.put("BUSAN", "부산");
    regions.put("JEJU", "제주");
    return regions;
}
```

체크박스로 서울, 부산, 제주 중에서 다중으로 선택할 수 있는 경우를 가정해보자.

```html
<div>
    <div>등록 지역</div>
    <div th:each="region : ${regions}" class="form-check form-check-inline">
        <input type="checkbox" th:field="*{regions}" th:value="${region.key}" class="form-check-input">
        <label th:for="${#ids.prev('regions')}"
               th:text="${region.value}" class="form-check-label">서울</label>
    </div>
</div>
```
단일과 다르게 다중으로 선택할 경우 주의해야 할 점은 HTML 속성의 태그중 `id`는 모두 달라야 한다는 점이다.  
그리고 타임리프의 `th:field` 를 사용하면 `th:each`를 통해 반복할 때 임의로 숫자 1,2,3 을 뒤에 붙여준다.

또한 `id` 가 타임리프에 의해 동적으로 만들어지기 때문에 label 태그의 id 값을 임의로 지정하는 것이 불가능하다. 그래서 타임리프는 `th:for="${#ids.prev('regions')}"` 를 제공해준다.

`ids.prev` 는 자신의 태그 기준으로 이전의 id를 사용하고  
`ids.next` 는 자신의 태그 기준으로 이후의 id를 사용한다.

<br>

위의 html 코드 렌더링 결과
```html
<div>
    <div>등록 지역</div>
    <div class="form-check form-check-inline">
        <input type="checkbox" value="SEOUL" class="form-check-input" id="regions1" name="regions"><input type="hidden" name="_regions" value="on"/>
        <label for="regions1"
               class="form-check-label">서울</label>
    </div>
    <div class="form-check form-check-inline">
        <input type="checkbox" value="BUSAN" class="form-check-input" id="regions2" name="regions"><input type="hidden" name="_regions" value="on"/>
        <label for="regions2"
               class="form-check-label">부산</label>
    </div>
    <div class="form-check form-check-inline">
        <input type="checkbox" value="JEJU" class="form-check-input" id="regions3" name="regions"><input type="hidden" name="_regions" value="on"/>
        <label for="regions3"
               class="form-check-label">제주</label>
    </div>
</div>
```
id를 보면 regions 뒤에 숫자 1,2,3 이 차례로 붙은걸 확인할 수 있다.  
추가로 label 태그를 보면 이전의 id를 그대로 가져와 사용하는 걸 확인할 수 있다.

