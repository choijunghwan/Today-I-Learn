# CDN(CloudFront)

CloudFront는 전 세계에 파일을 빠른 속도로 배포하는 CDN 서비스입니다.

CloudFront를 사용해야 하는 이유는 전송 속도 향상과 비용 절감에 있습니다.

전 세계를 대상으로 하는 서비스를 준비할 때, 세계 곳곳에 서버를 구축하는 것은 현실적으로 어렵습니다. 또한, AWS를 사용한다 하더라도 모든 리전에 EC2 인스턴스나 S3 버킷을 생성하는 것은 비효율적이며 비용도 많이 듭니다.

그래서 CloudFront인 캐시 서버를 이용해서 각 사용자들의 가까운 에지 로케이션 파일이나 데이터를 캐시해 속도를 높여줍니다.

![image1](/Img/CDN/CDN1.png)

EC2 인스턴스나 S3 버킷은 동작되는 장소가 리전에 한정적입니다. 그래서 먼 거리의 지역의 경우 전송 속도가 느린데 이를 CloudFront를 이용하여 EC2, ELB,S3 버킷의 내용을 에지 로케이션에 캐시하면 빠른 성능을 낼 수 있습니다.

![image2](/Img/CDN/CDN2.png)

사용자가 맨 처음 CloudFront 에지 로케이션에 접속했을 때, 원하는 파일이 에지 로케이션에 있으면 바로 파일을 받지만 파일이 없으면 에지 로케이션이 원본 파일이 있는 오리진에 접속하여 파일을 가져온뒤 사용자에게 전달합니다.

![image3](/Img/CDN/CDN3.png)

에지 로케이션에 원하는 파일이 있다면 오리진에 접속하지 않고 바로 파일을 전송 받을 수 있습니다. 캐시 파일이 유지되는 시간은 기본적으로 24시간이며 HTTP 헤더의 Cache-Control을 이용하여 시간을 조절할 수 있습니다. 그리고 에지 로케이션에 있는 파일이 즉각 갱신되어야 할 경우 무효화요청을 통해 캐시된 파일을 삭제시킬 수 있습니다.

CloudFront는 일반적인 CDN 서비스와는 다른 특징이 있습니다.

- 동적 콘텐츠 전송(Dynamic Content Delivery)을 지원합니다.
    - URL 규칙에 따라 정적 페이지는 캐시하고, 동적 페이지는 바로 EC2 인스턴스로 접속하도록 구성할 수 있습니다.
    - HTTP GET 메서드 이외에도 HEAD, POST, PUT, DELETE, OPTIONS, PATCH 메서드를 지원합니다. 특히 POST, PUT, DELETE, OPTIONS, PATCH 메서드는 캐시되지 않고 곧바로 오리진으로 전달됩니다.
    - HTTP 쿠키를 지원합니다. CloudFront 에지 로케이션이 중간에서 쿠키를 전달해줍니다.
- 동영상 전송을 위한 라이브 스트리밍 프로토콜을 지원합니다. Microsoft Smooth Streaming, Adobe RTMP Streaming 등의 다양한 프로토콜을 지원합니다.
- 사용한 만큼 가격을 지불합니다.

<br><br>

# CloudFront 배포

CloudFront 배포는 CloudFront의 가장 기본적인 단위입니다. 이 배포가 독립적인 도메인을 가지고 후에 DNS 서비스인 Route 53이나 별도의 DNS 서버에서 배포 도메인을 CNAME으로 설정하면 사용자가 구매한 도메인과 연결할 수 있습니다.

여기서 중요한 개념이 오리진(Origin)입니다. CloudFront는 CDN서비스이기 때문에 항상 오리진 서버가 있어야 합니다. 

CloudFront가 지원하는 오리진은 다음과 같습니다.

- S3 버킷 : 가장 기본적인 오리진 입니다. S3에 버킷을 생성하고 파일을 저장하기만 하면 되기 때문에 가장 간편합니다.
- EC2 인스턴스 : EC2 인스턴스에 웹 서버를 구축하면 오리진으로 사용할 수 있습니다.
- ELB(Elastic Load Balancing) : EC2 인스턴스 여러 대의 부하를 분산하는 ELB도 오리진으로 사용할 수 있습니다.
- AWS 이외의 웹 서버 : 웹 서버의 형태만 띠고 있다면 운영체제, 서버 애플리케이션의 종류에 상관없이 오리진으로 사용할 수 있습니다.

<br><br>

# S3와 CloudFront 연동하기

![image4](/Img/CDN/CDN4.png)

CloudFront 배포 생성을 클릭합니다.

<br>

![image5](/Img/CDN/CDN5.png)

- 원본 도메인
    - 오리진으로 사용할 도메인을 설정합니다.
    - S3, EC2, ELB 등 CloudFront가 지원하는 오리진을 선택합니다.
- 이름
    - 원본 도메인을 선택하면 자동으로 설정됩니다.
- S3 버킷 액세스
    - S3 버킷에 CloudFont만 접근할 수 있도록 설정하는 옵션입니다.
- Origin Shield 활성화
    - 오리진의 접속을 최소화하기 위해 추가 캐싱 계층을 설정하는 옵션입니다.

<br>

![image6](/Img/CDN/CDN6.png)

- 경로 패턴
    - CloudFront로 파일을 가져올 규칙입니다.
    - 기본값은 *로 설정되어 있어서 모든 파일을 가져오게 됩니다.
- 뷰어 프로토콜 정책
    - CloudFront로 보여질 프로토콜 정책입니다.
    - HTTP and HTTPS: HTTP와 HTTPS를 둘 다 사용합니다.
    - Redirect HTTP to HTTPS: 모든 HTTP 접속을 HTTPS로 리다이렉트 합니다.
    - HTTPS only : HTTPS만 사용합니다.
- 허용된 HTTP 방법
    - 허용하는 HTTP 메서드 종류를 설정합니다.
    - GET, HEAD : 주로 파일을 읽기만 할 때 선택합니다.
    - GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE : 동적 콘텐츠 전송을 사용할 때 선택합니다.

<br>

![image7](/Img/CDN/CDN7.png)

- 캐시 정책
    - CachingOptimized
        - 기본 정책으로 캐시 키에 포함된 값을 최소화 하여 캐시 효율성을 최적화한다.
        - 캐시 키에 쿼리 문자열이나 쿠키를 포함하지 포함하지 않는다.
        - 압축된 형식의 객체도 캐시가 가능
    - CachingOptimizedForUncompressedObjects
        - 기본 정책으로 캐시 키에 포함된 값을 최소화 하여 캐시 효율성을 최적화한다.
        - 캐시 키에 쿼리 문자열이나 쿠키를 포함하지 포함하지 않는다.
        - 압축된 객체 캐시 설정을 비활성화
    - CachingDisabled
        - 캐싱을 비활성화
        - 동적 컨텐츠 및 캐시할 수 없는 요청에 주로 사용한다.
    - Elemental-MediaPackage
        - AWS Elemental MediaPackage 엔드포인트인 오리진과 함께 사용할때 설정
    - Amplify
        - AWS Amplify 웹 앱인 오리진과 함께 사용하때 설정
        
- 원본 요청 정책
    - 오리진에서 원본 파일을 요청하기 위한 정책 설정
    - UserAgentRefererHeaders
        - `User-Agent`, `Referer` 헤더만 포함
    - CORS-CustomOrigin
        - 오리진이 사용자 지정 오리진일 때 CORS(Cross-Origin Resource Sharing)요청을 활성화 하는 헤더가 포함
    - CORS-S3Origin
        - 오리진이 Amazon S3 버킷일 때 CORS 요청을 활성화하는 헤더가 포함
    - AllViewer
        - 뷰어 요청의 모든 값(헤더, 쿠키, 쿼리 문자열)이 포함됩니다.
    - Elemental-MediaTailor-PersonalizedManifests
        - AWS Elemental MediaTailor 엔드포인트인 오리진과 함께 사용할때 설정

<br><br>

![image8](/Img/CDN/CDN8.png)

- 가격 분류
    - 엣지 로케이션 사용 범위를 설정합니다. 세부적으로는 설정할 수는 없으며 실제 서비스에서 그다지 필요가 없는 지역을 제외할 때 설정합니다.
- 대체 도메인 이름
    - Route 53에서 도메인을 연결하려면 이 부분에서 설정합니다.
    - 기존 구입한 도메인이 있을 경우 구입한 도메인 이름을 설정하면 됩니다.
    - 여러 도메인도 설정가능합니다.
- 사용자 정의 SSL 인증서
    - HTTPS 프로토콜을 사용하기 위한 인증서 설정입니다.
- 기본값 루트 객체
    - CloudFront 배포 도메인의 최상위(Root)로 접속했을 때 기본적으로 보여줄 파일 이름입니다.

<br>

![image9](/Img/CDN/CDN9.png)

- 표준 로깅
    - S3 버킷 - CloudFront 로그를 저장할 S3 버킷을 선택합니다.
    - 로그 접두사 - S3 버킷에 로그를 저장할 때, 디렉터리 명을 설정합니다.
    - 쿠키 로깅 - 로그에 쿠키를 포함여부를 설정합니다.

설정이 완료되었으면 배포 생성을 클릭합니다.

![image10](/Img/CDN/CDN10.png)

<br>

생성이 완료되면 도메인 이름이 표시됩니다.

웹 브라우저에서 `https://도메인이름` 을 통해 CloudFront에 접속할 수 있습니다. 

`https://도메인이름/example.png` 와 같이 뒤에 S3에 저장되어 있는 객체를 CloudFront를 통해 접근이 가능합니다.