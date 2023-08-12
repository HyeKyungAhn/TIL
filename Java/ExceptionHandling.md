# 예외처리
## 프로그램 오류
- 오류의 세 가지 종류
    1. 컴파일 에러
        - 소스코드에서 컴파일러가 오타나 구문오류 등의 오류를 잡아낸 것
        - 컴파일 에러를 해결해야 class파일을 생성하고 실행할 수 있다
    2. 런타임 에러
        - 프로그램 실행 도중 발생할 수 있는 잠재적인 오류
        - 프로그램의 비정상적인 종료 등을 야기하는 에러
    3. 논리적 에러
        - 정상적으로 실행되지만, 의도와 다르게 동작하는 에러
- 런타임 에러의 두 종류
    1. 에러
        - 프로그램 코드로 수습할 수 없는 심각한 오류
    2. 예외
        - 프로그램 코드로 수습할 수 있는 다소 미약한 오류

## 예외 클래스의 계층 구조
- Exception과 그 자손들
    - 사용자등 외적인 요소에 의해 발생할 수 있는 예외
- RuntimeException과 그 자손들
    - 개발자의 실수로 발생하는 에외

## 예외의 발생과 처리
### 예외의 발생
- 예외 발생시, 예외에 해당하는 클래스의 인스턴스가 생성된다

### 예외의 처리
- 예외 처리란?
    - 정의 : 프로그램 실행 시 발생할 수 있는 예외 상황에 대비한 코드를 작성하는 것
    - 목적 : 비정상적인 종료를 막고 정상적인 실행 상태를 유지하는 것
- 처리되지 못한 예외는?
    - 처리되지 못한 예외(Uncatched Exception)는 JVM의 예외처리기가 받아서 화면에 예외원인을 표시(500..)

## try-catch
```
try{
    //예외 발생할 수 있는 코드
} catch(예외클래스 e){
    ...
} catch(예외클래스 e){

} catch(예외클래스 e){

}
```
- 하나 이상의 catch블럭이 올 수 있다
- 발생한 예외 인스턴스를 처리할 수 있는 catch블럭 딱 한 개 만 실행된다
- 일치하는 catch 블럭이 없으면 예외는 처리되지 못한다
- 하나의 메서드에 여러개 try-catch 블럭이 올 수 있고 catch블럭 내에 또 try-catch블럭이 올 수 있다
- catch블럭의 파라미터는 해당 블럭 내에서만 유효
- try블럭에서 예외 발생 시, 예외 발생한 코드 이후의 코드는 수행되지 않음. 코드 작성 순서나 try블럭의 범위도 잘 선택해야한다

## 예외의 발생과 catch 블럭
- try 블럭 안에서 예외 발생 시 
    1. catch블럭을 처음부터 차례로 내려가며
    2. catch 블럭의 괄호 내에 선언된 참조변수와 예외 인스턴스를 instanceof연산한다
    3. 참이면 catch블럭 내의 코드 실행 후, 전체 블록을 빠져나가고 false이면 다음 catch 블록을 연산한다
    - 참인 블럭이 없다면 예외는 처리되지 않는다
- catch의 괄호에 Exception 선언
    - 어떤 종류의 예외가 발생해도 모두 처리(마지막 catch문으로 선언하기도)
### printStactTrace, getMessage
- printStackTrace() : 예외 발생 당시의 호출 스택에 있던 메서드 정보, 예외 메시지
- getMessage() : 발생한 예외 인스턴스에 저장된 메시지 가져오기

### 멀티 catch블럭
- 하나의 catch블럭으로 합칠 수 있다
- 목적
    - 코드 중복 방지
- 특징
    - 갯수 제한이 없다
    - 연결된 예외 클래스간의 관계가 상속관계에 있다면 컴파일 에러 발생
        - 코드 중복 방지가 목적이고, 조상의 것만 선언하면됨
- 단점
    - 멀티 catch 블럭 내에 실제 어떤 예외가 발생했는지 알 수 없다
        - 사실, instanceof로 알아서 개별 처리는 가능하다
    - catch블럭의 참조변수는 상수로 값을 변경할 수 없다
        - 연결된 클래스들의 공통분모인 조상예외 클래스에 선언된 멤버만 사용 가능

## 예외 발생시키기
```
//1. 원하는 예외 클래스 인스턴스 생성
Exception e = new Exception("예외 발생");
//2. 키워드 throw로 예외 발생시키기
throw e;
```
- 생성자에 String넣으면 예외 메시지로 저장됨
### checked예외 unchecked예외
- Exception과 그 자손(RuntimeException 과 그 자손 제외)은 catched 예외
    - 처리하지 않으면 컴파일 조자 되지 않는 반드시 처리되어야하는 예외
- RuntimeException과 그 자손은 uncatched 예외
    - 개발자의 재량에 맡긴, 처리하지 않아도 되는 예외

## 메서드에 예외 선언하기
- 키워드 throws로 메서드 선언부에 발생할 수 있는 에외 작성
```
void methodA() throws Exception1, Exception2, ExceptionN {...}
```
- 목적
    - 메서드를 사용하는 쪽에서 이에 대한 처리를 강요
    - 자신(예외를 선언한 메서드)을 호출한 메서드에 예외를 전달하여 처리를 떠넘기기 위해
        - 메서드를 사용할 떄 API문서로 발생할 수 있는 에외를 확인하는 습관을 들이자
- 대상
    - 선언해야하는 예외는 보통 **반드시 처리**해야하는 예외만 선언한다(checked예외)

- 특징
    - Exception 클래스를 선언하면 모든 예외가 발생할 수 있는 메서드라는 뜻
    - 예외를 단순히 전달하는 역할이다. 어느 한 곳에서 반드시 try-catch로 예외처리를 해주어야한다

### 예외 처리를 어디서 할 것인가?
- 예외가 발생한 메서드에서 예외 처리
    - 호출한 쪽에서는 예외가 발생했는지도 모름
    - 예외를 자체적으로 처리할 수 있고 해도 되는 경우
- 예외 선언하여 예외 떠넘기기
    - 호출한 라인에서 예외가 발생한 것으로 간주, 처리함
    - 메서드에 호출 시 넘겨 받아야하는 값을 다시 받아야하는 경우, 자체적으로 해결이 안되는 경우에 예외를 메서드에 선언해야
- 두 메서드가 예외 처리를 분담할 수 도 있음

## finally 블럭
- 예외의 발생 여부와 관계없이 실행되어야할 코드를 포함시킬 목적으로 사용
- try블럭에서 return문이 실행되는 경우에도 finally블럭이 실행된 후 메서드를 종료
- catch 블럭에서 return문을 만나도 finally 블럭은 수행됨
## try-with-resources
- try문에서 사용되는 자원을 반환하기 위해 도입
- 자원 반환을 위한 close()를 finally에 선언하면 안되나?
    - close()가 예외를 발생시킬 수 있음
    - try문과 finally문 두 곳에서 모두 처리되지 않는 예외가 발생하는 경우, try블럭의 예외는 무시됨
    - 자원반환도, 예외 처리도 제대로 되지 않을 수 있음
```
try(FileInputStream fis = new FileInputStream("score.dat")){
    ...
}catch (EOFException e){...}

```
- 실행순서
    - try문의 괄호에 객체를 생성하는 문장을 넣으면, close()를 호출하지 않아도 try블럭을 벗어날 때 자동적으로 close()가 호출됨
        - 자동으로 close()가 호출되려면 클래스가 AutoCloseable이라는 인터페이스를 구현해야함
    - 그 다음 catch나 finally블럭이 수행됨

### SuppressedException
- 자동 호출된 close()에서 예외 발생하면 어떻게 될까?
    - close() 호출 시에만 예외 발생
        - 일반적으로 예외 출력됨
    - try문와 close() 호출 시 동시에 예외 발생
        - close()시 발생한 예외가 suppressed Exception으로 다룬다
- 억제된 예외
    - 실제 발생한 예외에 저장된다.
    - Suppressed라는 머리말과 함꼐 출력된다
    
    ```
    void addSuppressed(Throwable e) // 억제된 예외 추가
    Throwable[] getSuppressed() // 억제된 예외(배열)를 반환
    ```

- 기존의 try-catch문을 사용하면, 먼저 try문에서 발생한 예외는 무시되고 마지막으로 발생한 CloseException에 대한 내용만 출력되었을 것이다

### 사용자 정의 예외
- 보통 Exception, RuntimeException예외를 상속받아 생성
- 가능하면 기존의 예외를 활용하는 것을 권함
- 반드시 예외 처리를하지 않아도 되는 RuntimeException을 상속받아 처리하는 경우가 많다. 선택적으로 처리해야하는 상황이 많기 때문이다

## 예외 되던지기
- 정의
    - 예외를 처리한 후에 인위적으로 다시 발생시키는 것(exception re-throwing)
- 목적
    - 발생할 수 있는 예외가 여럿인 경우, 양쪽에서 나눠서 예외를 처리하도록
    - 단 하나의 예외에 대해서도 발생한 메서드, 호출한 메서드 양쪽에서 처리하도록
- 사용
```
class Test{
    public static void main(Strings[] args){
        try{
            methodA();
        } catch(Exception e){
            e.printStactTrace(); //5. 예외 재처리
        }
    }

    public static void methodA() throws Exception{ //4. 예외 던지기
        try{
            throw new Exception(); // 1. 예외 발생
        } catch(Exception e){
            //2. 예외 처리 코드 
            throw e; //3. 예외 재발싱, 
        }
    }
}
```
- return문
    - 반환값이 있는 return문은 catch문에도 return문이 있어야
    - 예외 되던지기를 catch문에서 하는 경우 return문이 없어도 된다
    - +finally에도 return문이 있을 수 있는데, try, catch 수행 이후 최종적으로 fianlly의 return문이 수행됨

## 연결된 예외
- 정의 :  예외 A가 예외 B를 발생시키면, A를 B의 원인 예외라고 한다
- 등록방법
    ```
    try {
        startInstall();
        capyFiles();
    } catch(SpaceException e){
        InstallException ie = new InstallException("설치 중 예외발생");
        ie.initCause(e);
        throw ie;
    } catch (MemoryException e){}
    ```
    1. InstallException 예외 생성
    2. InstallException에 SpaceException을 원인예외로 등록
    3. InstallException을 던지기
- 목적
    - 여러가지 예외를 하나의 큰 분류의 예외로 묶어 다루기 위함
        - 이를 상속으로 구현할 때의 단점이 있기 때문에 연결된 예외 도입
            - InstallException의 자식을 Spcae, MemoryException으로 구현
            1. 멀티 catch문처럼 어떤 예외가 발생했는지 알기 어렵고
            2. 공통된 멤버만 사용할 수 있으며
            3. 상속관계를 변경해야하는 부담
    - checked예외를 unchecked예외로 바꾸기 위함
        - unckecked예외를 생성하고, checked예외를 생성한 예외의 원인예외로 등록해 unckecked예외를 되던질 수 있다
        - initCuase()뿐만 아니라, RuntimeException의 생성자에 바로 checked 예외를 넣어줘서 원인예외로 등록 가능
        
        ```RuntimeException(Throwable casue) //원인 예외를 등록하는 생성자```
