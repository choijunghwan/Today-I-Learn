AWS Builders 프로그램에서 온라인으로 e-commerce에 대해 발표를 한다고 하는데 커머스 사업에 관심이 있어 신청을 하여 발표를 들었는데요

여러 발표 중 가장 흥미로웠던 무신사가 IDC환경에서 클라우드 환경으로 전환하는 과정을 설명해주는 발표에 대해 정리하겠습니다.

<br><br>

# [ 클라우드로 이전하게 된 계기 ]
![image1](/Img/Musinsa/GoToWebinar.png)

**결정적인 계기**
* 매년 블랙프라이데이 행사 때 트래픽 증가


**기존 IDC의 환경의 문제**
* 유동적인 서버 확장의 어려움 등의 IDC 환경의 문제


**업무 방식의 문제**
* 일에 팀을 맞추게 된다.
  * 예를 들면 새로운 일 A를 추진하게 되면 기존 업무 방식에선 각 팀마다 사람을 뽑고 회의를 통해 새로운 일 A에 대해 정의한뒤 프로젝트를 진행한다.
  * 진행이 완료되면 사람들은 각자 자신의 기존 팀으로 되돌아 간다.
  * 이러한 방식은 매번 새로운 일을 하게 되면 처음부터 다시 시작하게 되어 과거의 경험을 이용하지 못한다.
* 클라우드로 이전한뒤 MSA서비스로 전향한다면
* 팀에 일을 맞출수 있다.
  * 각 팀에 일을 할당할 수 있고, 신규 서비스도 일시적으로 끝나는게 아닌 꾸준히 운영하고, 발전시켜 나갈 수 있다.


<br><br>

# [ IDC -> Cloud 로 이전 ]
### 도전과제

* Environment Challenge
  * 무신사는 Physical 서버를 운영하고 있어 클라우드 환경으로 이전해야 한다.
* Human Resource Challenge
  * 서비스 인프라를 확장하기 위해서는 좋은 개발자(엔지니어가) 필요
  * 많은 서비스 기업(네이버, 카카오, 쿠팡, 배달의민족) 등에서도 엔지니어를 영입하고 있어 인력 채용의 어려움이 있다.

<br>

### 클라우드 이전 과정

**Lift-and-Shift 전략**

* IDC의 환경과 유사하게 클라우드로 이전
  * 빠르게 이전하기 위해 이 방법을 선택, 이전한 이후 개선
* 과도기의 부담 최소화
  * 이전하는 기간이 길어지면 엔지니어들도 많은 혼동이 올 수 있기 때문에 빠르게 이전을 목표
* 필수 적용 사항 선정
  * 필수로 적용해야 하는 사항들을 빠르게 뽑아 적용
  * Auto Scaling, Deploy Process(CodeDeploy)
  * Change(Config) Management,
  * Managed DB (Aurora DB)
  * DNS based Lookup/Routing

<br>

**Migration Steps**

* Move Static Contents(S3 - CDN)
* Build/Migrate Devel Env
* DB Replication Relay to Cloud (via VPN)
* Build Alpha/Product Env
* Testing, Testing, Testing..
* Switch Traffic

<br>

**클라우드 이전 성공 요인**

* 테스트 가능한 환경 (Testable env AEarlyAP)
  * 클라우드를 이전하는 과정에서 IDC와 클라우드의 환경을 완전 똑같게 만들며 이전하였다.
  * IDC에 코드를 추가하면 클라우드에도 똑같이 추가가 되게 하였다.
  * 이 방법을 통해 클라우드로 서버를 배포하기 전날까지도 Test를 시행해 볼 수 있었다.
* DB Replication Relay via VPN
* Amazon Aurora (only MySQL, PostgreSQL)
  * 오로라 DB는 비싸지만 실제로 도입했을때, 얻을 수 있는 이점이 더 크다.
  * 기회가 된다면 꼭 오로라 DB를 도입하길 추천
* Not Only Infra Duty
  * 클라우드 이전 과정은 Infra 부서만의 업무가 아닌 사내 전체의 업무라고 생각하고 모두가 자기 일처럼 뛰어들어서 가능했다.

<br><br>

# [ 2019 BlackFriday ]
클라우드로 이전후 한달만에 처음 맞이하는 블랙프라이데이 행사 결과
![image2](/Img/Musinsa/블랙프라이데이트래픽.png)

**BlackFriday 2019**

* 짧은 운영/ 적응 기간
* Infra & Network 확장, but App은 일부 변화
* Traffic 대응에 적합한 Architecture/SW 변화 필요

App 내부는 일부만 변화된 상황이라 작은 DB 지연 -> 전체 장애로 까지 퍼져나가는 상황.

본격적으로 아키텍쳐와 서비스를 개선

<br><br>

# [ MSA 서비스로 변화 ]
Micro Services로 변화해가기 시작

한번에 모든 시스템을 변환하기는 불가능하여 조그만 부분 부터 개선해 나가기 시작

<br>

**병목 구간 서비스 분리 (장애 격리) - Cloning**
* 주문, 이벤트, 메인 등 병목 구간이 많은 서비스는 Cloning 하여 한곳이 장애가 나도 시스템 전체가 죽지 않도록 하였다.

**큰 서비스 변경 시, 별도 구조 분리 - Refactoring**
* 회원, 전시, 검색, 결제 등 큰 서비스는 별도로 구조를 분리해 리팩토링

**분리된 도메인간 통신**
* 도메인간 Rest API 통신

**DB Splitting**
* 기존의 Legacy를 분리하는 것은 어려워 천천히 개선
* 대신 새로운 서비스는 DB를 분리하여 추가

**CQRS Pattern**
* 로그인 시 회원 등급 게산, 회원 가입 후 후처리 등에 적용할려고 노력

**Cache Strategy**
* Lazy Loading Cache - TTL 기반, 쉬운 적용, 갱신 시 이슈 내포
* **Write Through Cache** - 갱신 로직 필요, CQRS Pattern과 결합한 강력한 구조 가능
* 오리진의 응답속도가 느려지는걸 예측해서 적용하기는 어려움이 있다.
* 서비스 도중 모니터링을 통해 약간 느려지는 모습이 포착되면 Write Through Cache 방식을 적용

<br><br>

# [ 2020 BlackFriday ]
**기존 2019와 기록 비교**

* 결제 금액 743억원 (x 2.3배)
* 결제 약 100만건 (x 1.5)
* HTTP/s 요청 42억 (x 6)
* 사용 AWS 서비스 수 34개(x 1.8)
* Zero DownTime

모든 기록이 2019년보다 증가하였지만 서비스 장애 시간은 0분을 기록했다.

![image3](/Img/Musinsa/GoToWebinar1.png)

* 2019 블랙프라이데이에는 누적 결제액을 10분마다 갱신
* 2020 블랙프라이데이에는 실시간 누적 결제액을 보여주는 도전
* 실시간으로 결제액을 보여주기 위해서는 초당 6만건이 넘는 트래픽을 감당해야 했고 Redis, Lambda, Golang, 별도의 네트워크 VPC 기술을 적용해 문제를 해결

<br><br>

# [ 현재 2021 ~ ]

**효율과 독립성을 높이는데 집중**



**Container**
* 배포 시간을 줄이고, OS Level Resource 절감
* Devel 환경 전환 완료
* Alpha/Production 환경 전환 중

**Event Driven / Event Dus (Kafka - MSK)**
* API기반의 밀결합된 구조에서
* Event 기반의 독립된 개발 환경

**Distributed Tracing**

**ML based photo/Image Processing Tech for Musinsa**

<br><br>

# [ 후기 ]
많은 기업들이 클라우드 서비스로 전환하는데 그 이유에 대해 실제 사례를 가지고 좀더 깊은 이해를 할 수 있는 시간이었다. 각각의 과정에서 어떠한 고민을 하였고 그러한 문제를 어떻게 해결했는지에 대해 들으며 많은걸 배울 수 있었고 공부할게 많다는걸 한번 더 느낄 수 있었다.

발표를 들으면서 처음 들어보는 용어가 한 두개 있었는데 추후에 공부해보고 내 사이드 프로젝트를에 다양한 클라우드 기술을 적용해 보고 싶어졌다. 

