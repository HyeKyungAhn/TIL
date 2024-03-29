# 객체지향적 프로그래밍

## 상속
### 상속이란?
기존의 클래스를 재사용하여 새로운 클래스를 작성하는 것

### 사용
```
class Child extends Parent{}
```
### 상속의 특징
- 조상, 자손 클래스
- 자손 클래스는 조상의 모든 멤버(생성자, 초기화 블록 제외)를 상속받음
- 조상 클래스가 변경되면 자손이 영향을 받음
- 자손 클래스의 멤버는 조상보다 같거나 많다
- 클래스간의 관계에서 형제 관계는 없다. 상속 관계만 있을 뿐.
- 자손 클래스의 인스턴스 생성 시, 조상과 자손의 멤버가 합쳐진 하나의 인스턴스로 생성됨

### 상속의 장점
- 클래스를 재사용하기 떄문에 작성해야하는 코드의 수가 줄어든다
- 중복 코드가 줄어든다
    - 같은 내용의 코드를 하나 이상의 코드에 중복 추가해야하는 경우, 상속으로 중복 제거
- 기존 클래스를 변경하면 새로운 클래스도 변경되기 때문에 추가 및 변경에 용이하다(유지보수, 생산성)
    - 하지만 기존 클래스의 변수와 메서드는 이를 상속받는 모든 클래스에서도 사용되기 떄문에 매우 잘 작성해야함
<details>
<summary>상속과 메서드의 차이</summary>
<div markdown = 1>

상속과 메서드의 작성의 의도를 헷갈릴 때가 있다. 큰 규모의 프로젝트를 안해봐서 그런듯.

### 메서드

- 메서드는 반복되는 코드를 하나로 묶어 다루는 것이다. 마치 블랙박스처럼 안에가 어떻게 구현되어있는지를 몰라도 이 메서드의 기능과 입력해야하는 입력값만 넣으면 원하는 결과값을 받아볼 수 있다.
- 메서드 작성을 통해 구조화된 코드를 작성할 수 있다
- 또 중복 코드를 제거할 수 있다. 같은 코드를 다른 곳에 똑같이 작성하는 대신, 메서드를 호출하기만 하면 된다.
- 메서드는 재사용성이 높다. 다른 말로 하면, 여러번 사용될 것같은 코드만 메서드로 작성한다는 의미이다. 내가 작성한 메서드를 API로 만들어 놓으면 다른 누군가가 다른 프로젝트에서도 사용할 수 있다
### 상속
- 상속은 클래스의 재사용이다
- 메서드와 달리 상속은 기존 코드와 재사용하는 코드 간에 상속'관계'가 생긴다
- 클래스는 '확장'된다
    - 뼈에 계속 살을 덧붙이는 식으로 확장된다.
- 조상의 변화가 자손에게 영향을 주지만, 그 반대로는 하지 못한다. 마치 유산과도 같은것..
- 단순히 코드의 중복을 방지하기 위해 상속을 해서는 안된다. 확장될 가능성이 있는 클래스인지, 확장 될거라면 무엇이 공통된 코드가 되어야하는지를 고려해서 상속해야한다.
</details>

### 포함관계
- 한 클래스의 멤버변수로 다른 클래스 타입의 참조변수를 선언하는 것

### 포함관계, 상속관계 결정하기
- is/has

### 단일 상속
- 자바에서는 오직 단일 상속만 허용
- 다중 상속 시 시그니쳐가 같은 메서드를 구분할 수 없어 충돌 발생
- 다중 상속을 구현해야할 경우, 비중이 높은 클래스 1개를 상속, 나머지는 포함관계로(인터페이스를 활용하면 다형성 활용 가능)

### Object 클래스 - 모든 클래스의 조상
- 모든 클래스 상속 계층도의 최상위의 조상 클래스
- 다른 클래스의 상속을 받지 않는 클래스는 Object를 상속(컴파일러가 붙여줌)

## 오버라이딩
### 오버라이딩이란
- 상속받은 메서드의 내용을 재작성하는 것
- 자손 클래스 자신에 맞게 변경해야하는 경우에 오버라이딩

### 오버라이딩 조건
- 이름 동일
- 매개변수의 갯수, 타입 동일
- 반환타입 동일(JDK1.5~ 자손클래스 타입으로 변경 가능)
- 접근 제어자는 같거나 넓은 범위로 변경 가능
- 더 많은 수의 예외를 선언할 수 었다(갯수가 아닌 범위)
- 인스턴스 메서드를 static으로, 또는 그 반대로 변경 불가
- static 메서드 오버라이딩 가능하나 불가. 오버라이딩이 아닌 클래스의 새로운 static 메서드를 작성하는 것(그래서 클래스이름.메서드이름 으로 접근함)

### 오버라이딩 vs 오버로딩
- 오버로딩: 기존에 없던 메서드를 새로 작성
- 오버라이딩: 상속받은 메서드의 내용을 변경

### super
- 자손클래스에서 조상클래스로부터 상속받은 멤버를 가리킬 때 사용하는 참조변수
- 자손에서 오버라이딩 했을 떄 구별을 위해 사용
- 모든 인스턴스 멤버에는 자신이 속한 인스턴스의 주소가 지역변수로 저장(this, super)
- 조상 클래스의 메서드에 내용을 '추가'(완전 새로 작성x)하는 경우에 super를 사용하여 조상클래스의 메서드를 포함하는 것이 좋다.(조상의 변경이 자동 반영되니까)

### super() - 조상클래스의 생성자
- 조상 클래스의 생성자를 호출하는데 사용
- 초기화 작업이 목적
- Object 클래스를 제외한 모든 클래스의 생성자 첫 줄에 생성자 this() 또는 super()가 호출되어야한다. 그렇지 않으면 컴파일러가 super()를 생성자의 첫줄에 삽입
    - 자손 클래스의 멤버에서 조상의 멤버를 사용할 수도 있기에 먼저 초기화되어야한다
- 조상클래스의 멤버는 조상클래스의 생성자로 초기화되어야한다

## package와 import
### 패키지
- 관련있는 클래스들의 묶음
- 클래스의 실제 이름은 패키지를 포함한 것
- 클래스가 물리적으로 하나의 클래스 파일, 패키지는 물리적으로 하나의 디렉터리
- 하나의 소스파일에는 첫 문장으로 단 한 번의 패키지 선언만 가능
- 모든 클래스는 반드시 하나의 패키지에 속해야
- 패키지는 점(.)을 구분자로 계층구조로 구성 가능

### 패키지의 선언
```package 패키지명;```

### import문
- 소스파일에서 사용된 클래스 이름에서 패키지 이름을 생략하기 위해 사용
- 컴파일러에 import문의 패키지에 대한 정보를 제공하는 것

### import문의 선언
```
1. package문
2. import문
3. 클래스 선언
```
- *의 사용
    - 패키지에 속하는 모든 클래스를 패키지 명 없이 사용할 수 있다
    - 컴파일러가 해당 패키지에서 일치하는 클래스 명을 찾지 않아도 된다
    - 실행 시 성능상의 차이가 없다
    - 하위 패키지의 클래스까지 포함하는 것은 아님
- java.lang 패키지의 import문은 생략되어있어 작성하지 않아도 사용할 수 있다

### static import문
- static 멤버를 호출할 떄 클래스 이름 생략 가능

## 제어자
- 선언부에 사용되어 부가적 의미 부여

### static - 클래스의, 공통적인
- 인스턴스 생성하지 않고 사용가능
- 클래스 변수
    - 모든 인스턴스가 공유하는 변수
    - 클래스가 메모리에 로드될 때 생성
- 메서드
    - 메서드 내에서 인스턴스 멤버를 사용하지 않는 메서드
    - 확장되지 않는 메서드
- static이 사용될 수 있는 곳
    - 멤버변수, 메서드, 초기화 블럭

### final - 마지막의, 변경될 수 없는

|종류|특징|
|---|---|
|클래스|변경될 수 없는 클래스, 확장될 수 없음|
|메서드|변경될 수 없는 메서드, 오버라이딩 불가|
|멤버변수, 지역변수|값을 변경할 수 없는 상수|

- 생성자를 이용한 final변수의 초기화
    - 보통 생성과 초기화를 동시에 하지만, 인스턴스 변수는 생성자로 초기화 가능
    - 인스턴스 별로 다른 값을 가진 상수를 선언하기 위한 목적

### abstract - 추상의, 미완성의
- 메서드에 사용
    - 선언부만 작성된 추상 메서드를 선언하는데 사용
- 클래스에 사용
    - 클래스 내에 추상 메서드가 존재함을 쉽게 알 수 있게 하는 목적
    - 인스턴스 생성 불가
    - 반드시 상속 해서 추상메서드를 완성하게 강제한다
    - 추상 메서드가 없고 빈 몸통만 있는 메서드만 있어도(java.awt.event.WindowAdapter) 인스턴스를 생성할 수 없게하고, 상속받아 원하는 메서드만 오버라이딩해서 사용할 수 있게하는 경우 존재.

### 접근 제어자(access modifier)
- 멤버(멤버변수, 메서드, 생성자) 또는 클래스에 사용
- 멤버 또는 클래스를 외부에서 접근하지 못하도록 사용
- 접근 제어자가 지정 안되어있다면, default임

|제어자|같은 클래스|같은 패키지|자손 클래스|전체|
|---|--|--|--|--|
|public|O|O|O|O|
|protected|O|O|O|X|
|(default)|O|O|X|X|
|private|O|X|X|X|

```Public > Protected > default > private```

#### 접근 제어자를 이용한 캡슐화
- 접근제어자 사용 이유는 
    1. 클래스 내부에 선언된 데이터를 보호하기 위함
        - 데이터를 함부로 변경할 수 없도록 접근을 제한
    2. 위부에는 보여줄 필요없는, 내부에서만 사용되는 부분을 감추기 위해서
    3. 코드 수정 시 영향 범위 최소화
- 상속을 확장될 수 있는 클래스는 자손에서 접근이 가능하도록 protected로

#### 생성자의 접근 제어자
- 보통 생성자의 접근제어자는 클래스의 접근 제어자와 같으나, 다르게 작성 가능
- 생성자의 접근 제어자가 private
    - 외부에서 객체 생성 불가(내부에서는 가능)
    - 내부에서 생성한 객체를 반환하는 public static 메서드를 제공 -> 싱글톤 패턴
    - 사용할 수 있는 인스턴스의 개수 제한
    - 생성자가 private인 클래스는 자손 클래스를 가질 수 없다
        - final을 더 추가해서 상속 불가능함을 알리자

### 제어자의 조합
1. 메서드에 static과 abstract를 함께 사용 불가
    - static은 바디가 있는 메서드에만 사용 가능
2. 클래스에 abstract와 final 동시 사용 불가
    - abstract는 상속을 강제하는 제어자인데, final은 상속을 불가하게 하는 제어자로 모순됨
3. abstract 메서드의 제어자가 private일 수 없다
    - 접근 제어자가 private이면, 상속하여 바디를 구현할 수 없기 때문이다
4. 메서드에 private과 final을 같이 사용할 필요가 없다
    - 오버라이딩이 불가함을 알리는 것은 private만으로도 충분하다

## 다형성
### 다형성이란?
1. 여러가지 형태를 가질 수 있는 능력
2. 조상 클래스타입으로 여러 자손 클래스타입의 인스턴스를 참조할 수 있는 성질

### 특징
- 인스턴스를 **어떤 타입으로 참조**하는지에 따라 **사용 가능한 멤버의 갯수**가 달라짐
- 조상 타입의 참조변수로 자손 타입의 인스턴스를 참조 가능
- 자손타입의 참조변수로 조상타입의 인스턴스를 참조 불가
    - 참조변수가 접근할 수 있는 범위가 인스턴스가 실제 가진 멤버 보다 더 많음, 존재하지 않는 멤버를 사용할 수 도 있다
### 참조변수의 형변환
- 서로 상속관계에 있는 클래스 사이에서만 가능
- 자손 -> 조상 형변환 생략 가능
    - 참조변수가 다룰 수 있는 멤버의 수가 실제 인스턴스 멤버의 수보다 작거나 같아 문제될게 없음
- 자손 <- 조상 형변환 생략 불가
    - 참조변수가 존재하지 않는 멤버를 사용할 가능성이 있어 생략 불가
- 컴파일 시, 참조변수 간 형변환만 체크(실제 인스턴스 체크 x)
    - 형변환 전에 instanceof 연산자 사용해서 실제 인스턴스 타입 확인하기
- 형변환은 실제 인스턴스에 아무런 영향 x, 단지 형변환을 통해 참조변수를 통해 인스턴스에서 사용할 수 있는 멤버의 갯수를 조정하는 것

### instanceof 연산자
- 참조변수가 참조하는 인스턴스의 실제 타입을 확인하는 연산자
- 왼쪽에 참조변수, 오른쪽에 클래스 타입
- true -> 실제 인스턴스를 클래스의 타입으로 형변환, 참조 가능
    - 실제 인스턴스의 멤버와 클래스 멤버 수가 같거나 인스턴스가 더 많음

### 참조변수와 인스턴스의 연결
- 메서드
    - 오버라이딩한 경우에도 참조변수의 타입과 관계없이 실제 인스턴스에 정의된 메서드 호출
- 참조변수
    - 오버라이딩한 경우, 참조변수의 타입에 따라 달라짐
    - 하지만 중복정의 되지 않았다면 참조변수에 관계없이 동일한 참조변수를 불러온다
- 인스턴스 변수에 직접 접근시, 참조변수의 타입에 따라 사용되는 인스턴스 변수가 달라질 수 있다
    - getter, setter를 통해 **간접접근** 해야한다

### 매개변수의 다형성
- 매개변수가 A타입의 참조변수라면, 메서드의 매개변수로 A의 자손타입 클래스는 다 인자로 올 수 았다는 의미

### 여러 종류의 객체를 매열로 다루기
- 조상타입의 참조변수 배열 사용하면 공통 조상을 가진 서로 다른 종류의 객체를 배열로 묶어 다룰 수 있다.

## 추상 클래스
### 추상 클래스란?
- 미완성 설계도(설계도 = 새 클래스를 작성하는데 바탕이 됨)
- 미완성 메서드를 포함하는 클래스
    - 이 특징을 제외하고는 일반클래스와 다른 점이 없다
- 상속을 통한 멤버 재사용과 확장에 목적이 있음
### 추상 클래스의 완성
- 자손클래스가 상속을 받아야지만 완성 가능
- 미완성이므로 인스턴스 생성 불가
### 추상클래스 사용
```
abstract class 클래스이름{
    ...
}
```
### 추상메서드
- 선언부는 작성, 구현부는 작성하지 않은 메서드
- 사용 이유
    - 메서드의 내용이 상속받는 클래스에 따라 내용이 달라질 수 있음 
        - 주석을 붙여 추상 메서드의 역할을 설명
        - 실제 내용은 상속 클래스에서 구현하게 비워둠
- 장점
    - 선언과 구현을 분리
        - 구현하지 않아도 추상 메서드를 사용하는 코드 작성 가능
        -> 구현부보다 중요한 부분이 선언부

### 추상메서드 사용
```
abstract 리턴타입 메서드이름();
```
- 상속 받는 자손클래스가 추상 메서드를 모두 구현해야
- 일부만 구현한다면, 자손클래스도 추상 클래스로 지정

### 추상클래스의 작성
- 중복 코드를 제거하기 위해서 두 가지 **추상화** 방법을 쓴다
    1. 공통점을 뽑아서 클래스를 바로 작성
    2. 공통점을 뽑아 추상 클래스로 만들어 상속
- 공통점을 찾아내 공통의 조상을 만드는 것을 **추상화**라고 한다
    - 반대로 상속을 통해 클래스를 구현, 확장하는 것을 **구체화**라고 한다
- 1번에서 빈 블록만 작성하고 오버라이딩으로 작성하는 것도 가능한데 왜 굳이 추상메서드를 사용할까?
    - 자손 클래스에서 추상 메서드를 반드시 구현하도록 **강제**하기 위해서
        - 실수할 수 있으니까(자바는 실수하는 사람들에게 친절한 언어다)
    - 이 특징 빼고 조상 클래스와 추상 클래스의 작성은 거의 동일

## 인터페이스
### 인터페이스란?
- 일종의 추상 클래스. 추상 클래스보다 추상화의 정도가 높다
    - 추상 메서드와 상수 (JDK1.8 ~ + default, static 메서드)
- 기본 설계도(새로운 클래스 작성에 도움 줄 목적)

### 작성
```
interface 인터페이스이름{
    [public static final] 타입 상수이름 = 값;
    [public abstract] 메서드이름(매개변수);
}
```
- 접근제어자 : public, default(구현 범위를 패키지 내로 제한)
- public static final, public abstract
    - 모든 멤버는 반드시 '노출' 되어야 함. (기본 설계도이기 때문에 encapsulation이 필요 없어서 그런가?)
- 인터페이스 이름
    - 어떤 기능, 행위를 하는데 필요한 메서드 제공하는 의미로 able로 끝나는 이름이 많음
    - 상속과 달리 인터페이스는 **어떤 행위가 가능한 것을 보장**함을 알 수 있다

### 인터페이스의 상속
    - 인터페이스로부터만 상속 가능
    - 다중 상속 가능(충돌 문제가 없음)

### 구현
- implements
- 일부만 구현하면 추상클래스로 선언해야
- 상속(extends)과 구현(implements)를 동시에 가능

### 인터페이스를 이용한 다중상속
- 인터페이스로 다중상속을 구현하는 경우는 거의 없음을 알아두자
- 두 개의 클래스를 인터페이스 만으로 다중상속해야할 때
    - 비중이 높은 쪽을 상속, 그렇지 않은 클래스를 포함
    - 둘 중 하나의 필요한 부분을 뽑아 인터페이스로 작성하고 구현
        - 다형적 특성 이용가능

### 인터페이스를 이용한 다형성
- 인터페이스 타입의 참조변수로 이를 구현한 클래스를 참조 가능(매개변수, 리턴타입)
- 인터페이스 타입으로 형변환 가능

### 인터페이스의 장점
- 개발시간 단축(선언과 구현의 분리로 양쪽에서 동시에 개발 진행 가능)
- 표준화 가능
    - 기본 틀을 인터페이스로 먼저 작성할 수 있어서
- 관계 없는 클래스에 관계를 맺어줄 수 있음
- 독립적 프로그래밍 가능
    - 선언과 구현의 분리
    - 직접적 관계 -> 간접적 관계로 변경
    - 수정에 유리한 프로그래밍 가능
<details>
<summary>이미 상속중인 A 클래스에 공통 코드를 추가해야할 때(다중 상속과 동일)</summary>
<div markdown = 1>

1. 인터페이스 작성
2. 구현 클래스(impl) 작성
3. A클래스가 인터페이스 구현
4. A클래스에 구현 클래스 포함하여 내부적으로 호출
- 장점
    - 구현 클래스에서 공통 코드를 한 번에 관리 가능
    - 재사용 가능
</details>

### 인터페이스의 이해
- 클래스를 사용하는 쪽과 제공하는 쪽이 있다
    - (사용과 제공을 분리하기 위해서는 input, ouput이 분명해야한다)
    - 직접적인 관계일 때, 제공이 변하면 사용도 변해야한다
    ```
    class A { //사용하는 쪽
        public void methodA(B b){
            b.methodB();
        }
    }

    class B { //제공하는 쪽
        public void methodB(){...}
    }

    class Test{
        public static void main(Strings[] args){
            A a = new A();
            a.methodA(new B());
        }
    }
    ```

    ```
    class A { //사용하는 쪽
        public void methodA(C c){
            c.methodB();
        }
    }

    class B {
        public void methodB(){...}
    }

    class C { //제공하는 쪽
        public void methodB(){...}
    }
    ```
    - 인터페이스를 통해서 선언과 구현부를 분리할 수 있다
    ```
    interface I {
        public abstract void methodB();
    }

    class B implements I{ //제공
        public void methodB(){...}
    }

    class A { //사용
        public void methodA(I i){
            i.methodB();
        }
    }

    class Test{
        public static void main(Strings[] args){
            A a = new A();
            a.methodA(new B());
        }
    }
    ```
    ```
    interface I {
        public abstract void methodB();
    }

    class B implements I{
        public void methodB(){...}
    }

    class C implements I{ //제공
        public void methodB(){...}
    }

    //제공이 변해도 사용은 변하지 않는다
    //직접 -> 간접 관계로 바뀜
    class A { //사용
        public void methodA(I i){
            i.methodB();
        }
    }

    class Test{
        public static void main(Strings[] args){
            A a = new A();
            a.methodA(new C());
        }
    }
    ```
- 메서드를 사용하는 쪽은 선언부만 알면 됨(블랙박스)
- 매개변수나 클래스, 파일등을 통해서 인터페이스 I를 구현한 클래스를 동적으로 제공받아야한다

### 디폴트 메서드와 static 메서드
#### 디폴트 메서드
- 인터페이스에 새로운 메서드 추가 시 모든 구현 클래스에서 구현해야한다는 부담 때문에 도입
- 구현된 일반 메서드
- 추상 메서드가 아니기 때문에 구현 클래스에서 구현할 필요 없다
- default 키워드
- 접근 제어자는 public, 생략 가능
- 충돌 시
    - 여러 인터페이스의 디폴트 메서드 간의 충돌
        - 인터페이스 구현 클래스에서 오버라이딩
    - 디폴트 메서드와 조상 메서드의 충돌
        - 디폴트 메서드 무시, 조상 메서드 자동 상속

## 내부 클래스
### 내부클래스란?
- 클래스 내에 선언된 클래스
- 두 클래스가 서로 긴밀한 관계 있기 때문에 선언
- 장점
    - 내부 클래스에서 외부 클래스 멤버들을 쉽게 접근할 수 있다
    - 코드의 복잡성을 줄 일 수 있다(캡슐화)

### 종류와 특징
- 선언위치에 따라 종류가 달라짐
- 변수 선언과 같은 위치에 선언, 멤버와 내부 클래스의 유효범위와 성질이 비슷

|내부 클래스|특징|
|------|------------|
|인스턴스 클래스| 멤버변수 선언위치, 외부클래스의 인스턴스 멤버처럼 다뤄짐, **외부 클래스의 인스턴스 멤버와 관련된 작업**에 사용|
|스태틱 클래스|멤버변수 선언위치, 외부클래스의 스태틱 멤버처럼 다뤄짐, **외부 클래스의 static멤버, static 메서드에서 사용**될 목적으로 선언|
|지역 클래스|외부 클래스 메서드나 초기화 블럭에 선언, **선언된 영역 내에서만 사용가능**|
|익명 클래스|객체 선언과 생성을 동시에 하는 이름없는 일회용 클래스|

- 클라스 파일 명은 '외부 클래스명$내부클래스명.class'
    - 지역 내부 클래스명은 내부 클래스 명 앞에 숫자가 붙음. 한 클래스 내에 같은 이름이 있을 수 있으니까
### 내부 클래스의 제어자와 접근성
#### 제어자
- abstract나 final 사용 가능(클래스 특성)
- private protected(+기존 public, default) 사용 가능(멤버변수 특성)
- 내부 클래스 중 static class만 static 멤버를 가질 수 있다
    - 내부 클래스를 사용하려면 외부 클래스 객체를 생성해야
    - static 내부 클래스는 static 멤버와 동일하기 떄문에 클래스 로딩 시 생성, 외부 클래스의 인스턴스 생성하지 않고서도 생성됨. 따라서 static 내부 클래스 안의 static 멤버도 외부 클래스를 생성하지 않고 접근 가능
    ```
    class OuterClass{
        static class staticInner{
            static int cv = 100;
        }
    
        public static void main(Strings[] args){
            System.out.println(StaticInner.cv); //100
        }
    }
    ```
    - final과 static이 동시에 붙은 변수는 상수, 모든 내부 클래스에서 사용 가능
        - constant pool에서 따로 관리해서
#### 접근성
- 인스턴스 내부 클래스
    - 외부 클래스 인스턴스를 먼저 생성해야 생성할 수 있다
    - 인스턴스 변수, static변수 모두 사용 가능
- static 내부 클래스
    - 인스턴스 멤버, 인스턴스 내부 클래스 접근 불가
    - static 멤버 접근 가능
- 지역 클래스
    - 외부 클래스의 인스턴스, static 멤버 모두 사용 가능
    - 메서드 내의 지역변수 사용 가능
        - final만
        - JDK1.8 부터 지역변수 앞에 final생략 가능(메서드 내에서 값이 변하면 사용 불가, 컴파일 에러)
- 외부 클래스가 아닌 다른 클래스에서 내부 클래스에 접근하는 경우는 없다. 생긴다면, 내부 클래스를 잘못 선언한 것

### 익명 클래스
- 이름이 없다
- 클래스의 선언과 생성을 동시에 함
- 일회용
- 생성자도 가질 수 없다
- 오로지 단 하나의 클래스를 상속 받기 or 인터페이스 구현
- 오버라이딩 한 멤버만 사용 가능. 참조 변수의 타입이 부모 클래스 또는 인터페이스이기 때문이다. 

#### 사용
1. 클래스 필드로 이용
    - 특정 클래스 내부나 여러 메서드에서 이용될 때 고려
2. 지역 변수로서 이용
    - 메서드에서 일회용을 사용할 때 고려
3. 메개변수로서 이용
    - 일회성으로 인자를 넘겨줘야할 때 고려


참고
- 남궁성의 자바의 정석




