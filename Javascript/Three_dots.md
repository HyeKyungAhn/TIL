# Rest parameter and Spread Operators

## Rest Parameter

```
  function sum(...args){
    let total = 0;
    for(const arg of args){
      total += arg;
    }

    return total;
  }
```

- ES6~
- 함수가 정해지지않은 수의 인자(argument)를 배열 형태로 받을 수 있게함
- 함수의 마지막 파라미터만 접두사로 (...)를 붙일 수 있음(파라미터가 하나일 경우에도 처음이자 마지막이니 붙일 수 있다)
- 함수는 단 하나의 rest parameter만 가질 수 있다
- 함수의 length 속성으로 rest parameter의 길이는 알 수 없다
  - 함수의 length 속성으로 파라미터의 갯수를 알 수 있음
- arguments Object vs rest parameter <!-- TODO : arguments Object 공부 -->

## Spread Operators

```
  function sum(x, y, z) {
    return x + y + z;
  }

  const numbers = [1, 2, 3];

  console.log(sum(...numbers));
```

- ES6~
- iterable한 배열이나 문자열을 펼쳐서(expand) 내부 요소들을 각각 사용할 수 있게 함
- argument list에서 어떤 argument든 spread 문법을 사용하 수 있고, 여러 번도 사용 가능
- 함수의 apply() 를 대체 가능
- 객체나 배열의 모든 요소를 새로운 객체나 배열에 포함시키거나 호출하는 함수의 인자에 하나씩 적용해야할 때 사용

## rest syntax vs. spread syntax

- rest syntax는 spread syntax와 정 반대의 면을 가지고 있음. Spread 문법은 예를 들어 여러 값을 가지고 있는 하나의 배열을 펼쳐서(spread into) 각 요소로 나누지만, Rest 문법은 각 요소들을 하나의 배열로 응축시킨다.
