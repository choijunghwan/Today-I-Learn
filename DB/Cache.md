# 캐시란?

데이터베이스는 SQL문을 빠르게 처리하기 위한 많은 방법을 사용합니다.

그중에서도 데이터베이스의 처리속도에 가장 큰 영향을 미치는 것은 디스크 I/O 동작입니다.그래서 데이터베이스는 '캐시'라고 불리는 기술을 사용해 가능한 디스크에서 처리하지 않고, 메모리에서 처리하는 구조를 갖고 있습니다.

빈번하게 사용하는 데이터를 매번 디스크에서 꺼내오지 않고 캐시라고 불리는 메모리에 두고 꺼내오므로써 디스크 I/O하는 동작을 생략해 처리속도를 올리는 것입니다.

오라클에서의 데이터 캐시는 '버퍼 캐시'라고 불립니다.

오라클에서 버퍼 캐시의 존재 유무에 따라 동작이 어떻게 다른지 한번 살펴보겠습니다.

![image1](/Img/DB/cache1.png)

**<버퍼 캐시에 데이터가 존재할 때>**

1. 클라이언트가 `'데이터1'`을 요청합니다.
2. 요청한 데이터가 버퍼 캐시에 놓여 있는지 확인합니다.
3. 버퍼 캐시에 요청한 데이터가 존재하므로 해당 데이터를 클라이언트에게 전송해줍니다.

![image2](/Img/DB/cache2.png)

**<버퍼 캐시에 데이터가 존재하지 않을 때>**

1. 클라이언트가 `'데이터2'`을 요청합니다.
2. 요청한 데이터가 버퍼 캐시에 놓여 있는지 확인합니다.
3. 버퍼 캐시에 요청한 데이터가 없어서 데이터를 디스크에서 읽어옵니다.
4. 읽어온 데이터를 버퍼 캐시에 저장한뒤 데이터를 클라이언트에게 전송해줍니다.

캐시는 데이터 뿐만 아니라 인덱스도 캐시할 수 있습니다.

여기서 인덱스를 3계층이라고 가정하고 인덱스를 사용해 테이블에서 데이터를 한건만 꺼내오는 SQL문에 걸리는 시간을 예측해보겠습니다.

<br>

![image3](/Img/DB/disk2.png)

'빌'이라는 데이터를 찾는 과정을 살펴보겠습니다.

I/O 한번 당 10밀리초, SQL 처리에 1밀리초가 걸린다고 가정해보겠습니다.

<br>

### 캐시를 사용하지 않을때

1. `'빌'`이라는 데이터가 버퍼 캐시에 없으니 디스크에서 읽어 와야합니다.
2. 인덱스를 통해 **32번 주소**를 읽어라는 것을 알 수 있습니다. (+10밀리초)
3. 32번 주소에 대한 데이터가 버퍼 캐시에 없으므로 디스크에서 읽어 와야합니다.
4. 32번 주소를 통해 **151번 주소**를 읽어라는 것을 알 수 있습니다. (+ 10밀리초)
5. 151번 주소에 대한 데이터가 버퍼 캐시에 없으므로 디스크에서 읽어 와야합니다.
6. 151번 주소를 통해 **501번 주소**를 읽어야 하는 것을 알 수 있습니다. (+10밀리초)
7. 501번 주소에 대한 데이터가 버퍼 캐시에 없으므로 디스크에서 읽어 와야합니다.
8. 501번 주소 데이터가 `'빌'`이므로 '빌'이라는 데이터를 읽어 옵니다.(+10밀리초)

<br>

인덱스를 사용하더라고 '빌'이라는 데이터를 읽어오기 위해 디스크 I/O 동작이 총 4번 반복되어야 하고 그로인해 총 처리 시간은 **40밀리초 + 1밀리초 = 41밀리초**가 소요되게 됩니다.

하지만 캐시에 '빌'이라는 데이터가 있다면 처리시간은 **1 밀리초**로 줄어들게 됩니다.

캐시를 사용했을의 효과와 왜 캐시를 사용해야 하는지를 이해 할 수 있을 것이라고 생각합니다.

<br><br>

# 공유메모리

그러면 캐시는 어디에 저장하고 있을까요??

만약 각 프로세스 마다 캐시를 가지고 있다면 다른 프로세스에서 변경된 데이터를 볼 수 없고 각 프로세스마다 캐시메모리를 할당해야 하므로 낭비가 심해지는 문제가 발생합니다.

하지만 OS에서 다른 프로세스의 메모리를 보는 것은 불가능합니다. 프로세스는 자원을 공유하지 않는 특성을 가지고 있기 때문입니다. 이러면 DBMS 입장에서는 불편함이 발생할 수 밖에 없으므로, OS는 **'공유 메모리'**라는 특수한 메모리 기능을 제공합니다.

공유메모리를 사용하면 자신의 메모리 영역에 기록했던 데이터를 다른 프로세스에서도 즉시 볼 수 있습니다.

각 프로세스에게는 마치 자신의 메모리인 것처럼 보이지만 실제로는 하나의 메모리를 모든 프로세스가 공유하고 있는 형태입니다. 오라클에서는 이런 공유 메모리를 'SGA(System Globaal Area)', 공유하지 않는 메모리의 일부를 'PGA(Program Global Area)'라고 합니다.

하지만 이 공유 메모리에는 누구든지 접근할 수 있으므로 Lock을 걸어서 배타적 제어를 하지 않으면 데이터에 손상을 입을 수도 있습니다. 그래서 DBMS는 데이터가 손상되는것을 막기 위해 Lock이라는 기능을 이용하고 있습니다.

<br>

### 세마포어

세마포어는 OS가 제공하는 자원을 관리하기 위한 장치의 일종으로, 자원의 수에 비해 사용하고자 하는 프로세스의 수가 많을 경우에 순서대로 자원을 사용할 수 있도록 프로세스 제어를 수행합니다.

<br><br>

# 버퍼 캐시 LRU 알고리즘

버퍼 캐시는 자주 사용하는 데이터를 더 빠르게 가져오기 위해 존재합니다. 허나 버퍼 캐시의 크기는 한정되어 있으므로, 어떤 식으로든 관리하고 정리해야 합니다.

캐시에 사용하는 알고리즘 중 일반적으로 알려진 것이 '**LRU(Least Recently Used)'** 알고리즘입니다.

간단하게 최근에 사용하지 않은 데이터부터 정리하는 알고리즘입니다.

그러면 오라클의 버퍼 캐시가 LRU 알고리즘을 통해 어떻게 동작하고 있는지 한번 보겠습니다.

먼저 버퍼캐시에 적재할 수 있는 블록은 4개이며, 이미 ABCD 순서로 4개의 블록이 캐시에 적재되어 있다고 가정합니다.

![image4](/Img/DB/cache3.png)

1. E라는 데이터를 요청
2. 버퍼 캐시에 놓여 있지 않은지를 확인
3. 버퍼 캐시에 없다면 데이터를 디스크에서 읽어옴
4. 읽어온 데이터를 캐시에 놓음. 이때 LRU 알고리즘 기반으로 어떤 블록을 정리할지 결정
    1. 버퍼 캐시에 ABCD의 순서로 적재되었으므로 가장 오래 사용하지 않은 데이터는 'A'이다.
    2. 'A'데이터를 정리한뒤 'E' 데이터를 적재해준다.

또한, 자주 사용하지 않는 데이터를 버퍼 캐시에 둘 필요는 없습니다. 예를 들어 데이터가 많은 테이블을 풀 스캔한 데이터는 적재해두더라도 사용하는 빈도도 적고, 기존에 자주 사용하던 데이터를 캐시에서 쫓아내는 일이 발생하게 됩니다. 그래서 오라클은 큰 테이블이라고 판단하면 풀 스캔했을 때의 데이터를 버퍼 캐시에 적재하지 않습니다.

<br><br>

# 가상 메모리

캐시는 오라클 뿐만 아니라 OS에도 버퍼 캐시가 있습니다. 추가로 OS에 가상 메모리라고 불리는 기능이 있습니다. OS의 버퍼 캐시는 오라클의 버퍼 캐시와 비슷한 기능을 가지고 있습니다.

 

OS에서는 가상 메모리라는 기능을 이용해 실제로 가지고 있는 물리 메모리보다 더 많은 메모리를 사용할 수 있습니다. 가상 메모리는 자주 사용되지 않는 데이터를 디스크에 저장합니다. 하지만 프로세스 관점에서 데이터는 메모리에 있는 것처럼 보이는 구조입니다.

가상메모리는 버퍼 캐시와는 반대로 동작하도록 구현되어 있습니다. 버퍼 캐시는 디스크에 빠르게 접근하지 위해 메모리의 일정 부분을 할당하여 사용합니다. 반면에 가상메모리는 속도가 느려지더라도 사용할 수 있는 메모리의 크기를 늘리기 위해 사용합니다.

![image4](/Img/DB/cache4.png)

가상 메모리의 동작 방식

1. 최근에 사용하지 않아 'A'라는 데이터를 디스크에 옮겨놓습니다.
2. 클라이언트가 'A'라는 데이터를 요청합니다.
3. 프로세스가 요청한 데이테가 실제 메모리에 존재하지 않기 때문에 디스크에서 읽어 옵니다.

이렇게 가상 메모리에 있는 데이터를 요청하게 되면 프로세스는 메모리에 있는것처럼 생각하여 데이터를 요청하지만 실제로는 디스크에서 데이터를 읽어오기때문에 속도가 느려지게 됩니다.

그러면 가상 메모리를 왜 사용할까요??

속도가 느려지더라도 가상메모리를 사용하는 이유는 다중 프로그래밍을 실현하기 위해서는 많은 프로세스들을 동시에 메모리에 올려두어야 합니다.. 가상메모리는 프로세스 전체가 메모리 내에 올라오지 않더라도 실행이 가능하도록 하는 기법 이며, 프로그램이 물리 메모리보다 커도 된다는 주요 장점이 있다.

<br>

### 버퍼 캐시 와 가상메모리의 관계

일반적으로는 버퍼캐시는 속도를 빠르게 도와주고 가상메모리는 메모리 크기를 늘려주는 작업을 통해 각각이 작동하는 것처럼 보이지만 조금 더 생각해보면 주의해야 할 상황이 있습니다.

버퍼 캐시와 가상 메모리는 밀접한 관계가 있습니다.

예를 들어 오라클의 버퍼 캐시를 물리 메모리보다 많게 설정한다고 가정해보겠습니다.

그러면 우리는 데이터를 가져올때 버퍼 캐시를 통해 빠르게 가져오기를 기대하지만 버퍼 캐시가 가상 메모리에 적재되어 있으면 실제 데이터는 버퍼 캐시에 있는 것이 아니므로 디스크에 접근하여 데이터를 가져오게 됩니다. 그러면 버퍼 캐시의 장점이 가상메모리때문에 상쇄되어 버리는 것입니다.

<br><br>

# 정리

이렇게 오라클의 아키텍처를 알고 있다면 다양한 상황에 대해 답을 알 수 있는 것이 많이 있습니다.

아래와 같은 상황에 대해 한번 생각해보겠습니다.

**질문**

- 오라클에서 같은 성능 테스트를 반복해서 수행하고 있습니다. 첫 번째 테스트가 종료한 후 캐시를 정리하기 위해 오라클을 재기동했더니 두번째의 테스트에서는 속도가 빨라졌습니다. 왜 그럴까요??

정답은 'OS의 버퍼 캐시에 데이터가 적재되어 있기 때문'입니다. 오라클을 재기동해도 OS는 재기동하지 않았기 때문에 OS의 버퍼 캐시에 데이터가 남아 있다는 의미입니다.

**질문**

- 성능 테스트를 통해 버퍼 캐시에서 데이터를 가져오는 것을 확인하였습니다. 그런데 버퍼 캐시를 사용하지 않는것과 속도가 비슷하였습니다. 왜 그럴까요??

정답은 '버퍼 캐시를 물리 메모리보다 크게 설정하여 가상 메모리에 있는 데이터를 가져오기 때문'입니다.

실제로 다른 이유가 있을 수 있지만 공부한 내용을 정리 할 수 있도록 문제를 만들어 보았습니다. 

위의 질문을 통해 내용을 한번 정리 할 수 있으면 좋을 것 같습니다.

그리고 아키텍쳐와 구조를 아는것이 추후에 더 좋은 설계나 문제에 대한 해결책을 찾는데 도움이 될 수 있을거라고 생각하고 구조에 대한 이해의 중요성을 느낄 수 있었습니다.

### 참고자료
* <그림으로 공부하는 오라클구조> 책
