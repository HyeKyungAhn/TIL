# host에서 host에서 실행되는 container(redis server) 연결하기

container가 정상적으로 작동하는지 로컬에서 확인하기 위해 host 컴퓨터에서 동작하는 web application에서 docker container에 접근하는 방법을 알아보겠습니다.

이 글에서는 spring webapp(legacy)에서 redis server가 실행되는 container에 접근하는 방법을 알아보겠습니다.

## 포트 매핑 추가하기

컨테이너 실행 시 포트 매핑을 추가합니다. 포트매핑이란 컨테이너 포트를 외부에서 접근할 수 있도록 호스트의 포트를 컨테이너의 포트에 연결(매핑)하는 것을 말합니다.

### 포트매핑 형식

`host_port:container_port`

[예시] 6379:6379

=> 호스트의 포트 6379를 컨테이너 포트 6379와 연결. 외부에서 접근가능하도록 설정. 컨테이너 외부의 localhost:6379로 접근하면 내부 container 서버로 요청이 전달됨.
(host는 컨테이너를 실행하는 로컬 컴퓨터를 의미)

### 방법1. 실행 시 docker cli로 추가

`docker run -d --name redis_container -p 6379:6379 redis`

### 방법2. docker compose를 사용해 포트 매핑 설정하기

```
version: '3.8'

services:
  redis:
    image: redis
    container_name: redis_container
    ports:
      - "6379:6379" # 포트 매핑
```

## 포트 매핑을 확인하는 방법

### 방법1. `docker ps` 명령어 사용

```
[출력]
CONTAINER ID   IMAGE    COMMAND              CREATED        STATUS        PORTS                     NAMES
123abc456def   redis    "redis-server"       5 minutes ago  Up 5 minutes  0.0.0.0:6379->6379/tcp    redis_container

```

`0.0.0.0:6379->6379/tcp`

- 6379->6379

  컨테이너 내부의 6379 포트가 호스트의 6379 포트와 매핑되었다는 의미

- 0.0.0.0

  호스트의 모든 네트워크 인터페이스에서 접근 가능하다는 의미

### 방법2. `docker inspect` 명령어 사용

`docker inspect [container name]`

```
[출력]

"HostConfig": {
    "PortBindings": {
        "6379/tcp": [ //container의 port
            {
                "HostIp": "",
                "HostPort": "6379"  //container 6379와 매핑된 host의 6379 port
            }
        ]
    }
},
"NetworkSettings": {
    "Ports": {
        "6379/tcp": [
            {
                "HostIp": "0.0.0.0",
                "HostPort": "6379" //container 6379와 매핑된 host의 6379 port
            }
        ]
    }
}
```

### 방법3. 호스트에서 redis 서버에 직접 연결 테스트하기

#### `redis-cli`로 테스트

```
redis-cli -h localhost -p 6379
```

연결되었다면 다음과 같은 프롬프트가 표시됨

```
127.0.0.1:6379>
```

## web application에 container 정보 알려주기

spring 웹앱에서 redis server를 실행하는 contianer에 접근해보겠습니다. 따로 spring legacy의 redis 관런 설정은 따로 설명하지는 않겠습니다.

```
<bean id="redisStandaloneConfiguration" class="org.springframework.data.redis.connection.RedisStandaloneConfiguration">
    <constructor-arg name="hostName" value="${redis.hostname}" />
    <constructor-arg name="port" value="${redis.port}" />
    <property name="password">
        <bean class="org.springframework.data.redis.connection.RedisPassword" factory-method="of">
            <constructor-arg value="${redis.password}"/>
        </bean>
    </property>
</bean>

<bean id="lettuceConnectionFactory" class="org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory">
    <constructor-arg ref="redisStandaloneConfiguration" />
</bean>
```

${redis.hostname} = localhost

${redis.port} = 6379

${redis.password} = 개인비밀번호(생략가능)

변수를 위와 같이 설정합니다. 위의 예시에서는 application.properties 파일에 따로 값을 저장하고 값을 불러와 사용했습니다.
