# [ Java Collcetion Framework(JCF) ]

다음은 Java 컬렉션 프레임워크의 상속구조이다.

![Map1](/Img/Map1.png)

오늘은 이중에 Map에 대해 공부해보고자 한다.

Map은 list,Set과 다르게 Collection을 상속받고 있지 않습니다.

<br>

## Collection Interface

모든 Collection의 상위 인터페이스로써 Collection들이 갖고 있는 핵심 메서드를 선언하고 있다.

```java
public interface Collection<E> extends Iterable<E> {

		int size();

		boolean isEmpty();

		boolean contains(Object o);

		Iterator<E> iterator();

		Object[] toArray();

		boolean add(E e);

		boolean remove(Object o);

		boolean containsAll(Collection<?> c);
		.
		.
		. //등등
}
```

우리가 자주사용하면 메서드들이 선언되어 있는걸 볼 수 있다.

<br>

## Map Interface

Map은 List, Set과는 조금 다르게 Key와 Value의 형태로 연관지어 저장하는 객체이다.

그래서 기존의 Collection의 것을 사용할 수 없고, 따로 기능을 선언해줬다.

<br>

Map interface 코드를 보면

```java
public interface Map<K, V> {

		int size();

		boolean isEmpty();

		boolean containsKey(Object key);

		V get(Object key);

		V put(K key, V value);

		V remove(Object key);
		.
		.
		. // 등등
}
```

새롭게 Map의 자료구조에 맞춰 메서드들이 정의 되어 있는걸 볼 수 있다.

 Map의 주요 특징으로는 키(Key)의 중복을 허용하지 않으나 값(value)의 중복은 허용한다.

 <br><br>

# [ Map의 종류 ]

**HashMap**

- Map 인터페이스를 구현하기 위해 해시테이블을 사용한 클래스
- 중복을 허용하지 않고 순서를 보장하지 않음
- 키와 값으로 null이 허용

**Hashtable**

- HashMap 보다는 느리지만 동기화가 지원
- 키와 값으로 null이 허용되지 않음

**TreeMap**

- 이진검색트리의 형태로 키와 값의 쌍으로 이루어진 데이터를 저장
- 정렬된 순서로 키/값 쌍을 저장하므로 빠른 검색이 가능
- 저장시 정렬을 하기 때문에 저장시간이 다소 오래 걸림

**LinkedHashMap**

- 기본적으로 HashMap을 상속받아 HashMap 과 매우 흡사
- HashMap은 순서를 유지하지 않지만 LinkedHashMap은 순서를 유지시켜 줍니다.

<br>

## MultiValueMap

MultiValueMap은 spring Framework가 제공해주는 인터페이스 입니다.

자바 레퍼런스를 찾아보니 안나오고 spring 레퍼런스에서 보이는군요.

[MultiValueMap](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/MultiValueMap.html)

MultiValueMap은 기존의 Map<Key, Value> 구조를 좀 더 확장해서 Map<Key, List<V>> 로 Value가 list 형태로 선언되어 있습니다.

주로 HTTP header의 내용을 받을때 사용하는 것 같습니다.


```java
MultiValueMap<String, String> paramMap = new LinkedMultiValueMap<>();
paramMap.add("Content-Type", "application/json", "text/plain");
```

<br><br>

## ConcurrentMap

ConcurrentMap 인터페이스는 Map<K,V> 를 확장한것으로

멀티쓰레드 환경에서 사용할 수 있도록 나온 클래스가 ConcurrentHashMap 입니다.

멀티쓰레드 환경이여서 동시성 문제가 발생할 가능성이 있으면 ConcurrentHashMap 사용을 고민해 봐야 합니다.

<br>

**장점**

- Hashtable의 단점을 보완했다.
- Thread-Safe 임에도 불구하고 검색작업(get)에는 Lock이 수반되지 않으며, 전체 테이블을 잠궈야 하는 액션도 없다.
- 검색작업은 Lock이 이루어지지 않으며 갱신작업(put,remove 등)과 동시에 수행 될 수 있다.