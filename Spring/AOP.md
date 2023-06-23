# Spring AOP란?

## AOP 정의
AOP란 핵심기능을 앞 뒤로 둘러싸고 있는 반복되고 부가적인 코드를 따로 분리해서 실행 중에 자동으로 추가해주는 기능을 말한다.

### 횡단 관심사
횡단관심사(cross-cutting concerns)라는 어려운 이름도 가지고 있다. 모듈의 구분 없이 공통적으로 사용하는 기능들(logging, security, Tx..)을 각 모듈마다 구현하지 않고 따로 떼어서 실행 중에 자동으로 추가해주기 때문에 관심사(모듈들)를 가로지른다(=모듈의 구분없이 추가한다)는 특징에서 그 이름이 나왔다.
</br></br>
<p align="center">
<img src="image.png" width="80%" height="80%" align="center">
</p>
</br></br>



### AOP의 적용
AOP는 핵심 기능에 부가기능을 동적으로(=실행 중에) 추가해주는 기능이다. 그렇다면 어디에 추가를 할 수 있을까? 핵심기능의 시작, 끝 또는 양쪽에 추가할 수 있다. 정확하게는 핵심기능이 구현된 메서드의 시작과 끝에 추가할 수 있다.

## AOP와 관련된 용어
- target : 핵심 기능이 구현된 객체
- advice : target에 동적으로 추가될 기능(코드)
- proxy : advice가 동적으로 target에 추가되어 생성된 객체
- weaving : advice가 동적으로 target에 추가되어 proxy가 만들어지는 것
- join point : advice가 추가된 메서드
- pointcut: advice가 추가될 join point의 패턴을 설정

## AOP를 간단하게 구현해보기
핵심 기능이 구현된 MyTarget 클래스와 그 안에 정의된 pointA, pointB, pointC 메서드가 있다. @Component를 사용해서 target 클래스를 빈으로 등록해주자.
```
@Component
class MyTarget {
    public void pointA(){
        System.out.println("pointA is called");
    }

    public void pointB(){
        System.out.println("pointB is called");
    }

    public void pointC(){
        System.out.println("pointC is called");
    }
}
```

위의 메서드들에 부가기능으로 추가할 MyAdvice도 정의하자. 
```
@Component
@Aspect
class MyAdvice {
    @Around("execution(void aop.MyTarget.*(..))") //pointcut
    public void weave(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("[start]");
        pjp.proceed();
        System.out.println("[end]");
    }
}
```

- MyAdvice 클래스도 컨테이너에 빈으로 등록되어야 하고, @Aspect를 선언해야한다.
@Aspect는 해당 클래스를 aspect로 선언한다.
- ProceedingJoinPoint는 join point 메서드에대한 모든 정보를 가지고 있다.
- pointcut은 [pointcut 패턴을 잘 설명한 waterpunch님의 글](https://wpunch2000.tistory.com/22)을 보자.
- 리턴타입은 다른 advice에 결과값을 넘길 때는 Object, advice를 단독으로 사용할 때는 void로 선언한다.

이제 MyTarget에 선언된 pointA,B,C 메서드를 호출해보자. applicationContext를 직접 생성해서 객체들을 스캔하고 생성해서 가져왔다.
```
Class Main {
        ApplicationContext ac = new GenericXmlApplicationContext("file:web/WEB-INF/applicationContext_aop.xml");

        MyTarget target = (MyTarget) ac.getBean("myTarget");
        target.pointA();
        target.pointB();
        target.pointC();
}
```

```
//결과
[start]
pointA is called
[end]
[start]
pointB is called
[end]
[start]
pointC is called
[end]
```
MyTarget의 메서드들의 결과값 앞 뒤에 MyAdvice의 코드들이 추가된 것을 확인할 수 있다.


## 마무리
- AOP든 OOP든 결국은 수정에 유리한 코드를 작성하도록 돕는 기능이나 방법론이라 관심사의 분리와 중복제거가 얼마나 중요한지를 오늘 복습을 통해 다시 한번 깨닫는다.
- todo
    - aspect에 대해 공부
    - Transactional에 대해 공부
        (https://stackoverflow.com/questions/1099025/spring-transactional-what-happens-in-background)