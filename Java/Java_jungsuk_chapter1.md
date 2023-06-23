# 자바의 정석 복습 - 1장
## 자바(Java Programming Language)
### 1. 자바란
- 객체지향 언어
- 운영체제에 독립적
- 풍부한 클래스 라이브러리 제공
</br>
</br>
### 2. 자바의 역사
- 가전제품용 소프트웨어를 만들기위해 C++의 단점을 보완한 새로운 언어 만듬
- 인터넷의 등장으로 인터넷에 적합한 방향으로 개발방향을 전환
- 자바로 구현한 applet으로 인기를 얻었음
- 서버 프로그래밍을 위한 JSP, Servlet등이 더 많이 사용되고 있음
</br>
</br>

### 3. 자바언어의 특징
1. 운영체제에 독립적
2. 객체지향 언어
3. 비교적 배우기 쉽다
4. GC(Garbage Colletion)가 메모리 관리를 자동적으로 해준다
5. 네트워크와 분산화 처리 지원
6. 동적 로딩 지원(=클래스가 필요할 때 클래스의 정보를 로딩한다)
7. 멀티쓰레드 지원
</br>
</br>

### 4. JVM(Java Virtual Machine)

- 자바를 실행하는 가상 컴퓨터. 가상 컴퓨터는 하드웨어를 소프트웨어화 시켜 구현한 것. 
- 자바 응용 소프트웨어는 JVM에서만 실행이 가능하다.
- 다른 언어는 OS위에서 바로 실행되나, Java는 반드시 JVM을 거쳐야하기 때문에 속도가 느리다는 단점이 있음(JIT 컴파일러 사용, 최신 기술이 적용되어 속도 차이 줄임)
- JVM이 있기에 Java 응용 소프트웨어가 운영체제에 독립적으로 실행될 수 있다
- 다만 JVM이 운영체제에 종속적 -> OS에 맞는 JVM 설치 필요

## 자바 개발환경 구축
### JDK와 JRE
#### 1. JDK(Java Development Kit)
    
- JRE + 자바 개발에 필요한 실행파일(javac.exe등)
#### 2. JRE(Java Runtime Environment)
- 자바 실행에 필요한 최소 환경
- JVM + 클래스 라이브러리 (Java API)

## 자바로 프로그램 작성하기
### 자바 파일의 컴파일과 실행
```
Class Hello {
    public static void main(String[] args){
        System.out.println("Hello, world");
    }
}
```

Hello.java = (javac.exe 컴파일==바이트 코드로 변환) => Hello.class =(java.exe 실행)=> 'Hello, world' 출력

### psvm
자바 애플리케이션의 시작과 종료눈 자바 파일에 선언된 public static void main 메서드 호출에서 마지막 코드까지 실행하고 종료하는 것과 동일하다. 그래서 반드시 1개의 psvm이 존재해야 애플리케이션을 실행시킬 수 있다.

### public
- .java 파일의 제목은 반드시 파일 내에 선언된 public 클래스의 이름과 동일해야한다. 
- 대소문자를 구별하며
- public 클래스가 없다면 안에 선언된 클래스의 이름 중 하나만 일치하면 된다

### class파일
- 하나의 .java 파일에 여러개의 클래스가 정의될 수 있다.
- 하나의 java 파일에 작성되지만 javac.exe를 실행하여 자바 파일을 컴파일하면 정의된 클래스의 수만큼 .class 파일이 생성된다.

## 자바 프로그램의 실행 과정
`> java Hello`

1. Hello 클래스파일(.class)을 로드한다
2. 클래스 파일을 검사한다(악성 코드, 파일 형식)
3. 클래스의 psvm을 호출한다

## 주석
- 주석은 다른 사람이 내 코드를 읽을 떄 이해를 돕기 위해 작성해야한다
- 이해를 돕기 위해서는 변수명이나 메서드명도 잘 지어야겠다

## 그 외
- 질문을 잘 하자