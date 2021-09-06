Thymeleaf가 스프링에서 사용할 때 편리한 기능들이 많이 있습니다.

처음 프로젝트를 진행할 때 뷰 템플릿에 대한 기능은 프론트 부분이다 라고 생각하여 기능 구현을 하는데 급급하였습니다.

추후에 잘못된 생각이었다고 깨달으면서 타임리프에 대해 공부하였고, 공부한 내용을 바탕으로 저의 프로젝트 코드들을 리팩토링 해보겠습니다.

<br>

# 1. 템플릿 레이아웃

HTML로 웹 페이지를 만들다 보면 Head 부분에 많은 중복된 코드가 발생하게 됩니다.

웹 폰트, css, 파비콘 같은 자료들은 매 페이지 마다 고정으로 들어가게 되는데 중복 코드를 줄여주기 위해 타임리프는 템플릿 조각 기능을  제공해줍니다.

먼저 기존 코드입니다.

**fragments/header**

```html
<!doctype html>
<html xmlns:th="http://www.thymeleaf.org">
<head th:fragment="header">

    <!-- 모든 장치에서 웹 사이트가 잘 보이도록 뷰포트를 설정하는 에제 -->
    <meta name="viewport" content="user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, width=device-width">
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <!-- 문서의 저자 -->
    <meta name="author" content="charile">
    <!-- 웹 페이지 설명 -->
    <meta name="description" content="찰리 웹사이트">
    <!-- 검색 엔진을 위한 키워드 -->
    <meta name="keywords" content="메가박스, 유투브, 영화, 최신영화, 영화관, CGV, 롯데시네마, 웹스토리보이, 웹스, 사이트 만들기, 따라하기">

    <!-- CSS -->
    <link rel="stylesheet" type="text/css" href="/css/reset.css">
    <link rel="stylesheet" type="text/css" href="/css/swiper.css">
    <link rel="stylesheet" type="text/css" href="/css/font-awesome.css">

    <!-- 파비콘 -->
    <link rel="shortcut icon" type="image/ico" href="/favicon/favicon.ico">
    <link rel="apple-touch-icon-precomposed" type="image/png" href="/favicon/favicon_72.png" />
    <link rel="apple-touch-icon-precomposed" sizes="96x96" type="image/png" href="/favicon/favicon_96.png" />
    <link rel="apple-touch-icon-precomposed" sizes="144x144" type="image/png" href="/favicon/favicon_144.png" />
    <link rel="apple-touch-icon-precomposed" sizes="192x192" type="image/png" href="/favicon/favicon_192.png" />

    <!-- 자바스크립트 라이브러리  -->
    <script type="text/javascript" src="/js/jquery-3.5.1.min.js"></script>
    <script type="text/javascript" src="/js/modernizr-custom.js"></script>
    <script type="text/javascript" src="/js/ie-checker.js"></script>
    <script type="text/javascript" src="/js/swiper.min.js"></script>
    <script type="text/javascript" src="/js/iframe_api.js"></script>
    <script type="text/javascript" src="/js/slick.min.js"></script>

    <!-- 웹 폰트 -->
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@100;300;400;500;700;900&display=swap" rel="stylesheet">
</head>
```

`<head th:fragment="header">`  **"header"** 라는 템플릿 조각 생성

**index.html**

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org" xmlns:sec="http://www.w3.org/1999/xhtml">
<head>
    <div th:replace="fragments/header :: header"></div>
	<title>찰리집</title>
    <link rel="stylesheet" href="css/style.css">
    <link rel="stylesheet" href="css/slick.css">
</head>
```

`<div th:replace="fragments/header :: header"></div>`

`th:replace` 를 이용하면 미리 지정해 놓은 **"header"** 템플릿 조각으로 대체됩니다.

하지만 head 부분을 보면 공통된 부분이외에도 각 페이지마다 title, link 처럼 개별적으로 들어가는 코드가 발생하게 됩니다. 그래서 추가적인 코드를 작성하게 되는데

이런 문제를 해결해 주기 위해 좀 더 개념이 확장된 **템플릿 레이아웃** 기능을 타임리프는 제공해줍니다. 

바로 코드를 보고 설명하겠습니다.

**template/layout/head.html**

```html
<!doctype html>
<html xmlns:th="http://www.thymeleaf.org">
<head th:fragment="common_header(title,links)">

    <title th:replace="${title}">레이아웃 타이틀</title>

    ...

    <!-- 추가 -->
    <th:block th:replace="${links}" />

		...

</head>
```

위의 **fragments/header** 파일에 

`<title th:replace="${title}">레이아웃 타이틀</title>`

`<th:block th:replace="${links}" />`

두가지를 추가해주고

`<head th:fragment="common_header(title,links)">`

템플릿 레이아웃을 새롭게 정의해주었습니다. 

이렇게 괄호안에 파라미터를 넣어주면, 동적으로 title과 link의 값을 받을 수 있습니다.

**index.html**

```html
<head th:replace="template/layout/head :: common_header(~{::title}, ~{::link})">

    <title>찰리집</title>

    <link rel="stylesheet" th:href="@{css/style.css}">
    <link rel="stylesheet" th:href="@{css/slick.css}">

</head>
```

이처럼 사용하면 모든 페이지마다 title을 다르게 정의해줄 수 있고, link 태그도 추가 해 줄 수 있습니다.

템플릿 조각과 템플릿 레이아웃이 둘다 추가적인 코드를 작성하게 되니 비슷해보이지만 템플릿 레이아웃은 좀 더 확장하여 `head` 뿐만 아니라 `html` 전체의 틀을 정해서도 사용할 수 있습니다.

개인적으로는 코드의 추가 없이 재사용에 목적이 더 크다면 템플릿 조각을

구조는 일정한데 부분적인 코드 추가가 필요하면 템플릿 레이아웃을 사용하는걸 권장합니다.

<br>

# 2. 폼 분리

기존에는 회원을 등록하는 것과, 수정하는것을 분리하지 않고 하나의 **Form.html** 로 관리하였습니다.

등록과 수정은 거의 구조와 내용이 비슷하기 때문에 하나의 Form 으로 관리하는게 코드도 줄어들어 좋아보여 합쳐놓았습니다.

하지만 실무에서는 등록과 수정의 요구사항이 다른 경우가 많고, 분리를 하는것이 추후 확장이나 유지 보수 측면에서 유리하다는걸 깨닫고 **addForm**, **editForm** 으로 분리해주었습니다.

<br>

