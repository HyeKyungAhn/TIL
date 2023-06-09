# 조건문과 연산자

## 제어문
<details>
<summary>제어문이란?<del>(종류, 구성, 기능)</del></summary>
<div markdown = 1>

- 프로그램의 흐름을 바꾸는 역할을 하는 문장
- 종류 
    - 조건문, 반복문
- 구성
    - 조건문
    - 문장들을 포함하는 블럭{}
- 기능
    - 조건식의 결과에 따라 수행하는 문장이 달라져 프로그램의 흐름을 제어
    - 조건식의 경우의 수가 많을 때에는 switch문을 사용하지만, 제약이 많아 if문을 더 많이 사용한다
</details>

### 조건문

<details>
<summary>if문</summary>
<div markdown = 1>

- 구성
    ```
    if(조건식){
        //조건식이 참일 때 실행된 문장들
    }
    ```
    - 조건식과 괄호{}
    - 조건식
        - 비교연산자와 논리연산자로 구성
        - 결과값은 반드시 boolean 타입이어야
    - 블럭{}
        - 여러 문장을 하나의 단위로 묶음
        - {} 뒤에 ; 붙이지 않는다
        - 하나의 문장만 넣으면 {} 생략가능하나, 실수 방지를 위해 되도록 생략하지 말자
- 기능
    - 조건식이 참이면 괄호 안의 문장 수행
</details>

<details>
<summary>if-else문</summary>
<div markdown = 1>

- 구성
    ```
    if(조건식){
        //조건식의 결과가 참일 때 실행될 문장들
    } else {
        //조건식의 결과가 거짓일 떄 실행될 문장들
    }
    ```
- 기능
    - 조건식의 결과에 따라 두 블럭 중 하나가 실행되고 전체 if문을 빠져나온다
</details>

<details>
<summary>if-else if문</summary>
<div markdown = 1>

- 구성
    ```
    if(조건식){
        //조건식의 결과가 참일 때 실행될 문장들
    } else if(조건식2) {
        //조건식2의 결과가 참일 떄 실행될 문장들
    } else if(조건식3){
        //조건식3의 결과가 참일 떄 실행될 문장들
    } else {
        //어느 조건식도 만족하지 않을 때
    }
    ```
- 기능
    - 첫번째 조건부터 '순서대로' 평가해서 결과가 참인 조건식을 만나면 해당 블럭만 실행한 후, 전체 if-else if문을 빠져나온다
    - '순서대로' 평가하기 때문에 n번째 조건식은 그 전 n-1까지의 조건을 중복해서 확인할 필요가 없다. 이미 거짓임을 확인했기 떄문이다.
    ```
    if(score >= 90){
        System.out.println("A등급");
    } else if(score >= 80){  // 80 <= score && score <= 90 과 동일
        System.out.println("B등급");
    } else if(score >= 70){
        System.out.println("C등급");
    }
    ```
</details>

<details>
<summary>중첩 if문</summary>
<div markdown = 1>

- if문 안에 if문을 포함시킬 수 있다
- 중첩의 횟수에는 거의 제한이 없다
- 정말 헷갈리기 떄문에 들여쓰기나 {}의 생략을 주의해야한다
</details>

<details>
<summary>switch문</summary>
<div markdown = 1>

- 사용하는 경우
    - 처리할 경우의 수가 많은 경우에는 if문보다 switch문을 사용
    - 그러나 switch문은 제약이 있다
- 평가 과정
    1. 조건식 계산
    2. 결과값에 해당하는 case문 찾기
    3. 해당 case문의 문장들을 수행
    4. break나 switch문의 끝을 만나면 swicth문 전체를 빠져나감
- default
    - 조건식의 결과와 일치하는 case문이 없는 경우에는 default문으로 이동
    - 위치는 상관없으나, 보통 마지막에 놓음
    - 마지막에 놓으면 break문은 쓰지 않아도 됨
- break
    - 기능
        - 각 case문의 영역을 구분하는 역할
        - 생략 시, break문을 만나거나 switch문의 끝을 만날 때 까지 모든 문장을 수행
- 제약조건
    1. switch문의 조건식 결과는 정수 또는 문자열이어야
    2. switch문의 값은 정수, 상수, 문자열(JDK 1.7 이후) 만 가능, 중복 x
- 조건문으로 case의 수를 줄이는 것이 중요
- switch문의 중첩 가능
</details>

## 반복문
<details>
<summary>반복문<del>(정의, 종류, 조건문, 특징)</del></summary>
<div markdown = 1>

- 정의
    - 어떤 작업이 반복적으로 수행되도록 할 때 사용
- 종류
    - for문, while문, do-while문
</details>

<details>
<summary>for문</summary>
<div markdown = 1>

- 특징
    - 반복 횟수를 알고 있을 때 적합
- 구조
    ```
    for(초기화식;조건식;증감식){
        //조건식이 참일 때 수행될 문장들
    }
    ```
    - 초기화, 조건식, 증감식, {}
        - 초기화
            - 반복문에서 사용되는 변수를 선언
            - 반복문의 제어에 사용되는 변수의 값은 문장 내에서 임의로 수정하지 않는 것이 좋다
            - 둘 이상의 변수를 ','를 구분자로 사용가능
            - 단 변수들의 타입은 같아야한다
            - 하지만 변수의 갯수는 적을수록 좋다
        - 조건식
            - 조건식이 참인동안 반복을 계속
        - 증감식
            - 변수의 값을 증가 또는 감소
            - 다양한 연산자들로 증감식 작성 가능
            - 두 개 이상의 증감식을 ','를 구분자로 사용 가능
            - 생략 가능하나 무한 반복문이 되겠지요
- 수행순서
    - 초기화식 -> (조건식 -> 블럭 -> 증감식)(반복)
- 중첩 for문은 별뽑기로 연습
- 향상된 for문
    - 배열과 컬렉션에 저장된 요소에 접근할 떄만 사용
</details>

<details>
<summary>while문</summary>
<div markdown = 1>

- 구성
    ```
    while(조건식){
        //조건식의 결과가 true일 때 수행될 문장들
    }
    ```
    - 조건식과 블럭{}
- 기능
    - 조건식이 참인 동안, 조건식이 거짓이 될 때까지 블럭내 문장 실행
- for문과 while문의 비교
    - 서로 100% 호환 가능
    - 초기화나 증감식이 필요하지 않은 경우 while문이 더 적합
- 조건식
    - 생략 불가
- do-while문
    - 구조  
        ```
        do {
            //조건식의 결과가 참일 때 수행될 문장들
        } whild(조건식); //세미콜론 빼먹지 않기!!
        ```
        - while문의 조건식과 {}의 위치를 바꿔놓은 것
        - {}블럭을 먼저 수행하기 때문에 {}안의 코드의 최소한 한 번 수행을 보장
</details>

<details>
<summary>while문</summary>
<div markdown = 1>

- break문
    - 기능  
        - 자신이 포함된 가장 가까운 반복문을 벗어남
        - 주로 if문과 함께 사용
- continue문
    - 기능
        - 반복문 내에서만 사용
        - 반복문의 끝으로 이동하여 다음 반복문으로 넘어간다
        - 전체 반복 중 특정 조건을 만족하는 경우를 제외하는데 유용
- 이름 붙은 반복문
    ```
    Loop1: for(int i=0 ; i<=9 ; i++){
        for(int j=2; j<=9; j++){
            if(j==5)
                break Loop1;
            System.out.println(i + " * " + j + " = " + (i*j));
        }
        System.out.pringln();
    }
    ```
    - 중첩 반복문 앞에 이름을 붙이고 break문과 continue문에 이름을 지정
    - 기능
        - 하나 이상의 반복문을 벗어나거나 반복을 건너 뛸 수 있다
</details>

## 참고
- 남궁성의 자바의 정석 4장-조건문과 연산자