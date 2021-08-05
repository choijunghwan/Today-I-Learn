# [ 자바에서 객체가 동일한지 확인하는 방법 ]

== 연산자

- **primitive type(int, double, float, boolean)**일 때는 값이 같은지 비교
- **reference type** 객체일때는 주소가 같은지 검사한다.

<br>

equals()

- 2개의 객체가 동일한지 검사하기 위해 사용된다.
- 2개의 객체가 가리키는 곳이 동일한 메모리 주소일 경우에만 동일한 객체가 된다.

<br>

**==연산자와 equals() 메서드의 가장 큰 차이점은** 

==연산자는 비교하고자 하는 두개의 대상의 주소값을 비교하는데 반해 equals() 메서드는 비교하고자 하는 두개의 대상의 값 자체를 비교합니다.

<br>


# [ 자바 메모리 구조 ]

간단하게 Static 영역, Stack 영역, Heap 영역으로 나눠서 알아보겠습니다.

## Static 영역

- 패키지나 클래스 정보가 올라갑니다.
- static이라는 키워드를 붙여서 선언된 필드, 메서드도 static 영역에 올라갑니다.
- static 영역에 자리잡게 되면 JVM이 종료 될 때까지 사라지지 않고, 고정된 상태로 유지됩니다.

## Stack 영역

- 메서드가 호출될때마다 스택프레임이 생기게됩니다.
- 스택영역은 FILO(First In Last Out)구조를 사용하고 있습니다.
- 원시 타입(primitive type)은 값을 저장하고, 참조 타입(reference type)은 객체의 참조값(주소값) 만을 저장하고 실제 값은 힙영역, static 영역에 저장됩니다.

## Heap 영역

- new 연산자로 생성된 객체 또는 객체(인스턴스)와 배열을 저장합니다.
- 힙 영역에 생성된 객체와 배열은 스택 영역의 변수나 다른 객체의 필드에서 참조합니다.

[자바(JVM)의 메모리 사용 방식 (T 메모리 구조)](https://siyoon210.tistory.com/124)

# [ Code ]

```java
// Primitive Type(원시 타입)
int a = 10;
int b = 10;
System.out.println(a == b);  //true

// Reference Type
String str1 = "hello";
String str2 = "hello";
System.out.println(str1 == str2); //true
```

<br><br><br>

```java
int a = 10;
int b = 10;
System.out.println(a.equals(b));  // error
```

위의 경우 true or false가 아닌 error가 발생하게 됩니다.

equals는 메서드로써 객체나 reference type만 비교가 가능합니다.

<br>

만약 int값을 비교하고 싶으시다면

```java
Integer a = 10;
Integer b = 10;
System.out.println(a.equals(b));  //true
```
<br>

Integer 타입으로 선언하여 비교하는 방법도 있습니다.

```java
String str1 = "hello";
String str2 = "hello";
String str3 = new String("hello");
String str4 = new String("hello");

System.out.println(str1 == str2);      //true
System.out.println(str1 == str3);      //false
System.out.println(str3 == str4);      //false
System.out.println(str1.equals(str2)); //true
System.out.println(str1.equals(str3)); //true
System.out.println(str3.equals(str4)); //true
```

다들 예상한대로 결과값이 나왔나요??

그림으로 바로 살펴보고 설명을 하겠습니다.

<br><br><br>


뭔가 이상한게 있을 것입니다. 모두 String으로 생성하였는데 str1과 str2는 같은 주소값을 가지고 str3와 str4는 다른 주소값을 갖습니다.

그 이유는 String은 생성방법차이에 있습니다.

String은 생성방법이 두가지가 있습니다.

1. 리터럴을 이용한 방식
2. new 연산자를 이용한 방식입니다.

간단하게 말하면 리터럴을 이용한 방식은 "hello" 값을 저장할때 이미 저장된 값이 있다면 추가로 저장하지 않고 이미 저장되어 있는 "hello"의 주소값을 참조합니다.

하지만 new 연산자를 이용한 방식은 "hello"가 저장되어 있는지 여부와 상관없이 새롭게 저장됩니다.

<br>

그럼 이제 다시 코드를 봐보겠습니다.

**== 연산자**는 **Primitive Type이면 값을 비교**하고 **reference Type이면 주소값을 비교**한다고 하였습니다.

그래서 **str1**과 **str2**는 같은 주소값 **500**을 가지므로 `True`가 나오지만 **str3,str4**는 각각 주소값이 **600**, **700** 이므로 결과가 `False`가 나옵니다.

허나 equals() 메서드는 대상의 값 자체를 비교하기 때문에 str1,st2,str3,str4 모두 `"hello"` 같은 값을 가지고 있으므로 True가 나옵니다.

> Reference Type을 비교할때는 ==연산자보다는 equals() 메서드를 사용하는게 좋습니다.

<br><br>

# [ equals() 와 hashCode() ]

equals() 메서드는 객체의 값이 같은지 확인합니다.

그럼 `Person`이라는 클래스가 존재합니다.

```java
class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

Person 객체 p1,p2를 equals로 비교해보겠습니다.

```java
Person p1 = new Person("hello", 20);
Person p2 = new Person("hello", 20);

System.out.println(p1.equals(p2));  //false
```

<br>

## 왜 결과가 false 일까요??

equals 메서드를 확인해보면 알수 있습니다.

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

Object의 ==로 비교하는 것을 볼 수 있습니다.

<br>

## 어떻게 equals 비교를 할 수 있을까요?

equasl()를 오버라이드해서 구현해주면 됩니다.

오바라이드 해줄때 동일하다고 판단할 기준을 정해줄 수 있습니다.

name만 같으면 같은 객체로 판단할지, age만 같으면 같은 객체로 판단하질 정할 수 있는데

저는 name, age 둘다 같아야 같은 객체로 판단하겠습니다.

```java
@Override
public boolean equals(Object obj) {

    if (obj == null) {
        return false;
    }

    if (this.getClass() != obj.getClass()) {
        return false;
    }

    Person p = (Person) obj;
    if (this.age == p.age && this.name.equals(p.name)) {
        return true;
    }

    return false;
}
```

<br>

## 결과

```java
Person p1 = new Person("hello", 20);
Person p2 = new Person("hello", 20);

System.out.println(p1.equals(p2));   //true
```

<br>

## 그럼 hashcode()는 왜 필요할까요??

```java
Map<Person, Integer> map = new HashMap<>();
map.put(p1, 1);
map.put(p2, 1);
System.out.println(map.size());   // 2
```

저희는 p1과 p2가 동일한 Key로 인식되어 size 값이 1이 나오길 기대했는데요.

결과는 size값이 2가 나왔습니다.

<br>

## 왜 크기가 2일까요?

Hash를 사용한 Collection 자료구조는 key를 결정할때 hashcode()를 사용하기 때문입니다.

즉 hashcode() 값이 동일하면 동일한 key로 인식하게 할 수 있습니다.

<br>

## hashcode() 구현

hashcode()를 오버라이드하면 문제를 해결할 수 있습니다.

```java
@Override
public int hashCode() {
    final int prime = 31;
    int hashCode = 1;

    hashCode = prime * hashCode + ((name == null) ? 0 : name.hashCode());
    hashCode = prime * hashCode + age;

    return hashCode;
}
```

<br>

## 결과

```java
Map<Person, Integer> map = new HashMap<>();
map.put(p1, 1);
map.put(p2, 1);
System.out.println(map.size());   // 1
```
<br>

## 어떻게 hashCode()는 결정되는걸까요??

**Object의 hashCode()**

```java
public native int hashCode();
```

hashCode()로 native call을 하여 Memory에서 가진 해쉬 주소값을 출력합니다.

**String의 hashCode()**

java 11버전부터 StringLatin1과 StringUTF16 두가지로 나뉘어져있는데요

StringUTF16 버전 코드를 보겠습니다.

```java
public static int hashCode(byte[] value) {
    int h = 0;
    int length = value.length >> 1;
    for (int i = 0; i < length; i++) {
        h = 31 * h + getChar(value, i);
    }
    return h;
}
```

보시면 31일 이라는 숫자가 눈에 보입니다.

### 왜 31을 사용할가요??

31은 소수이면서 홀수이기 때문에 선택된 값입니다.

소수는 1과 자기 자신을 제외한 숫자로는 나누는게 불가능하기 때문에 hash값을 생성할때 충돌이 가장 적은 숫자입니다.

또한 31이라는 숫자는 32에서 1을 뺀 숫자인데요. 

예를들어 5 * 31 = (5 * 32) - 5 와 같습니다. 32라는 숫자는 2^5으로 비트를 왼쪽으로 5번 shift 한것과 같습니다.

즉 5 * 31 = (5 << 5) - 5 로 shift 연산을 이용해 할 수 있는데요. 이러한 방식이 컴퓨가 계산할때 더 좋은 성능을 낼 수 있어 사용한다고 합니다.

> 객체에 equals()를 재정의한다면 hashCode()도 같이 재정의해주는것이 좋습니다.