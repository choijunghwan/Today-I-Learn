# RDS

RDS(Relational Database Service)는 관계형 데이터베이스 RDBMS를 손쉽게 생성하고 확장할 수 있는 서비스입니다.

> **프리티어**
RDS는 프리 티어에서 무료로 사용할 수 있습니다. 마이크로 DB 인스턴스 750시간, 데이터베이스 스토리지 20GB와 1000만 I/O, DB 백업 및 스냅샷 스토리지 20GB를 무료로 사용할 수 있습니다.
> 

RDS를 사용해야 하는 이유는 성능, 편의성, 시간 저약과 비용 절감에 있습니다.

RDS를 이용하면 클릭 몇 번 만으로 손쉽게 DB 인스턴스를 생성할 수 있고, 사용량이 늘어나면 스토리지 용량과 IOPS를 증가시켜 성능 확장이 가능합니다. 또한 장애가 발생해도 Failover 기능을 통해 

안정성이 높은 가용성과 내구성을 제공합니다. 그리고 **Read Replica**를 이용하여 읽기 성능을 향상시킬 수 있습니다.

**RDS 데이터베이스 엔진**

Amazon RDS는 여러 데이터베이스 인스턴스 유형으로 제공되며 Amazon Aurora, MySQL, MariaDB, ORACLE, PostgreSQL, SQL Server 의 데이터베이스 엔진 중에서 선택할 수 있습니다.

<br><br>

# RDS 생성

![image1](/Img/RDS/RDS1.png)

먼저 Amazon RDS 서비스에 접속하여 데이터베이스 생성을 클릭합니다.

<br>

![image2](/Img/RDS/RDS2.png)

데이터베이스 엔진의 옵션을 선택합니다. RDS는 데이터베이스 엔진마다 라이선스 모델이 다릅니다. 또한 같은 데이터베이스 엔진이라도 라이선스 모델의 종류에 따라 사용 요금이 달라질 수 있습니다.

자신의 사용목적과, 환경에 맞춰 적절한 엔진을 선택합니다.

<br>

![image3](/Img/RDS/RDS3.png)

템플릿

- 프로덕션 : 실제 운영 서비스에 알맞는 템플릿입니다. Multi-AZ을 지원합니다.
- 개발/테스트 : 개발 및 테스트 하기에 알맞는 템플릿입니다. 프로덕션보다 성능이 떨어집니다.
- 프리 티어 : RDS 프리 티어를 사용

<br>

![image4](/Img/RDS/RDS4.png)

- DB 인스턴스 식별자 : DB 인스턴스 이름을 입력해줍니다.
- 마스터 사용자 이름: DB 인스턴스에 접속할 때 사용할 아이디
- 마스터 암호 : 마스터 아이디의 비밀번호

RDS는 EC2와 비슷하게 인스턴스라는 개념이 있습니다. RDS DB 인스턴스도 EC2 인스턴스 처럼 여러 사양으로 나뉘어져 있습니다. 이것을 RDS에서는 DB 인스턴스 클래스라고 합니다.

RDS는 사용한 만큼 요금을 내기 때문에 자신의 사용에 맞게 클래스를 선택하면 됩니다.

- 스탠다드 클래스 : vCPU, 메모리, 네트워크 등이 평균적인 사양으로 제공됩니다.
- 메모리 최적화 클래스 : 다른 인스턴스 클래스보다 메모리 용량이 훨씬 큽니다.
- 버스터블 클래스 : 낮은 vCPU 성능과 적은 메모리를 제공합니다. 프리티어에서는 이 인스턴스 유형을 무료로 사용할 수 있습니다.

<br>

![image5](/Img/RDS/RDS5.png)

**스토리지 유형**

- 범용 SSD : 평균적인 성능을 제공합니다. 3 IOPS/Gib가 기본적으로 제공됩니다.
- 프로비저닝된 IOPS : I/O를 많이 사용하는 데이터베이스에 적합합니다. 1,000~80,000 IOPS 를 제공합니다.
- 마그네틱 : 제일 낮은 성능을 제공합니다.

다중 AZ 배포

- Availability Zone(가용 영역)은 다중으로 구성하면 하나의 RDS가 계획되지 않게 중단이 발생하여도 다른 가용영역에 복제본을 생성해놓아 시스템을 유지시켜줍니다.

<br>

![image6](/Img/RDS/RDS6.png)

- VPC : DB 인스턴스가 위치할 네트워크를 선택해줍니다.
- 서브넷 그룹 : DB 인스턴스가 어떤 서브넷과 IP 범위를 사용할 수 있는지를 정의합니다.
- 퍼블릭 액세스 : DB를 외부에서 접근할 수 있게 하는 옵션입니다.
- VPC 보안 그룹 : 방화벽 설정인 보안 그룹입니다.
- 접속 포트 : DB 접속 포트 번호입니다.

**추가구성**

초기 데이터베이스 이름 : 데이터베이스 이름을 필수로 지정해줘야 합니다.

백업 : 자동 백업 옵션입니다. 

백업 보존 기간: 백업 데이터 유지 기간입니다.

모니터링 : Enhanced 모니터링 옵션입니다.

유지관리 : DB 버전 업데이트 또는 보안 패치나 수정된 버전을 자동으로 업데이트합니다.

설정이 완료되었으면 데이터베이스를 생성하면 됩니다.

<br><br>

# RDS DB 스냅샷

RDS는 스냅샷을 생성할 수 있습니다. RDS DB 스냅샷은 DB의 전체 내용 중 특정 시점을 파일로 저장한 형태입니다. 스냅샷을 이용하여 특정 시점으로 RDS를 복원할 수 있습니다.

그러나 DB 스냅샷은 기존의 DB 인스턴스를 삭제하지 않고 새롭게 생성하여 복원을 진행합니다.

<br>

### 스냅샷 생성

![image7](/Img/RDS/RDS7.png)

RDS DB 인스턴스를 선택하여 DB 스냅샷 생성합니다.

<br>

![image8](/Img/RDS/RDS8.png)

스냅샷할 DB를 선택하고 이름을 지정해줍니다.

<br>

![image9](/Img/RDS/RDS9.png)

DB 스냅샷 생성이 완료됩니다.

<br>

### 스냅샷을 이용한 RDS DB 인스턴스 생성

![image10](/Img/RDS/RDS10.png)

생성한 스냅샷을 선택하고 스냅샷 복원을 선택해줍니다.

스냅샷으로 RDS DB 인스턴스 생성하기 전에 설정이 필요합니다.

이 단계에서 성능이 더 좋은 인스턴스 클래스로 변경도 가능합니다.

설정을 하는 단계는 RDS DB 인스턴스를 생성할때와 유사하므로 생략하겠습니다.

이렇게 RDS DB 인스턴스를 스냅샷을 이용해 생성할 수 있습니다.

<br><br>

# RDS DB scalability(확장성)

RDS DB도 EC2와 마찬가지로 scalability가 가능합니다.

DB의 사용량이 많아져 부하가 늘어난다면 scalability(확장)을 고려해 봐야하는데요 이때 사용할 수 있는 방법은 Scale up, Scale Out 두가지가 있습니다.

<br>

### Scale up

Scale up은 DB의 성능을 업그레이드 시켜주는 방법입니다.

![image11](/Img/RDS/RDS11.png)

RDS DB 인스턴스의 성능을 확장하는 방법은 DB 인스턴스 목록에서 성능을 높일 DB 인스턴스를 선택한뒤 위의 수정버튼을 통해 DB 인스턴스의 설정을 변경하여 성능을 확장시킵니다.

성능과 관련된 인스턴스 클래스, 스토리지 와 같은 설정을 변경시켜 줍니다.

마지막으로 설정 변경을 즉시 적용할지 예약 적용할지 선택해 주면 됩니다.

Scale up은 간단하면서 강력한 성능 업그레이드가 가능합니다. 하지만 Scale up은 어느 순간 더이상 성능을 업그레이드 할 수 없는 한계에 도달하게 됩니다.

그래서 한계를 극복하기 위해 Scale out을 알아보겠습니다.

<br><br>

# RDS DB 인스턴스 Read Replica 생성

DB는 크게 읽기와 쓰기의 동작으로 구분됩니다. 그중에서도 읽기작업이 많은 비중을 차지하게 됩니다. 그래서 DB를 Scale out하기 위해서 Read Replica라는 DB 인스턴스의 읽기 복제본을 만들어 성능을 향상 시키는 방법을 사용하빈다.

서비스에서 읽기 읽기 위주의 작업이 많을 경우 Read Replica를 여러 개 만들어 부하를 분산할 수 있습니다. 쓰기 작업은 마스터 DB 인스턴스에서 하고 읽기 작업은 Read Replica인 슬레이브 DB 인스턴스에서 실시한다면 마스터 DB 인스턴스의 부하를 줄일 수 있습니다.

<br>

![image12](/Img/RDS/RDS12.png)

RDS DB 인스턴스를 선택하고 읽기 전용 복제본 생성을 클릭합니다.

Read Replica 인스턴스의 설정을 지정해줍니다.

설정하는 단계는 RDS DB 인스턴스를 생성할때와 유사하므로 생략하겠습니다.

Read Replica 인스턴스를 생성하면 마스터 DB 인스턴스도 설정해 줘야 하기 때문에 기존의 DB의 상태도 변경됩니다.

Replica 인스턴스가 생성이 완료되면 DB의 역할이 나눠집니다.

<br>

![image13](/Img/RDS/RDS13.png)

Read Replica 인스턴스가 완전히 생성된 후 세부 내용을 살펴보면 현재 복제된 DB 인스턴스를 살펴볼 수 있습니다.

<br>

![image14](/Img/RDS/RDS14.png)

Read Replica 인스턴스는 마스터 DB와 동일한 데이터를 사용할 수 있지만 읽기 전용이어서 SELECT 문만 사용 가능합니다.

<br><br>

# RDS DB 사용하기(MySQL)

MySQL Workbench Windows 버전을 기준으로 설명하겠습니다.

![image15](/Img/RDS/RDS15.png)
MySQL Connections 옆의 + 버튼을 클릭해줍니다.

<br><br>

![image16](/Img/RDS/RDS16.png)

- Connection Name : 연결 이름을 정해줍니다.
- Hostname : RDS에 접속하기 위해서는 RDS의 엔드포인트 주소를 입력해줘야 합니다. RDS의 엔트포인트 주소를 입력해줍니다.
- Username : RDS DB 인스턴스르 생성할 때 설정했던 Username을 입력합니다. DB 인스턴스의 구성 부분에 들어가보면 Username을 확인할 수 있습니다.
- Password : Password를 입력해줍니다.

<br><br>

![image17](/Img/RDS/RDS17.png)

OK 버튼을 통해 생성을 완료한뒤 MySQL Workbench에서 RDS DB 인스턴스에 접속을 시도해봅니다.

<br>

![image18](/Img/RDS/RDS18.png)

RDS DB 인스턴스가 생성되었는데 엔드포인트 주소로 접속이 안되는 오류가 발생합니다.

이러한 오류가 발생하는 이유는 Security Group에서 접속이 차단되어 있기 때문입니다.

DB 인스턴스를 생성할 때 Security Group을 기본값인 default(VPC)로 설정했습니다. default(VPC)는 모든 트래픽에 대해 Inbound가 열려있지만 접속 가능한 IP대역은 default 자기 자신으로 설정되어 있습니다. 즉 같은 default(VPC) Security Group 설정 안에서만 접속이 허용되므로 외부에서는 접속할 수 없습니다. 

외부에서 엔드포인트 주소에 접속하려면 RDS DB 인스턴스(MySQL)전용 Security Group을 생성하고 MySQL 포트(3306)을 열어줘야 합니다.

<br><br>

RDS DB 인스턴스용 Security Group을 생성해보겠습니다.

![image19](/Img/RDS/RDS19.png)

먼저 EC2의 Security Group(보안 그룹)으로 화면을 이동해 Security Group을 생성합니다.

<br><br>

![image20](/Img/RDS/RDS20.png)

- 보안 그룹 이름 : Security Group의 이름입니다.

외부에서 들어오는 트래픽인 인바운드(Inbound)를 추가해줍니다.

- 유형 : MySQL 포트를 열어줘야 하므로 MySQL을 선택합니다.
- 소스 유형 : 접속가능한 IP 또는 IP 대역입니다. 여기서는 Anywhere을 선택하겠습니다.

설정이 완료되면 생성해줍니다.

<br>

이제 다시 RDS DB 인스턴스 목록으로 이동합니다. 

![image21](/Img/RDS/RDS21.png)

DB 인스턴스를 선택한뒤 수정을 눌러줍니다.

<br>

![image22](/Img/RDS/RDS22.png)

DB 인스턴스에 보안 그룹(Security Group)을 방금 생성한 보안 그룹으로 바꿔줍니다.

<br>

![image23](/Img/RDS/RDS23.png)

이제 다시 MySQL Workbench로 돌아와서 접속해보면 접속이 되는걸 확인할 수 있습니다.

<br>

![image24](/Img/RDS/RDS24.png)