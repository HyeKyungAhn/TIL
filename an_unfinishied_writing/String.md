# String(Java)

## 불변
Java의 문자열 String은 불변의 속성을 가지고 있다. 이는 한 번 값을 저장하면 수정할 수 없다는 의미이다. 문자열은 얼핏보면 수정이 가능한 것 같다.

```
String name = "Eric";
name = "Dominic";

System.out.println(name); // Dominic
```

String은 클래스이다. 큰따옴표("")로 생성되는 String객체는 String constant pool(JAVA 1.7이후 Heap내에 위치)에 저장,관리된다. 위의 name 변수의 값이 수정되는 것 같지만 사실은 name이라는 ***변수가 저장하고 있는 참조(주소값)가*** "Eric"을 저장하고 있는 String 객체에서 "Dominic"을 저장한 객체의 주소로 ***변하는 것***일 뿐이다. 

왜 JAVA는 String 문자열을 String constant pool을 사용하여 관리하는 걸까?
- 메모리 절약
- 보안상 문제
- Thread-safe
- 문장려은 Hashcode를 생성 때부터 캐시 하기 때문에 HashMap의 key를 String으로 사용하면 빠른 속도로 사용할 수 있기 때문

## new연산자와 큰따옴표("")

String 클래스는 조금 독특한 클래스이다. 클래스를 생성할 때 new 연산자를 사용하기도 하지만, ""을 사용할 수도 있기 때문이다.

```
String name = new String("Hanna");
String sameName = "Hanna";
```

그런데 이렇게 생성한 두 객체는 조금 다르다. 
- new 연산자를 사용하여 생성한 String 객체는 Heap 메모리에
- 큰따옴표("")를 사용하여 생성한 객체는 Heap 메모리의 String constant pool에 저장된다

### new 연산자가 String 객체를 생성하는 과정
1. String constant pool에 같은 문자열을 저장한 객체가 있는지 확인
2. 있으면 Heap에만 String 객체를 생성하고 주소를 반한다.
3. 없다면 Heap과 String constant pool에 각각 String 객체를 생성하여 저장하고 그 주소를 반환한다.

### 큰따옴표로 String 객체를 생성하는 과정
1. String constant pool에 같은 문자열을 저장한 객체가 있는 지 확인한다
2. 있으면 그 객체의 주소를 반환
3. 없으면 새로운 객체를 생성하고 그 객체의 주소를 반환한다

### intern()
new 연산자로 생성된 String 객체가 pool내에 같은 값을 가진 객체(equal())가 있다면 그 주소값을 반환하는 메서드가 있다.(없다면 pool에 하나를 추가하고 그 주소값을 반환한다.)


# 참조
- https://readystory.tistory.com/139
- https://readystory.tistory.com/140