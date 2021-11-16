# DBMS

오라클을 이해하기 위해 3가지 키워드를 소개하겠습니다.

**키워드**

1. 병렬 처리를 가능케 하고 높은 처리량을 실현한다.
2. 응답 시간(response time)을 중시한다.
3. 커밋(commit)한 데이터는 지킨다.

오라클을 포함한 모든 DBMS(Database Management System)은 매우 복잡합니다.

DBMS가 복잡해지는 이유는 위에서 이야기한 3가지의 특성을 모두충족시켜야 하기 때문입니다. 세가지 특성은 상반된 성향이 있어서 동시에 모두 만족시키기가 매우 어렵습니다.

예를 들어, '커밋(commit)한 데이터를 지킨다'를 만족하기 위해 커밋하는 순간 데이터를 디스크에 기록하고 싶지만, 그렇게 하면 응답 시간이 나빠지게 됩니다.

그러면 오라클은 이러한 특성을 어떻게 반영하고 있는지 차근차근 알아가 보겠습니다.

<br><br>

# 오라클과 디스크(하드디스크)

오라클은 디스크에서 데이터를 읽어오고, 필요한 처리를 한 후 다시 디스크에 기록합니다. 즉, 오라클이 다루는 데이터는 디스크에서 꺼내오고 다시 디스크로 돌아갑니다.

그러면 디스크가 어떻게 동작하는지 그림을 보고 설명하겠습니다.

![image1](/Img/DB/disk1.png)

위의 그림에서 디스크는 거의 항상 회전하고 있으며, 그 위로 헤드가 움직여서 데이터를 읽거나 기록합니다. 그리고 1분에 1만 번 정도의 엄청난 속도로 회전합니다.

그러면 I/O(input/output) 처리에 필요한 디스크의 동작을 살펴보겠습니다.

1. 데이터를 읽기 위해서 원하는 데이터가 저장되기 시작한 첫 머리를 찾는다. (탐색, seek)
2. 디스크에서 원하는 정보를 읽을 수 있는 위치가 회전해서 다가올 때까지 기다린다. (회전 대기 시간, rotational latency time)
3. 해당 위치가 다가오면 데이터를 읽고 씁니다.

I/O 처리에 단점이 있습니다. 그것은 접근 시간이 오래 걸린다는 것입니다.

메모리에 접근할 때는 나노초(ns) 단위로 수행할 수 있으나, 디스크에 접근할 때는 밀리초(ms) 단위의 시간이 필요합니다. 사람의 입장에서 보면 디스크에 접근하는 속도도 빨라보이지만 컴퓨터의 입장에서는 매우 느린 속도입니다.

이유를 간단하게 설명하자면 메모리는 전기 신호로 작업을 처리하지만, 디스크는 기계 동작이 필요하기 때문입니다. 하지만 DBMS의 입장에서 보면 디스크 I/O는 반드시 필요한 것이면서도 처리 시간을 단축하기 위해서는 가능한 한 줄여야만 하는 부분이기도 합니다.

<br><br>

# 인덱스

그러면 어떤 방식으로 I/O의 대기 시간을 줄일 수 있을까요??

<br>

### 시퀀셜 액세스

그 답을 알아보기 위해 먼저 Sequential(시퀀셜) 액세스에 관해 설명하겠습니다. Sequential(시퀀셜)은 '순서를 따라서'라는 의미의 '순차'를 뜻하며, 시작점에서부터 마지막까지 중간 부분을 빠트리지 않고 전부 액세스 하는 것을 의미합니다. 메모리에 테이블의 데이터가 없으면 풀 스캔(Full scan, 테이블의 모든 데이터를 읽어오는 것)할 때 시퀀셜 액세스가 발생합니다.

테이블의 크기를 1GB이고 전송 속도를 20MB/초 라고 가정하고 모든 데이터를 시퀀셜 액세스로 가져온다면 50초(1,000MB/20MB/초) 가 걸립니다. 데이터를 가져오는데 거의 1분이 걸린다면 SQL 성능에 만족할 수 없을 것입니다.

그래서 인덱스(index)라는 발상이 나오게 되었습니다. 책에서 무엇인가를 찾고 싶을 때를 생각해보면 대부분은 색인을 이용할 것입니다. 색인에는 키워드가 순서대로 나열되어 있으며, 해당 페이지 번호도 함께 기재되어 있습니다. 데이터베이스의 인덱스도 마찬가지입니다. 인덱스에는 검색할 때 사용하는 키 값과 그 키가 존재하고 있는 위치가 기록되어 있습니다.

그렇다면 인덱스 자체의 크기가 커지면 크기가 큰 테이블을 조회하는 것과 마찬가지로 작업에 필요한 시간이 늘어나게 될까요?? 결론은 아닙니다. 오라클의 인덱스는 '인덱스에 인덱스를 추가하는 것'처럼 여러 계층으로 구성되기 때문입니다. 이런 부분이 책에서 사용하는 색인과 다른 점입니다.

![image2](/Img/DB/disk2.png)

<br>

### 랜덤 액세스

인덱스를 사용할 때는 필요한 부분만 읽어오면 충분하지만, 필요한 부분이 디스크 위에 연속적으로 존재하는 경우는 거의 없습니다. 따라서 헤드를 움직여가면서 띄엄띄엄 접근하게 됩니다. 이렇게 접근하는 방식을 '랜덤 액세스(random access)'라고 하며, 시퀀셜 액세스와 반대의 의미가 있습니다.

디스크의 관점에서 생각해보면 랜덤 액세스는 탐색하는 작업과 회전 대기로 인해 데이터에 띄엄띄엄 접근할 때 마다 어쩔 수 없이 시간이 소모 되기 때문에 비효율적입니다.

그치만 만약 많은 수의 데이터를 보려고 할 때 매번 인덱스를 찾은 후 데이터를 찾아가게 되면 어떻게 될까요?? 네 속도가 느려집니다. 예를 들어, 테이블에 2만 건인 데이터가 저장되어 있다고 하고 그중 절반인 1만 건을 꺼낸다고 가정하겠습니다. 여기서한 로우는 8KB, 1초에 탐색할 수 있는 횟수를 100회, 디스크 전송 속도(헤드를 움직이지 않고 일고 쓸 수 있을 때나 가능한 처리량)를 20MB/초 라고 가정했을 때 '랜덤 액세스'방식을 사용한다면 1초 동안 800KB(8KB * 100)를 읽을 수 있고 1만건을 읽는데 100초가 걸립니다. '시퀀셜 액세스'방식을 사용한다면 모든 데이터(2만건)을 읽어온다고 해도 8초면 끝납니다. 

즉, 디스크 특성상 모든 데이터가 아니더라도 일정 크기 이상의 데이터를 읽는다면 시퀀셜 액세스를 사용하여 테이블을 풀 스캔하는 편이 빠릅니다.

그래서 인덱스는 전체 데이터의 약 15% 미만인 경우 유리하다라고 말합니다. 단 실제로는 캐시에 데이터가 보관된 경우도 있으므로 대략적인 기준이라고만 생각하시면 좋을것 같습니다.

<br><br>

# 데이터베이스란?

오라클은 SQL문과 그 결과의 데이터를 프로그램과 송수신합니다. DBMS인 오라클은 해당 데이터를 디스크에 저장하고 관리합니다. 고객인 각종 프로그램은 여러 개 존재할 수 있으며, 의뢰를 받는 입장인 오라클의 프로세스도 여러 개 존재합니다. 이 '여러 개'라는 개념은 DBMS를 이해하는 데 있어서 매우 중요합니다.

데이터베이스를 사용하지 않는 애플리케이션의 프로그래밍에서는 각각의 프로세스가 자신이 가진 변수(데이터)를 처리하는 것이 일반적입니다. 같은 프로그램이 여러 개 실행되었다 하더라도 실제로 변수는 개개의 프로그램마다 존재하기 때문에 프로세스간의 간섭을 신경을 쓸 필요가 없습니다. 하지만 데이터베이스에서는 여러 프로세스나 사용자가 하나의 데이터베이스에 접근하기 때문에 다른 사용자나 애플리케이션을 신경 쓰지 않을 수 없습니다.

예를들어 주문이력이라고 하는 테이블이 있고, 크기는 10GB이며, 이 테이블을 사용하는 사용자가 100명이 있다고 가정하겠습니다. 주문 이력 테이블을 사용하는 데 서로 간섭을 일으키지 않도록 100명분 X 10GB의 데이터를 가지고 있어야 할 필요가 있을까요?? 또한 어느 사용자가 주문 이력 테이블에 데이터를 추가한다면 입력한 주문 데이터를 다른 사용자가 볼 수 없다고 해도 상관없을까요??

이런 부분에서 알 수 있듯이, 기본적으로 데이터베이스는 데이터를 중복으로 보관하지 않으며, 보관해서도 안 됩니다. 기본적으로 '여러 사용자나 프로그램이 데이터베이스의 데이터를 공유하고 있다'라고 생각하는 것이 중요합니다.

DBMS는 다수의 사용자나 애플리케이션이 데이터를 공유한다는 전제로 만들어져 있어서 많은 사용자가 데이터를 동시에 검색하거나 변경할 수 있습니다. 또한 다수의 사용자가 동시에 데이터를 처리하는 경우에는 사용자의 실수로 데이터가 손상되지 않도록 처리하는 Lock이라는 장치를 가지고 있습니다.

예를 들자면 '이제부터 데이터를 변경할 테니 다른 사람은 데이터를 변경할 수 없게 하겠다'라든지 'Lock이 걸린 데이터를 조작하려고 하면 Lock이 풀릴 때까지 기다린다'와 같은 방식입니다.

<br>

### 프로세스와 스레드란?

**프로세스**는 실행 중인 상태에 있는 프로그램을 의미합니다. 실행 중인 상태이기 때문에 메모리나 자원을 가지고 있습니다.프로그램이 여러개 실행되어 있다고 해도 그것들은 각각 프로세스이므로 CPU가 여러 개 있다면 동시에 처리할 수 있습니다.

이에 비해 **스레드**는 프로세스 내에 존재하는 실행 단위를 말합니다. 하나의 프로세스 안에서 병렬로 작업을 처리하고 싶을 때 사용합니다.

어느 쪽이든 병렬로 처리하기 위한 구조이지만, 가장 다른 점은 부하의 크기와 메모리를 공유하는지 여부입니다. 프로세스는 각자 독립적으로 수행되어 자원을 독자적으로 사용하므로 부하가 크고, 메모리를 공유하지 않습니다. 스레드는 한 프로세스 안에서 수행되므로 부하가 적지만 스레드끼리 메모리를 공유하므로 주의해서 사용해야 합니다.

<br><br>

# 서버프로세스와 백그라운드 프로세스

오라클은 여러 개의 프로세스로 구성되어 있습니다. 어째서 여러 개의 프로세스로 구성되어 있을까요??

그 이유 중 한가지는 다중 처리를 위함입니다.디스크는 메모리 액세스에 비해서 속도가 매우 느립니다. CPU와 같은 자원을 속도가 느린 I/O가 반복되고 있는 동안에 방치하는 것은 매우 아까우므로 가능하다면 I/O가 반복되는 그 순간에는 다른 SQL 처리를 하는 것이 효율적입니다.

오라클은 여러 개의 프로세스를 실행하는 방식으로 병렬 처리를 사용합니다.

여러 개를 실행한다는 의미는 일반적으로 OS상에서 작업을 처리할 때 여러 개의  프로세스를 사용하는 것을 말합니다.

오라클은 다음의 두 가지 프로세스로 구성되어 있습니다.

- 서버 프로세스(SQL문을 처리하는 프로세스)
- 백그라운드 프로세스(주로 서버 프로세스를 지원하는 프로세스)

**백그라운드 프로세스 역할**

- ora_**dbw**X_XXXXX  : 'DBWR(DataBase Writer)'이라고 부름. 데이터를 디스크에 기록함
- ora_**lgwr**_XXXXX : 'LGWR(Log Writer)'이라고 부름. 로그(데이터 변경 이력)를 디스크에 기록함
- ora_**pmon**_XXXXX : 'PMON(Process MONitor)'이라고 부름. 프로세스를 감시하고, 프로세스에 장애(비정상 종료 등)를 발견했을 때 정리하는 역할을 함
- ora_**arcX**_XXXXX : 'ARCH(ARCHiver)'라고 부름. 로그 데이터를 아카이브(장기 보관하기 위해 별도의 파일로 보관)함

위는 서버 프로세스를 도와주는 백그라운드 프로세스의 일부 역할입니다. 오라클 클라이언트는 서버 프로세스와 통신을 하고 백그라운드 프로세스는 뒤에서 각종 지원을 해줍니다.

**서버 프로세스와 백그라운드 프로세스**로 나뉘어져있기 때문에 서버 프로세스는 SQL문 처리에만 집중을 할 수 있고 덕분에 좀 더 빠른 처리가 가능합니다.

<br><br>

# 프로세스 역할

오라클에서 수행하는 프로세스의 주요 처리는 다음과 같습니다.

1. SQL문의 수신
2. SQL문의 파싱(어떤 테이블에 어떻게 접근해야 하는지를 생각하는 작업)
3. 데이터 읽기(디스크에서 읽어온다)
4. 데이터 기록(디스크에 기록한다)
5. SQL문의 결과 회신
6. 로그 기록(데이터의 변경 로그를 디스크에 기록)
7. 각종 정리(프로세스의 비정상 종료로 인한 아무도 사용하지 않는 Lock의 해제 등)
8. 로그 보관(아카이브)

서버 프로세스는 서비스를 수행하는 프로세스로 SQL문을 빠르게 처리 할 수 있는 업무를 처리합니다. 위의 열거한 작업들 중에서는 1,2,3,5 가 이에 해당합니다.

오라클은 SQL문을 수신하면 파싱이라는 절차를 통해 어떤 테이블에 접근해야 할지 검색합니다. 검색이 완료되면 해당 테이블에서 데이터를 읽어오고 읽어온 데이터를 클라이언트에 전달해줍니다.

그러면 여기서 데이터 기록은 하지 않는건가?? 라는 의문이 들 수 있습니다.

결론은 데이터 기록을 하지만 기록에 대한 역할은 SQL문을 처리하는데 꼭 필요한 업무가 아니기 때문에 서버 프로세스가 아닌 백그라운드 프로세스가 수행합니다.

<br><br>

# 정리

공부한걸 요약해보겠습니다.

- 데이터베이스는 다수의 사용자가 데이터를 공유해서 사용한다.
- 오라클은 여러 프로세스로 되어 있어 여러 개의 SQL문이 동시에 동작한다.
- 서버 프로세스는 SQL문의 결과를 가능하면 빠르게 회신하기 위한 일을 한다.
- 이런 서버 프로세스를 도와주는 백그라운드 프로세서가 존재한다.

이러한 특징 덕분에 처음에 소개한 주요 키워드인 '병렬 처리'와 '응답 시간'을 오라클이 어떻게 해결하고 있는지 알아보았습니다.

<br>

### 참고자료
* <그림으로 공부하는 오라클 구조>