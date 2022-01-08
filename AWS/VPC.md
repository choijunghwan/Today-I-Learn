# VPC란?

VPC(Virtual Private Cloud)는 가상 네트워크를 제공합니다.

사용자의 상황에 맞게 VPC를 생성하여 여러 가지 형태의 네트워크를 구성할 수 있습니다.

만일 VPC 없이 인스턴스를 생성한다면 아래와 같이 인스턴스끼리의 구분없는 연결로 인해 시스템 복잡도가 증가할 것이다. 또한 인터넷을 통해 전달되는 트래픽의 전송이 굉장히 비효율적이게 될 것이다.

![image1](/Img/VPC/VPC1.png)

<br>

VPC를 적용하면 아래와 같이 인스턴스가 VPC에 속함으로써 네트워크를 구분할 수 있고 VPC 별로

필요한 설정을 통해 인스턴스에 네트워크 설정을 적용할 수 있다. AWS는 VPC의 중요성을 강조하여 2019년부터 모든 서비스에 VPC를 적용하도록 강제하였다. 

![image2](/Img/VPC/VPC2.png)

<br>

VPC 안에서 EC2 인스턴스, RDS DB 인스턴스, ElasticCache 캐시 클러스터 등을 생성하고 사용할 수 있습니다. 기본적으로 제공하는 VPC 이외에도 용도에 따라 VPC를 추가할 수 있고, VPC 안에는 서브넷을 여러 개 추가할 수 있습니다. 서브넷을 여러 개로 나누면 네트워크를 격리할 수 있고, 이 서브넷 간에 접근제어(ACL)를 설정할 수 있습니다.

예를 들어 인터넷에서 접근해야 하는 웹 서버는 공개 서브넷(Public Subnet)에 만들고, 외부에서 접근이 필요 없는 데이터베이스 서버는 사설 서브넷(Private Subnet)에 만드는 식으로 구성할 수 있습니다. 

VPC는 다른 AWS 계정의 VPC와도 연결할 수 있고, VPN(Virtual Private Network)을 이용하여 회사 네트워크와 연결할 수 있습니다.

![image3](/Img/VPC/VPC3.png)

<br>

추가로 VPC는 리전별로 생성하고, 서브넷은 가용 영역(Availability Zone)별로 생성합니다.

VPC를 만들 때 VPC의 IPv4 주소의  범위를 CIDR(Classless Inter-Domain Routing) 블록 형태를 지정해야 합니다.

![image4](/Img/VPC/VPC4.png)

<br><br>

# VPC 구성요소

### VPC

VPC는 독립된 하나의 네트워크를 구성하기 위한 가장 큰 단위이다. 각 Region에 종속되며 [RFC1918](https://datatracker.ietf.org/doc/html/rfc4632)이라는 사설 IP 대역에 맞추어 설계해야 한다.

<br>

1. 10.0.0.0 ~ 10.255.255.255 (10/8 prefix)
2. 172.16.0.0 ~ 172.31.255.255 (172.16/12 prefix)
3. 192.168.0.0 ~ 192.168.255.255 (192.168/16 prefix)

<br>

> **사설 IP란?**
인터넷을 위해 사용하는 것이 아닌 우리끼리 사용하는 아이피주소 대역.

<br>

각 대역폭마다 고정되어 있는 prefix가 다릅니다. 

1번 `10/8 prefix`를 살펴보면 IPv4주소인 xxx.xxx.xxx.xxx에서 xxx은 0~255 사이의 숫자로 2진수로 표현하면 총 8자리(8bit)로 표현할 수 있습니다.  10을 2진수로 변경하면 `00001010` 이 되기 때문에

`10/8 prefix` 는 8bit까지 00001010을 고정시킨다는 뜻이 됩니다.

2번 `172.16/12 prefix` 의 경우 **172.16 → 01011000.0001xxxx** 이므로 12자리까지 고정되어 있어 `12 prefix`가 됩니다.

<br>

VPC에서 한번 설정된 IP 대역은 수정할 수 없으며 각각의 VPC는 독립적이기 때문에 서로 통신할 수 없다. 만일 통신을 원한다면 VPC 피어링 서비스를 통해 VPC 간에 트래픽을 라우팅할 수 있도록 설정할 수 있습니다.

![image5](/Img/VPC/VPC5.png)

<br><br>

### Subnet(서브넷)

서브넷은 VPC의 IP 주소를 나누어 리소스가 배치되는 물리적인 주소 범위를 뜻한다. 서브넷은 하나의 AZ(Availability Zone)에 하나의 서브넷이 연결되기 때문에 region의 AZ 수를 미리 확인하고 설정해야 합니다.

서브넷은 다시 Public Subnet과 Private Subnet으로 나뉠 수 있다. 인터넷과 연결되어 있는 서브넷을 public subnet이라고 하고 인터넷과 연결되어있지 않은 서브넷을 private subnet이라고 한다. 이처럼 인터넷 연결 여부로 subnet을 구분하는 이유는 보안을 강화하기 위함이다.

private subnet은 외부에 노출이 되어 있지 않기 때문에 접근할 수 없다. 즉 인터넷과 연결되어 외부에 노출되어 있는 면적을 최소화함으로써 네트워크 망에 함부로 접근하는 것을 막기 위함이다.

![image6](/Img/VPC/VPC6.png)

<br><br>

### Router(라우터)

라우터는 VPC 안에서 발생한 네트워크 요청을 처리하기 위해 어디로 트래픽을 전송해야 하는지 알려주는 표지판 역할을 수행한다. 각각의 서브넷은 네트워크 트래픽 전달 규칙에 해당하는 라우팅 테이블을 가지고 있으며 요청이 발생하면 가장 먼저 라우터로 트래픽을 전송한다. 일반적으로 VPC 내부 네트워크에 해당하는 주소는 local로 향하도록 한다.

![image7](/Img/VPC/VPC7.png)

<br><br>

### Internet Gateway(IGW, 인터넷 게이트웨이)

인터넷 게이트웨이는 VPC 리소스와 인터넷 간 통신을 활성화하기 위해 VPC에 연결하는 게이트웨이이다. 앞서 설명했듯 pubblic subnet만 외부와 통신해야 하므로 public subnet의 라우팅 테이블에만 인터넷 게이트웨이로 향하는 규칙을 포함한다. 아래 그림과 같이 라우팅 규칙을 설정하면 네트워크 요청 발생 시 목적지의 IP 주소가 10.0.0.0/16에 해당하는지 확인한다. 해당하지 않는 모든 트래픽은 인터넷 게이트웨이를 통해 외부로 전송되어 목적지를 찾는다.

![image8](/Img/VPC/VPC8.png)

<br><br>

### NAT Gateway(NAT 게이트웨이)

public subnet만 인터넷 게이트웨이를 통해 외부와 트래픽을 주고받을 수 있다면 private subnet의 트래픽은 무조건 VPC 안에서만 처리된다는 뜻일까? 그렇지 않다.

private subnet 역시 마찬가지로 인터넷과 통신할 수 있다. 하지만 private subnet에서 직접하는 것은 불가능하므로 트래픽을 public subnet에 속한 인스턴스에 전송해서 인터넷과 통신을 해야한다. 

**NAT 게이트웨이**가 이 역할을 수행한다. private subnet에서 발생하는 네트워크 요청이 VPC 내부의 주소를 목적지로 하는 것이 아니라면 public subnet에 존재하는 NAT로 트래픽을 전송한다. 트래픽을 받은 NAT는 public subnet의 라우팅 규칙에 따라 처리함으로써 private subnet이 인터넷과 통신할 수 있도록 한다.

![image9](/Img/VPC/VPC9.png)

<br><br>

### VPC Endpoint(Private Link, VPC 엔드포인트)

**VPC 엔드포인트**는 **인터넷 게이트웨이**나 **NAT 게이트웨이**와 같은 다른 게이트웨이 없이 AWS 서비스와 연결하여 통신할 수 있는 private connection을 제공하는 서비스이다. Private link를 통해 AWS 서비스와 연결함으로써 데이터를 인터넷에 노출하지 않고 바로 접근할 수 있다. 또한 연결하는 서비스의 IP 주소를 바꾸는 등 네트워크 정보의 변경 등의 작업에서 불필요하게 발생하는 비용을 줄일 수 있다.

![image10](/Img/VPC/VPC10.png)

<br><br>

# VPC 및 서브넷 다이어그램

다음 그림은 여러 가용 영역의 서브넷으로 구성된 VPC를 보여줍니다.

![image11](/Img/VPC/VPC11.png)

**참조**

- VPC의 하나의 **Region**에는 3개의 서브넷이 존재합니다.
- 서브넷 1은 퍼블릭 서브넷, 서브넷 2는 프라이빗 서브넷, 서브넷 3는 VPN 전용 서브넷입니다.
- IPv6 CIDR 블록 하나는 VPC와 연결되어 있고, IPv6 CIDR 블록 하나는 서브넷 1과 연결되어 있습니다.
- 인터넷 게이트웨이(Internet gateway)를 통해 인터넷에서 통신할 수 있으며, 가상 프라이빗 네트워크(VPN) 연결을 통해서 기업 네트워크와 통신할 수 있습니다.

<br>

참고자료  
[https://jbhs7014.tistory.com/164](https://jbhs7014.tistory.com/164)
https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/VPC_Subnets.html#vpc-subnet-basics
