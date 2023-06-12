# Scanner와 BufferedReader

파일을 읽을 때 사용하는 두 자바 클래스를 내가 필요한 부분만 살펴보자

## Scanner
Scanner는 간단한 scanner로 원시타입이나 문자열을 정규 표현식을 통해 parse한다. Scanner는 들어오는 입력값을 delimiter pattern에 따라(기본값은 space) token들로 쪼갠다. 그 쪼개진 token들은 다양한 next 메서드(next, nextInt, nextLong, nextLine...)에 따라 다른 값들로 변환된다.

### next() hasNext()
이 메서드들과 짝꿍인 원시타입의 메서드(hasNextInt, nextInt...)는 어떤 순서로 동작하는가?

1. 먼저 delimiter 패턴과 일치하는 input을 skip한다.
2. 그리고 그 다음 tokem을 반환하려고 시도한다.
3. 만약 input이 없다면, hasNext와 next 메서드는 입력이 들어올 때까지 block한다. 

그래서 Scanner 생성자에 System.in과 같은 InputStream을 입력하면, hasNext는 늘 true일 것이다. 입력값이 없다면 들어올 때까지 block하는 기능의 메서드이기 때문이다. 반면 File Scanner의 source로 입력하면, 파일의 마지막 줄 이나 마지막 글자 이후에는 hasNext는 false가 된다.