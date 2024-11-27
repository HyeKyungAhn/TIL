# redis container에서 사용자 정의 redis.conf 사용하기

container에서 실행되는 redis server에 비밀번호를 설정하는 방법 중 두가지가 있다.

하나는 실행할 떄마다 명령어 옵션으로 `redis-server --requirepass password`를 추가하는 것이고 다른 방법은 redis 이미지 생성 시 사용하는 dockerfile에 비밀번호 관련 정보가 작성되어있는 redis.conf 파일을 사용하는 것이다.

container를 실행할 때마다 옵션을 입력하는 불편함을 없애기 위해 두 번쨰 방법을 사용해보겠다.

## dockerfile 작성

```
FROM redis:7.2.5
COPY ./redis.conf /usr/local/etc/redis/redis.conf
EXPOSE 6379
CMD [ "redis-server", "/usr/local/etc/redis/redis.conf" ]
```

- `FROM` :이미지를 생성할 때 기존 redis 7.2.5 이미지를 사용한다
- `COPY` :localhost에 위치한 redis.conf 파일을 container 내의 위치에 복사한다
- `EXPOSE` : container에서 노출 할 port 번호
- `CMD` : container 실행 시 '/usr/local/etc/redis/redis.conf' 파일로 redis-server를 실행

## redis.conf

### 왜 redis.conf 파일을 복사해야할까?

이미지를 생성할 때 기존의 redis 이미지를 사용하는데 왜 redis.conf 파일을 외부에서 복사해서 실행할 때 사용해야할까? **redis docker image에는 config 파일이 없기 때문이다**. 기본 설정이 사용되기 때문에 이 설정을 변경하려면, container 실행 시 옵션을 주거나 redis.conf 파일을 외부에서 복사해서 실행 시 이 파일의 경로를 알려줘야한다.

### redis.conf 파일 살펴보기

[redis 설정 파일 예시](https://redis.io/docs/latest/operate/oss_and_stack/management/config-file/)에서 redis.conf 파일을 받아올 수 있다.

#### requirepass

```
requirepass [비밀번호]
```

Redis server의 비밀번호를 설정할 수 있다.

실제 파일에서는 `# requirepass foobared`의 주석을 해제하고 foobared 대신 원하는 비밀번호를 작성하자

#### bind

```
bind [IPv4주소] [IPv6주소]
```

-
- IPv4, IPv6 주소를 다르게 설정할 수 있다.

- 기본 설정값은 `bind 127.0.0.1 -::1`이다.

  - `127.0.0.1`은 로컬호스트(자기자신)을 의미하는 IPv4 주소이다. 즉 Redis 서버는 이 IP를 통해서만 접속을 허용한다는 의미. 호스트가 아닌 외부 네트워크에서 접속할 경우 차단된다.
  - `-::1`부분은 IPv6의 로컬 주소인 `::1`을 나타내는 것으로 IPv6에서 자기 자신을 가리키는 주소이다. `-`의 의미는 이 주소로부터의 접근을 허용하려는 의미이다.

  즉 기본 설정값은 로컬머신에서의 접근만 허용되고 외부 네트워크에서의 접근은 차단한다는 의미이다.

- `bind 0.0.0.0 ::1`

  - `0.0.0.0`은 모든 IPv4 주소에서의 접속을 허용한다는 의미(외부 접속 허용)
  - `::1`는 로컬 IPv6 주소로, 여전히 로컬 접속은 허용된다.

    IPv6는 외부 환경에서 덜 사용되며 주소 범위가 넓고 네트워크 구조가 복잡해 보안설정이 까다롭기 때문에 로컬 접속만 허용해서 불필요한 보안 위험을 줄이려는 목적으로 `::1`로 설정.
    외부 접속을 완전히 허용하려면 `::0`으로 설정하면 된다.

#### protected-mode

redis 인스턴스의 public IP가 노출되어있어 외부 네트워크의 접속에서 보호하기 위해 redis 3.2.0 부터 도입한 모드이다.

redis **초기 설정**에서 비밀번호는 설정되어있지 않고 protected-mode의 값은 yes으로 활성화되어있다. 이 모드에서 Redis는 loopback interface(127.0.0.1, ::1)의 쿼리에만 응답하고 다른 주소에서의 연결은 오류를 발생시킨다.

**외부 네트워크에서의 접근**을 허용하려면 `no`로 설정하고 `bind`에서도 loopback IP address가 아닌 외부 접속을 허용하는 주소로 변경해야한다.(반드시 두 개 다 변경해야한다)

#### loopback interface를 잠깐 살펴보자

물리적 네트워크 장치 없이 네트워크 통신을 모방할 수 있게하는 가상의 개념. 즉 컴퓨터가 자기 자신과 통신할 수 있는 가상의 네트워크를 제공한다.
컴퓨터 내부에서 네트워크 통신 소프트웨어 테스트나 처리를 위해 사용된다.
예를 들어 host의 브라우저에서 127.0.0.1 로 요청을 해서 마찬가지로 host에서 실행되는 web server에서 요청을 정상적으로 처리하고 응답하는지를 테스트하는데 사용할 수 있다.

loopback interface는 구성된 이후 항상 활성화되어있으며(비활성화도 가능) loopback address, loopback IP address라고 불리는 특수한 IP 주소(127.0.0.1, ::1)를 할당한다.

#### port

기본값은 6379이다.

상황에 따라 최소한은 위의 설정을 변경한 뒤에 image 생성할 때 redis.conf 파일을 사용하자.
