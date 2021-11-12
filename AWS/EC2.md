# AWS 기본 개념 - 지역, 가용구역, 에지 로케이션

AWS의 인프라가 위치하고 있는 공간에 대한 구분이 지역(Region)과 가용구역(avaliability zone)이 있다.

<br>

### **지역(Region)**

어디에 위치해 있는 아마존 컴퓨터를 쓸건가를 결정하는것이다.

전기는 굉장히 빠른속도로 이동하지만 물리적인 거리가 멀다면 그만큼의 시간이 좀 더 걸리기 때문에 어느 지역으로 설정하느냐에 따라 **네트워크 속도와 관계**가 있다.

즉 나와 가까운 지역을 선택하는 것이 중요하다. 하지만 좀 더 중요한건 나의 서비스를 사용할 사용자가 어느 지역에 많이 위치해 있는가가 중요하다. 실제 사용자와 밀첩해야 사용자가 느끼는 네트워크 속도가 빠르기 때문이다.

그렇다면 위치가 가까운 곳을 어떻게 찾을 수 있을까??

[https://www.cloudping.info/](https://www.cloudping.info/)

위의 사이트를 들어가보면 네트워크의 Latency 속도가 얼마나 걸리는지 대략적인 점을 알 수 있다. 위 사이트처럼 속도를 측정할 수 있는 여러 사이트를 참고하여 결정하면 좋다.

<br>

### **가용영역(Avaliability Zone)**

가용영역은 흔히 말하는 데이터센터(IDC)입니다. 이 가용 영역이 위치한 데이터센터는 같은 지역 혹은 도시라 하더라도 멀리 떨어져 있습니다.

가용 영역이 이렇게 물리적으로 떨어져 있는 이유는, 하나의 가용 영역이 각종 재해, 정전, 테러, 화재 등 다양한 이유로 작동불능이 되더라도 다른 가용 영역에서 서비스를 재개할 수 있게 위해서입니다.

실제로 AWS에서는 EC2 가상 서버를 한 지역 안에서도 여러 가용 영역을 만들어서 사용할 것을 권장하고 있습니다. 로드밸런서인 ELB(Elastic Load Balancing)는 같은 지역 안에 여러 가용 영역에 걸쳐 있는 EC2에 트래픽을 분배해줄 수 있습니다. 이렇게 하면 가용 영역 하나가 작동불능이 되더라도 나머지 가용 영역은 정상 작동하므로 무중단 서비스를 제공할 수 있습니다.

<br>

### 에지 로케이션(Edge Location)

에지 로케이션은 AWS의 CDN 서비스인 CloudFront를 위한 캐시 서버들을 뜻합니다.

먼저 **CDN** 서비스는 **Content Delivery Network**의 약자로서, 콘텐츠(HTML, 이미지, 동영상, 기타 파일)를 사용자들이 빠르게 받을 수 있도록 전 세계 곳곳에 위치한 캐시 서버에 복제해주는 서비스입니다.

멀리 떨어진 서버보다 가까운 서버에 접속하는 것이 전송 속도가 훨씬 빠르기 때문에 CDN 서비스는 전 세계 주요 도시에 캐시 서버를 구축해 놓습니다.

이 CDN 캐시 서버는 인터넷 트래픽을 효과적으로 처리할 수 있는 지역에 POP(Point-of-Presence)을 구축합니다. CDN 서비스와 사용자가 직접 만나는 곳이라고 하여 에지(Edge)라고 부르는 것입니다.

<br><br>

# AWS EC2

### EC2란?
EC2(Elastic Compute Cloud)는 AWS에서 가장 기본적이면서 널리 쓰이는 인프라로 인터넷에 연결된 가상서버를 제공해줍니다.

EC2를 사용해야 하는 이유는 **효율성**과 **비용 절감**에 있습니다. EC2는 클릭 몇 번으로 서버를 생성하고, 사용한 만큼만 요금을 지불하면 되기때문에 비용도 절약할 수 있습니다.

EC2 인스턴스는 우리가 흔히 사용하는 컴퓨터와 같습니다. Linux나 Windows가 설치되어 있고 가상 서버이기 때문에 모니터에는 직접 연결할 수 없고 터미널 또는 원격 데스크톱 연결로 접속해야 합니다.

**EC2 상태**

- 시작(Start) : EC2 인스턴스를 시작합니다. 운영체제가 부팅되고 사용할 수 있는 상태입니다. 시작하는 순간부터 사용요금이 과금되며 1분을 사용하더라도 1시간 요금으로 책정됩니다.
- 정지(Stop) : EC2 인스턴스를 정지합니다. 운영체제를 종료하여 시스템이 정지한 상태이며 사용 요금이 과금되지 않습니다.
- 삭제(Terminate) : EC2 인스턴스를 삭제합니다.
- 재부팅(Reboot) : EC2 인스턴스를 재부팅합니다. 운영체제를 종료한 뒤 다시 시작합니다.
- Root 장치: 운영체제가 설치되는 스토리지입니다. Root 장치로 EBS와 인스턴스 스토리지를 사용할 수 있습니다.
- Kernel ID: EC2 인스턴스가 사용하는 Linux 커널입니다. Linux 반가상화는 외부에서 Linux 커널을 지정해주어야 합니다. 그리고 AWS에서 제공하는 다양한 Linux 커널을 선택할 수 있습니다.

### EC2 인스턴스 타입

인스턴스 유형은 t2.micro처럼 앞에 인스턴스 패밀리인 `t`에 세대(Generation)를 뜻하는 숫자`2`가 붙고 `.(점)` 전체적인 사양을 뜻하는 단어가 붙습니다.

먼저 .(점) 뒤에 붙는 사양은 아래와 같은 순서로 되어있습니다. 단어를 보면 쉽게 유추가 가능합니다.

`xlarge > large > medium > small > micro > nano`

인스턴스 패밀리는 다음과 같습니다.

- General purpose : 범용적인 유형으로 vCPU, 메모리, 네트워크, 저장 공간 등 평균적인 사양으로 제공됩니다.
- Compute optimized : 컴퓨팅 최적화 유형입니다. 메모리 대비 vCPU 비율이 높습니다.
- GPU Instances : 고성능의 GPU가 장착되어 있는 유형입니다.
- Memory optimized : 다른 인스턴스 패밀리에 비해 메모리 용량이 큽니다.
- Storage optimized : 다른 인스턴스 패밀리보다 스토리지 용량이 훨씬 크거나 초고속 I/O를 제공합니다.

좀 더 자세한 정보를 알고 싶다면 [여기](https://aws.amazon.com/ko/ec2/instance-types/)를 접속하시면 알 수 있습니다.

<br>

### EC2 가격정책

- 온디맨드
    - 실행하는 인스턴스에 따라 사용시간 만큼 비용을 지불합니다.
    - 장기적인 약정없이 유연하게 EC2를 사용하기에 적합합니다.
- 스팟 인스턴스
    - 스팟 인스턴스를 사용하면 AWS 클라우드에서 미사용 EC2 용량을 확보할 수 있습니다.
    - 미사용중인 EC2를 사용하는것이기 때문에 온디맨드 요금과 비교하여 매우 저렴합니다.
    - 주로 빅 데이터, CI/CD, 고성능 컴퓨터 등에 적합합니다.
- 예약 인스턴스
    - 일정 기간(1년 또는 3년) 예약금을 선불로 내고 예약하면 시간당 요금이 대폭 할인됩니다.
    - 장기적인 사용 계획이 있을때 유용합니다.

좀 더 자세한 요금정책은 [여기](https://aws.amazon.com/ko/ec2/pricing/)를 접속하시면 알아 볼 수 있습니다.

<br>

### EC2 인스턴스 세부 설정

![image1](/Img/EC2/EC2_1.png)

- 인스턴스 갯수 : 생성할 인스턴스 개수입니다.
- 구매 옵션 : 스팟 인스턴스로 구매 옵션입니다.
- 네트워크 : VPC 네트워크를 선택하는 옵션입니다. 기본값 그대로 사용합니다.
- 서브넷 : 가용 영역을 선택하는 옵션입니다. 기본값 그대로 사용합니다.
- 퍼블릭 IP : 공인 IP를 할당하는 옵션입니다. 기본값 그대로 사용합니다.
- 종료 방식 : EC2 인스턴스안에 설치된 운영체제를 종료했을 때 행동입니다. 중지(Stop)은 그냥 종료만 하고, 종료(Terminate)는 종료 후 인스턴스를 삭제합니다. 기본값 그대로 사용합니다.
- 종료 방지 기능 활성화 : 실수로 삭제하는 것을 방지하는 옵션입니다.
- 모니터링 : CloudWatch 세부 모니터링 사용 옵션입니다. 이 옵션을 사용하면 추가요금이 발생합니다. 기본값 그대로 사용합니다.
- 테넌시 : 가상 서버 실행 방식을 설정하는 옵션입니다. 공유 인스턴스(Shared tenancy)와 전용 인스턴스(Dedicated tenancy)를 선택할 수 있습니다. 기본값 그대로 사용합니다.

<br>

### EC2 인스턴스 스토리지 설정

![image2](/Img/EC2/EC2_2.png)

인스턴스의 스토리지를 설정합니다. 루트(Root) 장치는 꼭 있어야 합니다.

- 볼륨 유형 : 루트(Root)장치인지 추가 장치인지 설정하는 옵션입니다. 기본적으로 루트장치는 EBS만 사용할 수 있고, 추가 장치는 EBS와 인스턴스 스토리지(Instance Store)를 사용할 수 있습니다.
- 디바이스 : 장치 이름입니다.
- 볼륨 유형 : 스토리지 볼륨 유형입니다. Magenetic, General Purpose SSD(gp3, gp2), Provisioned IOPS SSD(io1, io2)를 선택할 수 있습니다.
    - Magnetic : 하드디스크를 사용하는 스토리지입니다.
    - General Purpose : SSD를 사용하는 스토리지입니다. 가격과 성능을 균형있게 제공합니다.
    - Provisioned IOPS : SSD를 사용하는 스토리지입니다. 지연시간이 짧거나 처리량이 많은 워크로드에 적합한 고성능을 제공합니다.
- IOPS : IOPS 값을 설정할 수 있습니다.
- 종료시 삭제 : EC2 인스턴스가 실행되고 있을 때 스토리지가 실수로 삭제되는 것을 방지합니다.

<br>

> **인스턴스 스토리지**  
EBS와 달리 인스턴스 스토리지는 EC2 인스턴스를 정지하면 데이터가 사라집니다. 대신 EBS보다 속도가 빠릅니다.

좀 더 자세한 내용은 [여기](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ebs-volume-types.html)를 통해서 확인할 수 있습니다.

<br>

### EC2 보안그룹

![image3](/Img/EC2/EC2_3.png)

인스턴스에 대한 트래픽을 제어하는 방화벽을 설정합니다.

인스턴스는 가상머신이므로 터미널을 통해 접속해야 하기 때문에, SSH 22번 포트만 기본값으로 설정되어 있습니다.(Linux 기준 SSH가 설정되어 있습니다. Windows 기준이라면 RDP가 설정되어 있을것입니다)

웹 서버로 활용되려면 `HTTP`, `HTTPS`를 추가로 설정해주면 됩니다.

<br><br>

# AWS EC2 Scalability

EC2의 가장 중요한 특징은 가상화, 종량제 입니다.

가상화를 얘기하기 위해선 가상머신을 이해할 필요가 있습니다.

가상머신은 쉽게 말해 소프트웨어로 만든 가상의 컴퓨터입니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/10cfb27b-1b64-4cb2-80c8-3aada4b4f0e5/Untitled.png)

위의 그림처럼 컴퓨터위에 OS, Linux 같은 운영체제가 있고 그 위에 가상머신이 있습니다. 가상머신은 실제로 자기가 CPU와 메모리를 가지고 있는 컴퓨터 같이 작동합니다.

이러한 가상머신은 실제가 아닌 가상의 소프트웨어로 만든것이기 때문에 성능을 조절할 수 있는 장점이 있습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/669bee81-3902-4c20-8b6a-08c50c6f4fbb/Untitled.png)

만약 우리에게 필요한 서버 컴퓨터가 하루에 2~3명 방문하는 서비스를 운영한다면 굳이 고성능의 컴퓨터가 아님 저렴한컴퓨터여도 충분합니다. 반면에 하루에 수천~수만명이 방문하는 서비스라면 강력한 성능의 컴퓨터가 필요합니다.

EC2의 가상화는 각 상황에 맞는 가상머신을 만들어 줍니다.

또한 만약 우리의 서비스가 2~3명 방문하는 서비스였는데 서비스가 점차 성능하여 수천~수만명이 방문하는 서비스가 되었다면 그에 맞게 성능을 향상시켜줄 수 있습니다. 

이렇게 상황에 맞춰 변화하고 성능을 늘리는걸 `scalability`(확장성)이라고 하는데 scalability는 `Scale up`, `Scale Out` 두가지 방법이 있습니다.

# Scale Up

Scale Up이란 하나의 컴퓨터를 더 좋은 컴퓨터로 교체하는 것을 의미합니다.

서비스가 성장함에 따라 수요에 맞춰 저렴한 컴퓨터 → 강력한 컴퓨터로 변경하는 것입니다.

그럼 AWS EC2를 어떻게 Scale Up 할 수 있을까요??

### AMI

그전에 먼저 AMI에 대해 알아보겠습니다.

AMI(Amazon Machine Images)는 EC2 인스턴스를 생성하기 위한 기본 파일입니다. 

AWS에서는 빈 EC2 인스턴스에 직접 OS를 설치할 수는 없습니다. 그렇기 때문에 미리 OS가 설치된 AMI를 이용하여 EC2 인스턴스를 생성하게 됩니다. 이 AMI는 단순히 OS만 설치되는 것이 아니라 각종 서버 애플리케이션, 데이터베이스, 방화벽 등의 네트워크 솔루션 등을 함께 설치될 수 있습니다.

쉽게 생각하면 내가 기존의 설치 해놓은 설정이나, 솔루션등을 저장해놓았다가 나중에 AMI를 통해 한번에 설치 할 수 있는것입니다. Spring boot로 생각하면 starter가 AMI이어서 starter를 설치하면 기본 옵션들이 한번에 설치되는 것과 같습니다.

AMI는 설치 및 설정이 완료된 EC2 인스턴스를 신속하게 생성해야 할 때, Auto Scaling 등으로 자동화 할 때, EC2 인스턴스를 다른 리전으로 이전해야 할 때, 상용 솔루션을 사용하고자 할 때 사용합니다.

### EC2 인스턴스로 AMI 생성하기

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/833dba9f-9c55-4349-8432-1967ccd65426/Untitled.png)

EC2 인스턴스목록에서 AMI를 생성할 EC2 인스턴스를 오른쪽 버튼을 클릭하면 이미지 및 템플릿 → 이미지 생성 메뉴가 나옵니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c9b0494a-f15f-45a2-b4a7-bc4b0d749a0e/Untitled.png)

- 재부팅 안함 : AMI를 생성할 때 EC2 인스턴스를 재부팅하지 않고 생성하는 옵션입니다. EC2 인스턴스를 재부팅하지 않고 AMI를 생성하면 파일시스템의 무결성을 보장하지 않습니다.
- 인스턴스 볼륨 : 기본 장착될 EBS 볼륨 설정입니다.

> **파일시스템 무결성**은 파일시스템의 성능을 높이기 위해 파일을 디스크에 바로 쓰지 않고 메모리에 모아놓았다가 씁니다. 따라서 쓰기 동작이 완료되지 않은 상태가 생길 수 있는데 이러면 파일시스템의 무결성이 깨진 상태가 됩니다.
> 

### EC2 인스턴스 Scale Up

이렇게 생성한 AMI를 이용하여 좀 더 좋은 성능의 새로운 EC2 인스턴스를 생성해줍니다. 

새로운 인스턴스를 생성하면 새로운 가상서버가 생성되지만 우리가 원하는 것은 기존의 서버의 성능 업그레이드를 원하는 것입니다. 그러기 위해서는 기존의 서버가 갖고있는 Elastic IP(퍼블릭 IP, 고정 아이피)를 새로 생성한 인스턴스에 재설정해주어서 사용자들이 서버에 접속하여도 Scale Up한 새로운 서버로 접속할 수 있게 해줍니다.

# Scale out

점차 요구하는 성능이 높아짐에 따라 Scale up을 진행하다보면 단일 컴퓨터가 할 수 있는 최대 성능까지 도달하게 되면 더이상의 성능 업그레이드가 불가능하게 됩니다.

이러한 한계를 뛰어넘기 위한 방법이 Scale out 입니다. 

Scale out은 단일 컴퓨터가 아닌 여러컴퓨터 즉 서버를 여러대로 증가시키므로써 처리 능력을 향상시키는 것입니다.

### Scale out의 흐름

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8abdfab4-71d3-405f-933e-7c3a4994604a/Untitled.png)

만약 우리의 웹서버가 가장 간단하게 구성되어 있다면 위와 같이 Web Server, Middle ware, Database가 모두 하나의 컴퓨터 안에 존재해 있을 것입니다.

여기서 Middle ware는 WAS(Web application Server)라고 생각하시면 됩니다.

그런데 점점 이용하는 사용자가 증가함에 따라 Scale out을 해야하는 상황에 놓였습니다.

Scale out은 단일 컴퓨터를 여러대로 증가시키는 것이라고 했는데요 현재는 하나의 컴퓨터에 웹 서버의 모든 과정이 들어있기 때문에 먼저 위의 과정을 각각의 컴퓨터로 분리합니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6ceb2ad8-1d43-46c2-beb2-6629f32dbe94/Untitled.png)

이렇게 `Web Server`, `Middle ware`, `Database` 세가지를 각각의 컴퓨터로 분리하면 Scalability(확장성)한 시스템이 준비가 된것입니다. 

만약 `Database`의 접근에 부하가 많이 발생한다면 `Database`를 늘려주고, `Middle ware`에 부하가 많이 발생한다면 `Middle ware`를 늘려주면 됩니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b2fb7342-a2fa-4d0d-958e-b7fe35a04af5/Untitled.png)

이제 이렇게 각각 `Middle ware`와 `Database`는 Scale out 하였는데 아직 늘리지 못한것이 있습니다.

바로 `Web Server` 입니다. 그런데 `Web Server`에는 현재 하나의 IP가 할당되어 있습니다.

`Web Server`를 늘려주기 위해서는 **IP**를 추가로 할당해주는 방법이 있습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9793789b-07d3-4923-a56a-5cc1ee78523f/Untitled.png)

IP를 추가로 할당받으면 DNS의 설정을 통해 사용자들을 각각의 IP에 접속하게 하여 Web Server를 부하를 줄여줄 수 있습니다.

그런데 IP를 추가로 할당받는 대신에 `ELB(Elastic Load Balancing)`을 통해서 부하를 분산시켜주는 방법이 있습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d3c02e4f-dc2c-489b-bce8-a369619819fa/Untitled.png)

위의 그림처럼 `Load Balancer`를 통해 부하를 분산시켜 줍니다. 이제는 `Load Balancer` 하나의 IP를 가지고 있고 사용자의 접속을 자동으로 `Web Server`에 분산시켜줍니다. 

만약 **computer 9** 서버가 중단되더라도 `Load Balancer`는 이를 감지하고 트래픽을 나머지 **computer8, computer1** 정상 서버에 보냅니다.

이렇게 좋은 서비스를 AWS에서는 몇 번의 클릭만으로도 Load Balancer인 ELB를 생성할 수 있습니다.

### 주의 사항

위의 Load Balancer에는 한가지 문제가 있습니다. 좀 더 간단한 아래 그림의 상황을 보며 얘기해보겠습니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e3b54126-14f9-4d7f-afe5-50880a92ee5c/Untitled.png)

만약 우리가 ELB를 통해 Computer1, Computer2로 Scale out을 했습니다.

만약 A라는 사용자가 접속해서 Computer2에 있는 Database에 데이터를 저장하게 되었습니다.

그리고 다시 A라는 사용자가 접속했는데 Computer1으로 접속이 되게 되면 방금 전에 저장한 데이터를 찾을 수 없는 데이터 불일치의 문제가 발생하게 됩니다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8f3c6cfc-01f0-4118-a8d2-d412c4406ef0/Untitled.png)

즉 Scale out을 할때는 Database를 따로 분리하여서 진행을 해야 합니다. 

만약 위의 방법처럼 하고 싶다면 Database에 쓰기 작업을 하지 않는 경우에만 가능합니다.

<br>

참고자료
* https://opentutorials.org/course/2717/11268