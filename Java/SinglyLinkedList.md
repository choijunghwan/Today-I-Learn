LinkedList의 한 종류인 Singly LinkedList를 구현해보겠습니다.

LinkedList의 경우 ArrayList와 가장 큰 차이점이라 한다면 바로 '노드'라는 객체를 이용하여 연결한다는 것이다. ArrayList의 경우 최상위 타입인 오브젝트 배열(Object[])을 사용하여 데이터를 담아두었다면, LinkedList는 배열을 이용하는 것이 아닌 하나의 객체를 두고 그 안에 데이터와 다른 노드를 가리키는 레퍼런스 데이터로 구성하여 여러 노드를 하나의 체인처럼 연결하는 것이다.

쉽게 배열과 LinkedList를 그림으로 비교해보자.

![image1](/Img/SinglyLinkedList/SinglyLinkedList1.png)

그림을 보면 LinkedList는 저장할 데이터 '3'과 reference 데이터(참조데이터) 'Node#1'의 데이터가 저장되어있다. reference 데이터는 다음에 연결할 노드를 가리키는 데이터가 담깁니다.

위와같은 노드들이 여러개가 연결 되어있는 것을 연결 리스트라고 하는데 이렇게 **단방향으로 연결 된 리스트**를 LinkedList 중에서도 Singly LinkedList라고 한다.

이렇게 연결된 노드들에서 '삽입'을 한다면 링크만 바꿔주면 되고, 반대로 '삭제'를 한다면 삭제 할 노드에 연결된 이전노드에서 링크를 끊고 삭제할 노드의 다음 노드를 연결해주면 된다.

<br><br>

# Node 구현

LinkedList를 구현하기 앞서 먼저 Node 클래스를 구현해주어야 한다. 

```java
public class Node<E> {

    E data;
    Node<E> next;  // 다음 노드객체를 가리키는 래퍼런스 변수

    Node(E data) {
        this.data = data;
        this.next = null;
    }
}
```

next가 앞서 그림에서 보여주었던 Reference 변수다. **다음 노드를 가리키는 변수**이며, **'노드 자체'를 가리키기 때문에** 타입 또한 Node<E> 타입으로 지정해주어야 한다.

그럼 이제 위 노드를 이용하기 위한 단일 연결리스트(Singly LinkedList)를 구현해보자.

<br><br>

# Singly LinkedList 클래스 및 생성자 구성하기

Singly LinkedList 이름은 너무 기니 SLinkedList로 이름을 대체하겠다.

```java
import Interface_form.List;

public class SLinkedList<E> implements List<E> {

    private Node<E> head;   // 노드의 첫 부분
    private Node<E> tail;   // 노드의 마지막 부분
    private int size;       // 요소 개수

    public SLinkedList() {
        this.head = null;
        this.tail = null;
        this.size = 0;
    }
}
```

일단 사전에 만들어두었던 List 인터페이스 implements 해준다.

`Node<E> head` : 리스트의 가장 첫 노드를 가리키는 변수다.

`Node<E> tail` : 리스트의 가장 마지막 노드를 가리키는 변수다.

`size` : 리스트에 있는 요소의 개숫

처음 단일 연결리스트를 생성 할 때에는 아무런 데이터가 없으므로 당연히 head와 tail이 가리킬 노드가 없기에 null로 초기화 및 size는 0으로 초기화 해주도록 한다.

<br><br>

# search 메소드 구현

SLinkedList 구현에 앞서 search()메소드를 작성하려 한다. 이유는 단일 연결리스트이다보니 특정 위치의 데이터를 삽입, 삭제, 검색하기 위해서는 처음 노드(head)부터 next변수를 통해 특정 위치까지 찾아가야 하기 때문이다.

```java
// 특정 위치의 노드를 반환하는 메소드
private Node<E> search(int index) {

    // 범위 밖(잘못된 위치)일 경우 예외 던지기
    if (index < 0 || index >= size) {
        throw new IndexOutOfBoundsException();
    }

    Node<E> x = head;  // head가 가리키는 노드부터 시작

    for (int i = 0; i < index; i++) {
        x = x.next;  // x노드의 다음 노드를 x에 저장한다.
    }
    return x;
}
```

<br><br>

# add 메소드 구현

add 메소드에는 ArrayList와 마찬가지로 크게 3가지로 분류가 된다.

- 가장 앞부분에 추가 - addFirst(E value)
- 가장 마지막 부분에 추가 (기본값) - addLast(E value)
- 특정 위치에 추가 - add(int index, E value)

<br>

### 1. addFirst(E value)

먼저 addFirst()를 구현해보자.

연결리스트는 '링크'로 연결 된 리스트다. 즉, 가장 앞에 추가한다면 다음과 같은 과정을 거친다.

![image2](/Img/SinglyLinkedList/SinglyLinkedList2.png)

데이터를 이동시키는 것이 아닌 새로운 노드를 생성하고 새 노드의 레퍼런스 변수(next)가 head 노드를 가리키도록 해주면 된다.

```java
public void addFirst(E value) {

    Node<E> newNode = new Node<E>(value); // 새 노드 생성
    newNode.next = head;
    head = newNode;
    size++;

    /**
     * 다음에 가리킬 노드가 없는 경우
     * 데이터가 한개 밖에 없으므로 새 노드는 처음 시작노드이자 마지막 노드다.
     * 즉 tail = head다.
     */
    if (head.next == null) {
        tail = head;
    }
}
```

위 그림처럼 새 노드를 하나 만들어준 다음 '가장 앞에 추가'해야 하므로 기존에 있던 head가 가리키는 노드앞에 존재해야 한다는 것이다.

즉, 새로운 노드의 next가 다음 노드인 head가 되고 head가 가리키는 첫번째 노드를 새로운 노드로 변경해주고 사이즈를 1 증가시키면 된다.

그리고 예외적으로 아무런 노드가 없는 상태에서 처음으로 추가하는 노드일경우 결국 head가 가리키는 노드는 없다. 이는 head 노드가 처음이자 마지막 노드가된다는 말이기 때문에 마지막을 가리키는 변수 tail은 곧 head와 같은 노드를 가리키게 된다.

<br>

### 2. add(E value) & addLast(E value) 메소드

LinkedList API를 보면 add메소드를 호출하면 add() 메소드는 addLast()를 호출한다.

그리고 size가 0일 경우(=아무런 노드가 없는 경우) 결국 처음으로 데이터를 추가한다는 뜻이기 때문에 addFirst()를 호출해주면 된다.

```java
@Override
public boolean add(E value) {
    addLast(value);
    return true;
}

public void addLast(E value) {
    Node<E> newNode = new Node<E>(value);

    if (size == 0) {
        addFirst(value);
        return;
    }

    tail.next = newNode;
    tail = newNode;
    size++;
}
```

기존에 있던 tail 노드가 다음 노드를 가리키는 변수(next)를 새 노드를 가리키도록 변경해주고 tail이 가리키는 노드를 새로운 노드로 변경만 해주면 된다.

<br>

### 3. add(int index, E value)

![image3](/Img/SinglyLinkedList/SinglyLinkedList3.png)

먼저 넣으려는 위치(예시에서는 index=3)의 노드와 이전의 노드를 찾아야 한다.

넣으려는 위치의 이전노드를 prev_Node라고 하고, 넣으려는 위치의 기존노드를 next_Node라고 할 때, 우리가 만든 메소드 search()를 사용하여 위치를 찾는다.

그리고 prev_Node의 링크를 새로 추가하려는 노드로 변경하고, 새로 추가하려는 노드의 링크는 next_Node로 변경해주는 것이다. 다만 index 변수가 잘못된 위치를 참조할 수 있으니 이에 대한 예외처리로 IndexOutOfBoundsException을 한다.

```java
@Override
public void add(int index, E value) {
    
    // 잘못된 인덱스를 참조할 경우 예외 발생
    if (index > size || index < 0) {
        throw new IndexOutOfBoundsException();
    }
    // 추가하려는 index가 가장 앞에 추가하려는 경우 addFirst 호출
    if (index == 0) {
        addFirst(value);
        return;
    }
    // 추가하려는 index가 마지막 위치일 경우 addLast 호출
    if (index == size) {
        addLast(value);
        return;
    }
    
    // 추가하려는 위치의 이전 노드
    Node<E> prev_Node = search(index - 1);
    
    // 추가하려는 위치의 노드
    Node<E> next_Node = prev_Node.next;
    
    // 추가하려는 노드
    Node<E> newNode = new Node<E>(value);

    /**
     * 이전 노드가 가리키는 노드를 끊은 뒤
     * 새 노드로 변경해준다.
     * 또한 새 노드가 가리키는 노드는 next_Node로 설정해준다.
     */
    prev_Node.next = null;
    prev_Node.next = newNode;
    newNode.next = next_Node;
    size++;
}
```

<br><br>

# remove 메소드 구현

remove() 메소드는 add() 메소드의 메커니즘을 반대로 생각하면 된다.

remove 메소드도 크게 3가지로 나눌 수 있다.

- 가장 앞의 요소(head)를 삭제 - remove()
- 특정 index의 요소를 삭제 - remove(int index)
- 특정 요소를 삭제 - remove(Object value)

기본적으로 삭제연산의 가장기초는 remove() 메소드로 head가 가리키는 요소, 즉 첫 번째 요소를 삭제하는 것이다. 인덱스로 생각한다면 0 위치에 있는 요소를 말한다. 그리고 remove() 메소들을 구현할 때 자칫 null을 참조하거나 잘못된 참조를 하는 경우도 있으니 조심해야 한다.

<br>

### 1. remove() 메소드

remove()는 '**가장 앞에 있는 요소**'를 제거하는 것이다. 즉, head가 가리키는 요소만 없애주면 된다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c12c3512-bedd-4b54-9ac0-306aeee4bb80/Untitled.png)

```java
public E remove() {

    Node<E> headNode = head;

    if (headNode == null) {
        throw new NoSuchElementException();
    }

    // 삭제된 노드를 반환하기 위한 임시 변수
    E element = headNode.data;
    
    // head의 다음 노드
    Node<E> nextNode = head.next;
    
    // head 노드의 데이터들을 모두 삭제
    head.data = null;
    head.next = null;
    
    // head 가 다음 노드를 가리키도록 업데이트
    head = nextNode;
    size--;

    /**
     * 삭제된 요소가 리스트의 유일한 요소였을 경우
     * 그 요소는 head 이자 tail이었으므로
     * 삭제되면서 tail도 null로 변환
     */
    if (size == 0) {
        tail = null;
    }
    return element;
}
```

<br>

### 2. remove(int index) 메소드

remove(int index) 메소드는 사용자가 원하는 특정 위치(index)를 리스트에서 찾아서 삭제하는 것이다. add(int index, E value)와 정반대로 구현해주면 된다.

```java
@Override
public E remove(int index) {
    
    // 삭제하려는 노드가 첫 번째 원소일 경우
    if (index == 0) {
        return remove();
    }
    
    // 잘못된 범위에 대한 예외
    if (index >= size || index < 0) {
        throw new IndexOutOfBoundsException();
    }

    Node<E> prevNode = search(index - 1); // 삭제할 노드의 이전 노드
    Node<E> removedNode = prevNode.next;       // 삭제할 노드
    Node<E> nextNode = removedNode.next;       // 삭제할 노드의 다음 노드

    E element = removedNode.data;
    
    // 이전 노드가 가리키는 노드를 삭제하려는 노드의 다음노드로 변경
    prevNode.next = nextNode;
    
    // 데이터 삭제
    removedNode.next = null;
    removedNode.data = null;
    size--;
    
    return element;
}
```

<br>

### 3. remove(Object value) 메소드

remove(Object value) 메소드는 사용자가 원하는 특정 요소(value)를 리스트에서 찾아서 삭제하는 것이다.

remove(int index) 메소드하고 동일한 메커니즘으로 작동한다. 다만 고려해야 할 점은 '삭제하려는 요소가 존재하는지'를 염두해두어야 한다. 예를들어 삭제하려는 요소를 찾지 못했을 경우 false를 반환해주고, 만약 찾았을 경우 remove(int index)와 동일한 삭제 방식을 사용하면 된다.

![image4](/Img/SinglyLinkedList/SinglyLinkedList4.png)

```java
@Override
public boolean remove(Object value) {
    
    Node<E> prevNode = head;
    boolean hasValue = false;
    Node<E> x = head;  // removeNode
    
    // value 와 일치하는 노드를 찾는다.
    for (; x != null; x = x.next) {
        if (value.equals(x.data)) {
            hasValue = true;
            break;
        }
        prevNode = x;
    }
    
    // 일치하는 요소가 없을 경우 false 반환
    if (x == null) {
        return false;
    }
    
    // 만약 삭제하려는 노드가 head라면 기존 remove()를 사용
    if (x.equals(head)) {
        remove();
        return true;
    }
    else {
        // 이전 노드의 링크를 삭제하려는 노드의 다음 노드로 연결
        prevNode.next = x.next;
        
        x.data = null;
        x.next = null;
        size--;
        return true;
    }
}
```

<br><br>

# get, set, indexOf, contains 메소드 구현

추가와 삭제 메소드를 구현했으면 거의 다 했다고 봐도 무방하다.

이제 나머지 메소드를 구현해보자.

<br>

### 1. get(int index) 메소드

get(int index) 메소드는 해당 위치에 있는 요소를 반환하는 메소드다. 우리는 앞서 search() 메소드를 구현해놓았기 때문에 이걸 이용하면 된다.

```java
@Override
public E get(int index) {
    return search(index).data;
}
```

search() 내부에서 잘못된 위치일 경우 예외를 던지기 때문에 따로 예외처리를 해줄 필요는 없다.

<br>

### 2. set(int index, E value) 메소드

set 메소드는 기존의 index에 위치한 데이터를 새로운 데이터(value)으로 '**교체**'하는 것이다. 이것도 search() 메소드를 이용해 쉽게 구현할 수 있다.

```java
@Override
public void set(int index, E value) {
    Node<E> replaceNode = search(index);
    replaceNode.data = value;
}
```

<br>

### 3. indexOf(Object value) 메소드

indexOf 메소드는 사용자가 찾고자 하는 요소(value)의 '위치(index)'를 반환하는 메소드다.

주의해할점은 "찾고자 하는 요소가 중복된다면??" 에는 **가장 먼저 마주치는 요소의 인덱스를 반환**한다는 것이다.

만약 "찾고자 하는 요소가 없다면??" **-1을 반환**한다.

```java
@Override
public int indexOf(Object value) {
    int index = 0;

    for (Node<E> x = head; x != null; x = x.next) {
        if (value.equals(x.data)) {
            return index;
        }
        index++;
    }
    // 찾고자 하는 요소를 찾지 못했을 경우 -1 반환
    return -1;
}
```

<br>

### 4. contains(Object value) 메소드

contains는 사용자가 찾고자 하는 요소(value)가 존재 하는지 안하는지를 반환하는 메소드이다.

그러므로 indexOf를 이용하여 **만약 음수가 아닌 수가 반환되었다면 요소가 존재**한다는 뜻이고, **음수(-1)이 나왔다면 요소가 존재하지 않는다는 뜻**이다.

 

```java
@Override
public boolean contains(Object value) {
    return indexOf(value) >= 0;
}
```

<br><br>

# size, isEmpty, clear 메소드 구현

이제 나머지 메소드를 구현해보자. 이부분은 간단하니 휙휙 넘어가겠다.

<br>

### 1. size() 메소드

```java
@Override
public int size() {
    return size;
}
```

size() 메소드를 구현하는 이유는 size 변수는 private로 외부에서 접근할 수 없기 때문에 값을 반환해주는 메소드를 따로 구현해준다.

<br>

### 2. isEmpty() 메소드

ArrayList에 요소가 존재하는지 안하는지를 알려주는 메소드이ㅏㄷ.

```java
@Override
public boolean isEmpty() {
    return size == 0;
}
```

<br>

### 3. clear() 메소드

clear() 메소드는 리스트에 있는 요소를 모두 초기화 할때 유용하게 쓰이는 메소드이다.

요소를 비워버린다는것은 size또한 0으로 초기화 해준다는 뜻이다.

```java
@Override
public void clear() {
    for (Node<E> x = head; x != null; ) {
        Node<E> nextNode = x.next;
        x.data = null;
        x.next = null;
        x = nextNode;
    }
    head = tail = null;
    size = 0;
}
```

그냥 객체 자체를 null 해주기 보다는 모든 노드를 하나하나 null 해주는 것이 자바의 가비지 컬렉터가 명시적으로 해당 메모리를 안쓴다고 인지하기 때문에 메모리 관리 효율 측면에서 조금이나마 더 좋다.

<br><br>

# 전체 코드

```java
import Interface_form.List;

import java.util.NoSuchElementException;

public class SLinkedList<E> implements List<E> {

    private Node<E> head;   // 노드의 첫 부분
    private Node<E> tail;   // 노드의 마지막 부분
    private int size;       // 요소 개수

    public SLinkedList() {
        this.head = null;
        this.tail = null;
        this.size = 0;
    }

    // 특정 위치의 노드를 반환하는 메소드
    private Node<E> search(int index) {

        // 범위 밖(잘못된 위치)일 경우 예외 던지기
        if (index < 0 || index >= size) {
            throw new IndexOutOfBoundsException();
        }

        Node<E> x = head;  // head가 가리키는 노드부터 시작

        for (int i = 0; i < index; i++) {
            x = x.next;  // x노드의 다음 노드를 x에 저장한다.
        }
        return x;
    }

    public void addFirst(E value) {

        Node<E> newNode = new Node<E>(value); // 새 노드 생성
        newNode.next = head;
        head = newNode;
        size++;

        /**
         * 다음에 가리킬 노드가 없는 경우
         * 데이터가 한개 밖에 없으므로 새 노드는 처음 시작노드이자 마지막 노드ㅏ.
         * 즉 tail = head다.
         */
        if (head.next == null) {
            tail = head;
        }
    }

    @Override
    public boolean add(E value) {
        addLast(value);
        return true;
    }

    public void addLast(E value) {
        Node<E> newNode = new Node<E>(value);

        if (size == 0) {
            addFirst(value);
            return;
        }

        tail.next = newNode;
        tail = newNode;
        size++;
    }

    @Override
    public void add(int index, E value) {

        // 잘못된 인덱스를 참조할 경우 예외 발생
        if (index > size || index < 0) {
            throw new IndexOutOfBoundsException();
        }
        // 추가하려는 index가 가장 앞에 추가하려는 경우 addFirst 호출
        if (index == 0) {
            addFirst(value);
            return;
        }
        // 추가하려는 index가 마지막 위치일 경우 addLast 호출
        if (index == size) {
            addLast(value);
            return;
        }

        // 추가하려는 위치의 이전 노드
        Node<E> prev_Node = search(index - 1);

        // 추가하려는 위치의 노드
        Node<E> next_Node = prev_Node.next;

        // 추가하려는 노드
        Node<E> newNode = new Node<E>(value);

        /**
         * 이전 노드가 가리키는 노드를 끊은 뒤
         * 새 노드로 변경해준다.
         * 또한 새 노드가 가리키는 노드는 next_Node로 설정해준다.
         */
        prev_Node.next = null;
        prev_Node.next = newNode;
        newNode.next = next_Node;
        size++;
    }

    public E remove() {

        Node<E> headNode = head;

        if (headNode == null) {
            throw new NoSuchElementException();
        }

        // 삭제된 노드를 반환하기 위한 임시 변수
        E element = headNode.data;

        // head의 다음 노드
        Node<E> nextNode = head.next;

        // head 노드의 데이터들을 모두 삭제
        head.data = null;
        head.next = null;

        // head 가 다음 노드를 가리키도록 업데이트
        head = nextNode;
        size--;

        /**
         * 삭제된 요소가 리스트의 유일한 요소였을 경우
         * 그 요소는 head 이자 tail이었으므로
         * 삭제되면서 tail도 null로 변환
         */
        if (size == 0) {
            tail = null;
        }
        return element;
    }

    @Override
    public E remove(int index) {

        // 삭제하려는 노드가 첫 번째 원소일 경우
        if (index == 0) {
            return remove();
        }

        // 잘못된 범위에 대한 예외
        if (index >= size || index < 0) {
            throw new IndexOutOfBoundsException();
        }

        Node<E> prevNode = search(index - 1); // 삭제할 노드의 이전 노드
        Node<E> removedNode = prevNode.next;       // 삭제할 노드
        Node<E> nextNode = removedNode.next;       // 삭제할 노드의 다음 노드

        E element = removedNode.data;

        // 이전 노드가 가리키는 노드를 삭제하려는 노드의 다음노드로 변경
        prevNode.next = nextNode;

        // 데이터 삭제
        removedNode.next = null;
        removedNode.data = null;
        size--;

        return element;
    }

    @Override
    public boolean remove(Object value) {

        Node<E> prevNode = head;
        boolean hasValue = false;
        Node<E> x = head;  // removeNode

        // value 와 일치하는 노드를 찾는다.
        for (; x != null; x = x.next) {
            if (value.equals(x.data)) {
                hasValue = true;
                break;
            }
            prevNode = x;
        }

        // 일치하는 요소가 없을 경우 false 반환
        if (x == null) {
            return false;
        }

        // 만약 삭제하려는 노드가 head라면 기존 remove()를 사용
        if (x.equals(head)) {
            remove();
            return true;
        }
        else {
            // 이전 노드의 링크를 삭제하려는 노드의 다음 노드로 연결
            prevNode.next = x.next;

            x.data = null;
            x.next = null;
            size--;
            return true;
        }
    }

    @Override
    public E get(int index) {
        return search(index).data;
    }

    @Override
    public void set(int index, E value) {
        Node<E> replaceNode = search(index);
        replaceNode.data = value;
    }

    @Override
    public boolean contains(Object value) {
        return indexOf(value) >= 0;
    }

    @Override
    public int indexOf(Object value) {
        int index = 0;

        for (Node<E> x = head; x != null; x = x.next) {
            if (value.equals(x.data)) {
                return index;
            }
            index++;
        }
        // 찾고자 하는 요소를 찾지 못했을 경우 -1 반환
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
        for (Node<E> x = head; x != null; ) {
            Node<E> nextNode = x.next;
            x.data = null;
            x.next = null;
            x = nextNode;
        }
        head = tail = null;
        size = 0;
    }
}
```

<br>

참고자료 : https://st-lab.tistory.com/167