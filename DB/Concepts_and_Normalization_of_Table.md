# 테이블의 개념과 1~3정규형
## 테이블이란
테이블은 공통된 요소가 모인 **'집합'** 이며 인풋이 있을 때 하나의 아웃풋이 나와야하는 **'함수'** 로 볼 수있다. 또한 현실세계를 모방하기 때문에 무작위로 데이터를 모은 것이 아니라, 개념이나 집합처럼 서로 '관련된 것'들을 묶어 놓은 것이 테이블이다.
## 테이블의 설계
- 테이블은 **가장 상위 개념의 집합**으로 묶는 것이 바람직하다. 

    테이블의 설계의 자유도는 무한의 조합이 가능하기 때문에 보통 상위 개념의 집합으로 테이블을 정의하고 WHERE 절을 사용하여 조건에 맡는 데이터를 조회함
- 열은 개체의 **속성(attribute)** 이다.

    객체지향의 관점에서 테이블(메서드 없는)-class, 행-instance
- 반드시 **기본키**가 있어야한다.
    - 중복을 방지하기 위해
    - 모든 기본키는 유일해야한다.
    - 데이터는 동적이다. 장기적으로 기본키의 유일성을 확보해야한다.
    - 테이블을 작성할 때 기본키는 <U>밑줄</U>을 긋는다.

## 정규형
- 데이터 갱신이 발생해도 부정합이 일어나기 어려운 상태
- 1~5 정규형이 있으나, 실무에서는 1~3 정규형까지만 알아도 충분
### 제1정규형(1NF)
테이블 셀에 복합적인 값(ex. 배열)을 넣지 않는다.(=스칼라 값을 제외한 값을 테이블에 넣지 않는다)

#### 왜 필요할까?
앞에서 테이블은 함수와 같다고 했다. 테이블은 인풋을 주면 하나의 아웃폿이 나와야한다. 테이블 셀에 복합적인 값이 들어가면 인풋(=기본키)을 주었을 때, 열값 전체가 하나의 고유한 아웃풋으로 나올 수 없다. 기본키와 열사이에 성립하는 함수와 같은 유일성을 '함수종속'이라고 한다.

*더이상 나눌 수 없는 값을 스칼라값이라고 한다.
### 제2정규형(2NF)
모든 열이 기본키와 완전 함수 종속 관계에 있어야한다. 기본키의 일부에만 종속되는 열이 없어야한다. 

#### 왜 필요할까?
올바른 집합 단위로 테이블이 나눠지지 않았기 때문에 발생하는 문제로, 테이블의 여러 정보를 알고 있어야 행을 삽입할 수 았다는 문제가 있다. 알지 못하는 정보를 null이나 더미 값으로 채울 수있지만 바람직하지 못함. 예를 들어 주문 접수를 하는데 접수 정보 뿐만 아니라 회사 정보까지 넣어야하는데 정보가 부족한 경우.
또한 열에서 중복 저장되는 데이터가 잘못 저장되는 갱신이상이 발생할 수 있다.

참고 - [제2정규형 위키백과](https://ko.wikipedia.org/wiki/%EC%A0%9C2%EC%A0%95%EA%B7%9C%ED%98%95)

### 제3정규형(3NF)
1,2 정규형을 만족하고 기본키가 아닌 열간 함수종속이 일어나서는 안된다. 열간 함수 종속이 나타나는 것을 추이함수종속이라고 한다.

## ER(Entity-Relationship) 다이어그램
- 객체(테이블)간의 관계를 그래픽으로 표현하여 쉽게 이해할 수 있게 함
- IE표기법
</br>
</br>

## 참조
- 데이터베이스 첫걸음 - 미크, 기무라 메이지(한빛미디어)

- [제2정규형 위키백과](https://ko.wikipedia.org/wiki/%EC%A0%9C2%EC%A0%95%EA%B7%9C%ED%98%95)