ArrayList를 구현해보기 전에 실제 Java에서 제공하는 ArrayList는 매우 복잡한 구조이고 구현하고 있는 메서드도 매우 많기 때문에 리스트 인터페이스글에서 구현해놨던 인터페이스를 기준으로 최소한의 메서드만 구현해보도록 하겠다.

ArrayList는 다른 자료구조와 달리 Object[] 배열을 두고 사용한다는 점이다. 

모든 자료구조는 '**동적 할당**'을 전제로 한다. 동적 할당을 안하고 사이즈를 정해놓고 구현한다면 메인함수에서 정적배열을 선언하는 것과 차이가 없다.

마지막으로 리스트 계열 자료구조는 **데이터 사이에 빈 공간을 허락하지 않는다**.

<br><br>

# ArrayList 클래스 및 생성자 구성하기

ArrayList 이름으로 생성해준 뒤 앞서 작성했던 List 인터페이스를 implements 해준다.

```java
import Interface_form.List;

public class ArrayList<E> implements List<E> {

    private static final int DEFAULT_CAPACITY = 10; //최소(기본) 용적 크기
    private static final Object[] EMPTY_ARRAY = {}; //빈 배열

    private int size;  // 요소 개수

    Object[] array; // 요소를 담을 배열

    // 생성자1 (초기 공간 할당 X)
    public ArrayList() {
        this.array = EMPTY_ARRAY;
        this.size = 0;
    }

    // 생성자2 (초기 공간 할당 O)
    public ArrayList(int capacity) {
        this.array = new Object[capacity];
        this.size = 0;
    }
}
```

**DEFAULT_CAPACITY** : 배열이 생성 될 때의 최초 할당 크기(용적)이자 최소 할당 용적 변수로 기본값은 10으로 설정해두었다.

**EMPTY_ARRAY** : 아무 것도 없는 빈 배열 (용적이 0인 배열)

**size** : 배열에 담긴 요소(원소)의 개수 변수

**array** : 요소들을 담을 배열

생성자가 2개를 두었다.

생성자 1의 경우 사용자가 공간 할당을 미리 안하고 객체만 생성하고 싶을 때, 즉 다음과 같은 상태일 때이다.

`ArrayList<Integer> list = new ArrayList<>();`

이 때는 사용자가 공간 할당을 명시하지 않았기 때문에 array 변수를 EMPTY_ARRAY로 초기화 시켜 놓는다.

반면에 사용자가 데이터의 개수를 예측할 수 있어서 미리 공간 할당을 해놓고 싶을 경우, 즉 다음과 같이 생성할 경우

`ArrayList<Integer> list = new ArrayList<>(100);`

array의 공간 할당을 입력 된 수 만큼 배열을 만든다.

<br><br>

# 동적할당을 위한 resize 메서드 구현

우리는 들어오는 데이터에 개수에 따라 '최적화'된 용적을 갖을 필요가 있다. 만약 데이터는 적은데 용적이 크면 메모리가 낭비되고, 반대로 용적은 적은데 데이터가 많으면 넘치는 데이터들은 보관할 수 없게 되는 상황을 마주칠 수 있다.

그렇기 때문에 size(요소의 개수)가 용적(capacity)에 얼마만큼 차있는지를 확인하고, 적절한 크기에 맞게 배열의 용적을 변경해야 한다. 또한 용적은 외부에서 마음대로 접근하면 데이터의 손상을 야기할 수 있기 때문에 private로 접근을 제한해주도록 하자.

```java
private void resize() {
    int array_capacity = array.length;

    // if array's capacity is 0
    if (Arrays.equals(array, EMPTY_ARRAY)) {
        array = new Object[DEFAULT_CAPACITY];
        return;
    }

    // 용량이 꽉 찰 경우
    if (size == array_capacity) {
        int new_capacity = array_capacity * 2;

        // copy
        array = Arrays.copyOf(array, new_capacity);
        return;
    }
    // 용적의 절반 미만으로 요소가 차지하고 있을 경우
    if (size < (array_capacity / 2)) {
        int new_capacity = array_capacity / 2;
        
        // copy
        array = Arrays.copyOf(array, Math.max(new_capacity, DEFAULT_CAPACITY));
        return;
    }
}
```

 현재 array의 용적(=array 길이)과 데이터의 개수(size)를 비교한다.

<br>

조건문 1 : `if(Arrays.equals(array, EMPTY_ARRAY))`

앞서 생성자에서 사용자가 용적을 별도로 설정하지 않은 경우 EMPTY_ARRAY로 초기화 되어있어 용적이 0인 상태다. 이 경우를 고려하여 이제 새로 array의 용적을 할당하기 위해 최소 용적으로 설정해두었던 DEFAULT_CAPACITY의 크기만큼 배열을 생성해주고 메소드를 종료한다.

또한 주소가 아닌 값을 비교해야 하기 때문에 Arrays.equels() 메서드를 사용한다.

<br>

조건문 2 : `if(size == array_capacity)`

데이터가 꽉 찰 경우에는 데이터(요소)를 더 받아오기 위해서 용적을 늘려야 한다. 이 때는 새롭게 용적을 늘릴 필요가 있으므로 새로운 용적을 **현재 용적의 2배**로 설정하도록 한다.

또한 기존에 담겨있던 요소들을 새롭게 복사해와야한다. 이 때 편리하게 쓸 수 있는 것이 Arrays.copyOf() 메서드다. Arrays.copyOf()는 복사할 배열보다 용적의 크기가 클 경우 먼저 배열을 복사한 뒤, 나머지 빈 공간은 null로 채워지기 때문에 편리하다.

<br>

조건문 3 : `if(size < (array_capacity / 2))`

데이터가 용적에 절반 미만으로 차지 않을 경우다. 이 경우 빈 공간이 메모리를 차지하고 있어 불필요한 공간을 낭비하고 있기 때문에 사이즈를 적절하게 줄여주는 것이 좋다.

데이터의 개수가 용적의 절반 미만이라면 Arrays.copyOf()를 통해 용적을 절반으로 줄여준다.

만약 복사할 배열보다 용적의 크기가 작을경우 새로운 용적까지만 복사하고 반환하기 때문에 예외발생에 대해 안전하다.

<br><br>

# add 메서드 구현

add() 메서드를 구현해보자.

add 메서드에는 크게 3가지로 분류 된다.

- 가장 마지막 부분에 추가 (기본값) - addLast(E value)
- 가장 앞부분에 추가 - addFirst(E value)
- 특정 위치에 추가 - add(int index, E value)

물론 현재 자바에 내장되어 있는 ArrayList에는 **addLast()역할을 add() 메서드**가 하고, **특정 위치에 추가는 add(int index, E element)**로 오버로딩된 메서드가 하며 , addFirst(0는 없다. 하지만 편의상 3가지로 나눠서 구현해보겠다.

<br>

### 1. 기본 삽입 : add(E value) & addLast(E value) 메서드

먼저 add()의 기본 값은 addLast()라고 했다.

```java
@Override
public boolean add(E value) {
    addLast(value);
    return true;
}

public void addLast(E value) {
    // 꽉차있는 상태라면 용적 재할당
    if (size == array.length) {
        resize();
    }
    array[size] = value;
    size++; // 사이즈 1 증가
}
```

addLast()는 add를 호출하면 자동으로 파라미터로 넘어온 value는 addLast로 보내진다.

<br>

### 2. 중간삽입 : add(int index, E value)

중간에 추가하는 것은 조금 까다롭다. 우선 index가 범위를 벗어나진 않는지를 확인해야 하고, 중간에 삽입할 경우 기존에 있던 index의 요소와 그의 뒤에 있는 데이터들을 모두 한 칸씩 뒤로 밀어야 하기 때문이다.

addLast는 단순히 데이터를 추가하는 과정이였다면 중간 삽입은 데이터를 미루는 코드가 추가된 것이다.

```java
@Override
public void add(int index, E value) {

    if (index > size || index < 0) {  // 영역을 벗어날 경우 예외 발생
        throw new IndexOutOfBoundsException();
    }
    if (index == size) { // index가 마지막 위치라면 addLast() 메서드로 요소 추가
        addLast(value);
    } 
    else {
        
        if (size == array.length) { // 꽉차있다면 용적 재할당
            resize();
        }   
        
        // index 기준 후자에 있는 모든 요소들 한 칸씩 뒤로 밀기
        for (int i = size; i > index; i--) {
            array[i] = array[i - 1];
        }

        array[index] = value;
        size++;
    }
}
```

먼저 index로 들어오는 값이 size를 벗어나는지, 또는 음수가 들어오는지를 확인한다. 만약에 범위를 벗어나거나 인덱스가 음수가 들어오면 예외를 발생시킨다.

그리고 사용자가넘겨준 index가 size와 같다는 것은 결국 가장 마지막에 추가하는 것과 같은 의미이므로 addLast() 메서드로 가도록 한다.

그 외의 경우가 중간 삽입되는 경우다.

index와 그 후방에 있는 데이터들을 한 칸씩 뒤로 밀어내고 array[index]에는 새로운 요소로 교체해주도록 한다.

### 3. addFirst(E value)

이 부분은 간단하다. 생각해보면 기존 데이터가 있다면 모든 데이터들을 뒤로 밀어야 하는 것이다.

즉앞서 만들었던 중간 삽입에서 index를 0으로 보내면 되는 것이다.

```java
public void addFirst(E value) {
    add(0, value);
}
```

<br><br>

# get, set, indexOf, contains 메서드 구현

### 1. get(int index) 메서드

get()은 index로 들어오는 값을 인덱스 삼아 해당 위치에 있는 요소를 반환하는 메서드다. 배열의 위치를 찾아가는 것이기 때문에 **잘못된 위치 참조에 대한 예외처리**를 꼭 해줘야 한다.

```java
@SuppressWarnings("unchecked")
@Override
public E get(int index) {
    if (index >= size || index < 0) { // 범위 벗어나면 예외 발생
        throw new IndexOutOfBoundsException();
    }
    // Object 타입에서 E타입으로 캐스팅 후 반환
    return (E) array[index];
}
```

여기서 `@SuppressWarnings("unchecked")` 에 대해 알아보자.

`@SuppressWarnings("unchecked")` 을 붙이지 않으면 type safe(타입 안정성)에 대해 경고를 보낸다. 반환되는 것을 보면 E 타입으로 캐스팅을 하고 있고 그 대상이 되는 것은 Object[] 배열의 Object 데이터다. 즉, Object → E 타입으로 변환을 하는 것인데 이 과정에서 변환할 수 없는 타입을 가능성이 있다는 경고로 메서드 옆에 경고표시가 뜨는데, 우리가 add하여 받아들이는 데이터 타입은 유일하게 E 타입만 존재한다.

그렇기 때문에 형 안정성이 보장되므로 ClassCastException이 뜨지 않으니 이 경고들을 무시하겠다는 의미이다. 대신 형 변환신 예외 가능성이 없을 확실한 경우에만 사용해야 한다.

<br>

### 2. set(int index, E value) 메서드

set 메서드는 기존에 index에 위치한 데이터를 새로운 데이터(value)으로 '**교체**'하는 것이다. 

index에 위치한 데이터를 교체하는 것이기 때문에 get이랑 메서드가 유사하다. 다만 get은 해당 인덱스의 값을 반환하는 것이였다면 set은 데이터만 교체해주면 된다.

```java
@Override
public void set(int index, E value) {
    if (index >= size || index < 0) { // 범위 벗어나면 예외 발생
        throw new IndexOutOfBoundsException();
    } 
    else {
        // 해당 위치의 요소를 교체
        array[index] = value;
    }
}
```

<br>

### 3. indexOf(Object value) 메서드

indexOf 메서드는 사용자가 찾고자 하는 요소(value)의 '위치(index)'를 반환하는 메서드다.

만약 찾고자 하는 요소가 중복된다면 **가장 먼저 마주치는 요소의 인덱스를 반환**한다. (실제로 자바에서 제공하는 indexOf 또한 동일하게 구현된다.)

찾고자 하는 요소가 없다면 이러한 경우 -1을 반환한다.

```java
@Override
public int indexOf(Object value) {
    int i = 0;

    // value와 같은 객체(요소 값)일 경우 i(위치) 반환
    for (i = 0; i < size; i++) {
        if (array[i].equals(value)) {
            return i;
        }
    }
    // 일치하는 것이 없을경우 -1을 반환
    return -1;
}
```

<br>

### 3 - 1. LastindexOf(Object value) 메서드

indexOf 메서드는 index가 0부터 시작했다면 반대로 거꾸로 탐색하는 과정도 있는 것이 좀 더 좋다. 예를들어 사용자가 찾고자 하는 인덱스가 뒤 쪽이라고 예상 가능할 때 굳이 앞에서부터 찾아 줄 필요가 없기 때문이다.

```java
public int lastIndexOf(Object value) {
    for (int i = size - 1; i >= 0; i--) {
        if (array[i].equals(value)) {
            return i;
        }
    }
    return -1;
}
```

<br>

### 4. contains(Object value) 메서드

indexOf 메서드는 사용자가 찾고자 하는 요소(value)의 '위치(index)'를 반환하는 메서드였다면 contains는 사용자가 찾고자 하는 요소(value)가 존재 하는지 안하는지 반환하는 메서드다.

찾고자 하는 요소가 존재한다면 true를, 존재하지 않는다면 false를 반환한다.

```java
@Override
public boolean contains(Object value) {
    
    // 0 이상이면 요소가 존재한다는 뜻
    if (indexOf(value) >= 0) {
        return true;
    } else {
        return false;
    } 
}
```

해당 요소가 존재하는지를 '검사'한다는 기능은 indexOf와 비슷하기 때문에 사용하였다.

이렇게 서로 메서드들을 연결하여 사용하면 다른 사람이 코드를 보았을 때도 훨씬 이해가 잘 되고, 무엇보다 코드가 깔끔해진다.

<br><br>

# remove 메서드 구현

remove 메서드의 경우크게 2가지로 나눌 수 있다.

- 특정 index의 요소를 삭제 -remove(int index)
- 특정 요소를 삭제 - remove(Object value)

<br>

### 1. remove(int index) 메서드

remove(int index)는 '특정 위치에 있는 요소를 제거'하는 것이다.

앞서 add(int index, E value)를 했던 방식을 거꾸로 하면 된다는 의미다.

쉽게 생각해서 index에 위치한 데이터를 삭제해버리고, 해당 위치 이후의 데이터들을 한 칸씩 '당겨오는 것'이다. 

```java
@SuppressWarnings("unchecked")
@Override
public E remove(int index) {

    if (index >= size || index < 0) {
        throw new IndexOutOfBoundsException();
    }

    E element = (E) array[index]; // 삭제될 요소를 반환하기 위해 임시로 담아둠
    array[index] = null;

    // 삭제한 요소의 뒤에 있는 모든 요소들을 한 칸씩 당겨옴
    for (int i = index; i < size; i++) {
        array[i] = array[i + 1];
        array[i + 1] = null;
    }
    size--;
    resize();
    return element;
}
```

위와 마찬가지로 Object 타입을 E 타입으로 캐스팅을 해주면서 경고창이 뜬다. 하지만 get() 메서드에서설명했듯이 삭제되는 원소 또한 E Type외에 들어오는 것이 없기 때문에 형 안정성이 확보되므로 경고표시를 무시하기 위해 @SuppressWarnings("unchecked")을 붙인다.

그리고 명시적으로 요소를 null로 처리해주어야 가비지컬렉터에 의해 더이상 쓰지 않는 데이터의 메모리를 수거(반환)해주기 때문에 최대한 null 처리를 하는 것이 좋다.

(물론 명시적으로 안해도 크게 문제는 없지만 그럴경우 가비지컬렉터가 쓰지 않는 데이터라도 나중에 참조될 가능성이 있는 데이터로 볼 가능성이 높아진다. 이는 결국 메모리를 잡아먹을 수 있는 가능성이 있다는 것이고 결과적으로 프로그램 성능에도 영향을 미친다는 것이다.)

 만약 index가 정상적인 참조가 가능한 값일 경우 해당 인덱스의 요소를 반환해준다. 이 때 원본 데이터 타입으로 반환하기 위해 E 타입으로 캐스팅을 해준다.

<br>

### 2. remove(Object value) 메서드

remove(Object value) 메서드는 사용자가 원하는 특정요소(value)를 리스트에서 찾아서 삭제하는 것이다. IndexOf메서드와 마찬가지로 리스트안에 특정 요소와 매칭되는 요소가 여러개 있을 경우 가장 먼저 마주하는 요소만 삭제한다.

결과적으로 이 메서드에서 필요한 동작은 value와 같은 요소가 존재하는지, 만약에 존재한다면 몇 번째 위치에 존재하는지를 알아야하는 것 하나. 그리고 그 index의 데이터를 지우고 나머지 뒤의 요소들을 하나씩 당기는 작업 하나. 이렇게 총 두가지의 동작이 필요하다.

그러면 우리는 그동안 만들었던 indexOf() 메서드와 remove(int index) 메서드를 사용하면 될거 같다.

```java
@Override
public boolean remove(Object value) {
    
    // 삭제하고자 하는 요소의 인덱스 찾기
    int index = indexOf(value);
    
    // -1이라면 array에 요소가 없다는 의미이므로 false 반환
    if (index == -1) {
        return false;
    }
    
    // index 위치에 있는 요소를 삭제
    remove(index);
    return true;
}
```

<br><br>

# size isEmpty, clear 메서드 구현

<br>

### 1. size() 메서드

ArrayList에서 size 변수는 private 접근 제한자를 갖고 있기 때문에 외부에서 직접 참조할 수 없다. 그렇기 때문에 size 변수의 값을 반환해주는 메서드를 만들어줄 필요가 있다.

```java
@Override
public int size() {
    return size;
}
```

<br>

### 2. isEmpty() 메서드

```java
@Override
public boolean isEmpty() {
    return size == 0;
}
```

리스트가 비어있을 경우 true를 비어있지 않을 경우 false를 반환해주면 된다.

<br>

### 3. clear() 메서드

모든 요소들을 비워버리는 작업이다. 예를들어 리스트에 요소를 담아두었다가 초기화가 필요할 때 쓸 수 있는 유용한 존재다. 모든 요소를 비워버린다는 것은  size를 0으로 초기화해주고, 배열의 용적을 현재의 절반으로 줄일 수 있도록 해준다.

왜 초기 값이 아니라 절반이죠? 라고 궁금해 할수도 있다. 물론 초기값으로 초기화 해주어도 되나 생각해보면 현재 배열의 용적은 결국 데이터를 해당 용적에 만족하는 조건만큼 썻다는 의미가 된다.

즉, 현재 용적량의 기대치에 근접할 가능성이 높기 때문에 일단은 용적량을 일단 절반으로만 줄이는 것이다.

```java
@Override
public void clear() {
    // 모든 공간을 null 처리 해준다.
    for (int i = 0; i < size; i++) {
        array[i] = null;
    }
    size = 0;
    resize();
}
```

<br><br>

# 전체코드

```java
import Interface_form.List;

import java.util.Arrays;

public class ArrayList<E> implements List<E> {

    private static final int DEFAULT_CAPACITY = 10; //최소(기본) 용적 크기
    private static final Object[] EMPTY_ARRAY = {}; //빈 배열

    private int size;  // 요소 개수

    Object[] array; // 요소를 담을 배열

    // 생성자1 (초기 공간 할당 X)
    public ArrayList() {
        this.array = EMPTY_ARRAY;
        this.size = 0;
    }

    // 생성자2 (초기 공간 할당 O)
    public ArrayList(int capacity) {
        this.array = new Object[capacity];
        this.size = 0;
    }

    private void resize() {
        int array_capacity = array.length;

        // if array's capacity is 0
        if (Arrays.equals(array, EMPTY_ARRAY)) {
            array = new Object[DEFAULT_CAPACITY];
            return;
        }

        // 용량이 꽉 찰 경우
        if (size == array_capacity) {
            int new_capacity = array_capacity * 2;

            // copy
            array = Arrays.copyOf(array, new_capacity);
            return;
        }
        // 용적의 절반 미만으로 요소가 차지하고 있을 경우
        if (size < (array_capacity / 2)) {
            int new_capacity = array_capacity / 2;

            // copy
            array = Arrays.copyOf(array, Math.max(new_capacity, DEFAULT_CAPACITY));
            return;
        }
    }

    @Override
    public boolean add(E value) {
        addLast(value);
        return true;
    }

    public void addLast(E value) {
        // 꽉차있는 상태라면 용적 재할당
        if (size == array.length) {
            resize();
        }
        array[size] = value;
        size++; // 사이즈 1 증가
    }

    @Override
    public void add(int index, E value) {

        if (index > size || index < 0) {  // 영역을 벗어날 경우 예외 발생
            throw new IndexOutOfBoundsException();
        }
        if (index == size) { // index가 마지막 위치라면 addLast() 메서드로 요소 추가
            addLast(value);
        }
        else {

            if (size == array.length) { // 꽉차있다면 용적 재할당
                resize();
            }

            // index 기준 후자에 있는 모든 요소들 한 칸씩 뒤로 밀기
            for (int i = size; i > index; i--) {
                array[i] = array[i - 1];
            }

            array[index] = value;
            size++;
        }
    }

    public void addFirst(E value) {
        add(0, value);
    }

    @SuppressWarnings("unchecked")
    @Override
    public E remove(int index) {

        if (index >= size || index < 0) {
            throw new IndexOutOfBoundsException();
        }

        E element = (E) array[index]; // 삭제될 요소를 반환하기 위해 임시로 담아둠
        array[index] = null;

        // 삭제한 요소의 뒤에 있는 모든 요소들을 한 칸씩 당겨옴
        for (int i = index; i < size; i++) {
            array[i] = array[i + 1];
            array[i + 1] = null;
        }
        size--;
        resize();
        return element;
    }

    @Override
    public boolean remove(Object value) {

        // 삭제하고자 하는 요소의 인덱스 찾기
        int index = indexOf(value);

        // -1이라면 array에 요소가 없다는 의미이므로 false 반환
        if (index == -1) {
            return false;
        }

        // index 위치에 있는 요소를 삭제
        remove(index);
        return true;
    }

    @SuppressWarnings("unchecked")
    @Override
    public E get(int index) {
        if (index >= size || index < 0) { // 범위 벗어나면 예외 발생
            throw new IndexOutOfBoundsException();
        }
        // Object 타입에서 E타입으로 캐스팅 후 반환
        return (E) array[index];
    }

    @Override
    public void set(int index, E value) {
        if (index >= size || index < 0) { // 범위 벗어나면 예외 발생
            throw new IndexOutOfBoundsException();
        }
        else {
            // 해당 위치의 요소를 교체
            array[index] = value;
        }
    }

    @Override
    public boolean contains(Object value) {

        // 0 이상이면 요소가 존재한다는 뜻
        if (indexOf(value) >= 0) {
            return true;
        } else {
            return false;
        }
    }

    @Override
    public int indexOf(Object value) {
        int i = 0;

        // value와 같은 객체(요소 값)일 경우 i(위치) 반환
        for (i = 0; i < size; i++) {
            if (array[i].equals(value)) {
                return i;
            }
        }
        // 일치하는 것이 없을경우 -1을 반환
        return -1;
    }

    public int lastIndexOf(Object value) {
        for (int i = size - 1; i >= 0; i--) {
            if (array[i].equals(value)) {
                return i;
            }
        }
        return -1;
    }

    @Override
    public int size() {
        return size;
    }

    @Override
    public boolean isEmpty() {
        return size == 0;
    }

    @Override
    public void clear() {
        // 모든 공간을 null 처리 해준다.
        for (int i = 0; i < size; i++) {
            array[i] = null;
        }
        size = 0;
        resize();
    }
}
```