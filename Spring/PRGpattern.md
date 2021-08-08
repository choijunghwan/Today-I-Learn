# [ PRG Post/Redirect/Get ]

```java
package hello.itemservice.web.basic;

import hello.itemservice.domain.item.Item;
import hello.itemservice.domain.item.ItemRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import javax.annotation.PostConstruct;
import java.util.List;

@Slf4j
@Controller
@RequestMapping("/basic/items")
@RequiredArgsConstructor
public class BasicItemController {

    private final ItemRepository itemRepository;

    @GetMapping
    public String items(Model model) {
        List<Item> items = itemRepository.findAll();
        model.addAttribute("items", items);
        return "basic/items";
    }

    @GetMapping("/{itemId}")
    public String item(@PathVariable long itemId, Model model) {
        Item item = itemRepository.findById(itemId);
        model.addAttribute("item", item);
        return "/basic/item";
    }

    @GetMapping("/add")
    public String addForm() {
        return "basic/addForm";
    }

    @PostMapping("/add")
    public String addItemV4(Item item) {
        itemRepository.save(item);
        return "basic/item";
    }

    /**
     * 테스트용 데이터 추가
     */
    @PostConstruct
    public void init() {
        itemRepository.save(new Item("itemA", 10000, 10));
        itemRepository.save(new Item("itemB", 20000, 20));
    }
}
```
## 환경 세팅

@GetMapping items  → 상품 목록

@GetMapping("/{itemId}) item → 상품 상세

@GetMapping("/add") addForm → 상품 등록 폼

으로 구성되어 있다.

**전체흐름**

![PRG1](/Img/PRG1.png)

이 그림을 보면 이해가 빠를것이다.

현재 진행하는 방식에는 치명적인 문제가 있다.

<br><br>

**POST 등록 후 새로 고침**

![PRG2](/Img/PRG2.png)

웹 브라우저의 새로 고침은 마지막에 서버에 전송한 데이터를 다시 전송한다.

상품 등록 폼에서 데이터를 입력하고 저장을 선택하며 `POST /add + 상품데이터`를 서버로 전송한다.

이 상태에서 새로 고침을 또 선택하면 마지막에 전송한 `POST /add + 상품데이터`를 서버로 다시 전송하게 된다.

즉, 같은 내용의 데이터가 중복해서 저장되는 것이다.

<br>

이 문제를 해결하기 위해서 Redirect를 사용해야 한다.

<br>


**POST, Redirect GET**

![PRG3](/Img/PRG3.png)

새로 고침 문제를 해결하려면 상품 저장 후에 뷰 템플릿으로 이동하는 것이 아니라, 상품 상제 화면으로 리다이렉트를 호출해주면 된다.

리다이렉트 특성으로 HTTP 요청이 실제로 두번 일어난다. `1. POST /add` 로 요청이 한번 왔지만,

리다이렉트가 실행되면 다시 새로운 `3. Get /items/{id}` 요청이 나가게 된다.

웹 브라우저는 리다이렉트의 영향으로 상품 저장 후에 실제 상품 상세 화면으로 다시 이동한다. 

따라서 마지막에 호출한 내용이 상품 상세 화면이 `GET /items/{id}` 가 되는 것이다.

**redirect 사용**

```java
/*    
@PostMapping("/add")
public String addItemV4(Item item) {
    itemRepository.save(item);
    return "basic/item";
} */

@PostMapping("/add")
public String addItemV5(Item item) {
    itemRepository.save(item);
    return "redirect:/basic/items/" + item.getId();
}
```

상품 등록 처리 이후에 뷰 템플릿이 아니라 상품 상세 화면으로 리다이렉트 하도록 코드를 작성하자. 이런 문제 해결 방식을 PRG Post/Redirect/Get 라 한다.

<br>

> `"redirect:/basic/items/" + item.getId();` 
redirect에서 `item.getId()` 처럼 URL에 변수를 더해서 사용하는 것은 URL 인코딩이 안되기 때문에 위험하다. `RedirectAttributes` 를 사용하자.

<br>

**RedirectAttributes 사용**

```java
/*
@PostMapping("/add")
public String addItemV5(Item item) {
    itemRepository.save(item);
    return "redirect:/basic/items/" + item.getId();
} */

@PostMapping("/add")
public String addItemV6(Item item, RedirectAttributes redirectAttributes) {
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    return "redirect:/basic/items/{itemId}";
}
```

<br>

## PRG 패턴은 완벽하지 않다

만약 요청이 오래 걸려 아직 POST에서 GET 요청으로 바뀌지 못했는데, 이 상황에서 새로고침을 하게 된다면?? 

POST 요청이 반복해서 실행되어 결국 데이터가 중복으로 저장되는 문제가 발생할 것이다.

PRG 패턴은 많은 경우 중복을 막아주지만 완벽한 방법은 아니다.

PRG 외에도 서버에서 중복을 체크하도록 로직을 추가로 개발해 주는것이 좋다.

<br><br>

# [ 뒤로 가기 ]

새로고침은 마지막으로 실행한 행동을 다시 실행주는 것입니다.

그렇다면 뒤로 가기는 어떨까요??

뒤로가기는 새로고침이라는 별개의 기능입니다.

뒤로가기는 과거의 히스토리로 돌아간다고 생각하면 됩니다. 

즉, 과거의 페이지로만 돌아가지 과거의 행동은 전혀 일어나지 않습니다.

---
참고자료 
https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard
