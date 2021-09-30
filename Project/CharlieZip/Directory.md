먼저 현재 저의 패키지 구조를 먼저 보여드리겠습니다.

<br>

**계층형**
```
main
├── java
│   └── stude
│       └── chalieZip
│           ├── config
│           ├── controller
│           ├── service
│           ├── repository
│           ├── dto
│           ├── entity
│           ├── interceptor
│           └── CharlieZipApplicaton.java
└── resources
       └── application.properties
```
저의 패키지 구조는 계층형 구조로 각 계층을 대표하는 디렉터리 기준으로 코드들이 구성됩니다. 

처음에 프로젝트를 시작할때는 매우 작게 규모로 시작했기 때문에 전체적인 구조를 빠르게 파악할 수 있는 계층형 구조를 선택하였습니다.

그런데 프로젝트를 점점 진행하다보니 계층형 구조의 아쉬운 점들이 생겨났습니다.

### 밀집된 클래스
계층형 구조를 사용하다보니 controller, Service, dao등 디렉토리에 매우많은 클래스들이 밀접하게 됩니다. 클래스가 몇개 없을 때는 편하였지만 점차 갯수가 늘어감에 따라 오히려 전체 구조를 파악하는데 어려움이 생겼습니다.

<br>

### 관련 코드의 응집
위의 문제와 이어지는 문제입니다. 만약 `회원`에 관련된 코드를 작성하게 된다면 계층형은 회원의 관련된 코드를 찾는데에 어려움이 발생하게 됩니다.

<br><br>

# 도메인형 구조 - 기본
그러면 계층형 구조를 도메인형으로 변경해 보겠습니다.
```
main
├── java
│   └── stude
│       └── chalieZip
│           ├── member
|           |   ├── entity
|           |   ├── repository
|           |   ├── service
|           |   ├── controller
|           |   ├── exception
|           |   └── dto
│           ├── coffee
|           |   ├── entity
|           |   ├── repository
|           |   ├── service
|           |   ├── controller
|           |   ├── exception
|           |   └── dto
│           ├── reply
|           |   ├── entity
|           |   ├── repository
|           |   ├── service
|           |   ├── controller
|           |   ├── exception
|           |   └── dto
│           ├── config
│           ├── interceptor
│           └── CharlieZipApplicaton.java
└── resources
       └── application.properties
```
도메인 디렉터리 기준으로 구조를 바꿔보았습니다. 도메인형의 장점은 관련된 코드들이 응집해 있는 장점이 있습니다.

그런데 이렇게 도메인형으로 바꾸고 보니 아직 하나 아쉬운점이 있습니다.

`config`와 같은 전체적인 설정을 관리하는 디렉터리들이 따로 관리가 안된다는 점입니다.

그래서 Global 코드들도 한번 분리를 해보겠습니다.

# 도메인형 구조 - 응용
```
main
├── java
│   └── stude
│       └── chalieZip
|           ├── domain
|           |   ├── member
|           |   |   ├── entity
|           |   |   ├── repository
|           |   |   ├── service
|           |   |   ├── controller
|           |   |   ├── exception
|           |   |   └── dto
|           |   ├── coffee
|           |   |   ├── entity
|           |   |   ├── repository
|           |   |   ├── service
|           |   |   ├── controller
|           |   |   ├── exception
|           |   |   └── dto
│           |   └── reply
|           |       ├── entity
|           |       ├── repository
|           |       ├── service
|           |       ├── controller
|           |       ├── exception
|           |       └── dto
|           ├── global
|           |   ├── common  
|           |   |   └── GlobalConst.java
│           |   ├── config
│           |   ├── error
│           |   └── interceptor
│           └── CharlieZipApplicaton.java
└── resources
       └── application.properties
```
도메인을 담당하는 디렉터리는 domian, 전체적인 부분에 영향을 미치거나 설정을 관리하는 global로 나누었습니다.

`member` 디렉터리 부터 간단하게 설명하겠습니다.
* entity : 엔티티에 대한 클래스로 구성됩니다.
* repository, service, controller : 각 계층의 클래스로 구성됩니다.
* dto : 주로 Request, Response 객체들로 구성됩니다. 저는 주로 Form 객체들로 구성하였습니다.
* exception : 해당 도메인에서 발생하는 예외로구성합니다.

`global`은 프로젝트 전방위적으로 사용되는 객체들로 구성합니다.
* common : 공통으로 사용되는 객체들로 구성합니다.
  * 공통으로 사용 되는 상수, Form 객체도 여기에 해당됩니다.
  * 공통으로 사용되는 Request, Response 객체도 여기에 놓으면 됩니다.
* config : 스프링 설정으로 구성합니다.
* interceptor : 인터셉터 부분 클래스로 구성합니다.
* error : 전체적인 예외 or 예외 핸들링 부분으로 구성합니다.

<br>

확실히 계층형에서 도메인형으로 구조를 바꾸고 추가로 도메인과 글로벌한 부분을 나누니 개인적으로는 좀 더 깔끔해진것 같습니다.

가장 애매했던 부분이 interceptor 부분이었습니다.

이걸 config로 묶을까, common 부분에 넣을까 등등의 고민을 하였는데 현재는 interceptor 부분을 따로 빼는걸로 결정하였습니다.

이러한 부분은 사실 취향차이라고 생각하고 프로젝트를 좀 더 진행하다보면 또 바꿀수도 있는 부분이라 크게 신경쓰지 않았습니다. 

<br><br>

# 정리
도메인형 기준으로 디렉터리를 구성해보니 프로젝트가 규모가 커가면서 좀더 체계적이다라는 느낌을 받았습니다. 반면에, 도메인에 속하지 않는 `common`, `config`, `error`, `interceptor` 이런 부분에 대한 추가적인 고민이 생겼습니다. 하지만 이러한 부분은 자신의 프로젝트에 맞춰 구성해 나가면 되는 부분이라고 생각하고 현재 계층형 구조를 사용하고 계신 분이 있다면 한번 도메인형 구조도 사용해보시길 추천드립니다.



