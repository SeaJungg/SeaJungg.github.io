---
layout: post
title: "docker container network 설정하면서 만난 에러들"
category: done
---


### 현상 파악
- redis에 붙어야 하는 특정 컨테이너를 띄우는 중
- `localhost`, `127.0.0.1`, `컨테이너 이름` 등을 넣어봐도 redis를 찾지 못하고 죽는다.
```cmd
sudo docker run -p 8081:8081 --env-file env.list retrieval
```
![죽는장면](..\assets\image\docker1.png)
```cmd
During handling of the above exception, another exception occurred:
...
redis.exceptions.ConnectionError: Error -2 connecting to redis-stack-server:6379. -2.
```

### 문제 정의
- 컨테이너들끼리는 격리되어 있으니 당연히 서로를 찾을 수가 없다.
  - 나는 호스트니까 redis-cli를 로컬호스트에 연결이 되는 건데, 각 컨테이너 `내부` 로컬호스트에는 있을 리가 없으니...

### 결론
- 네트워크 설정 확인하고, 필요하다면 하나 만들고, --network 옵션으로 같은 네트워크 안에 있게끔 하는 작업이 필요.

### 조치 내용
- 도커 현재 네트워크 확인
```cmd
sudo docker network ls
```
![sudo docker network ls](..\assets\image\docker2.png)
- bridge, host, none 은 기본 생성되어 있는 네트워크들.
- 나만의 네트워크를 만든다. 이름은 sub1로 지정
```cmd
sudo docker network create sub1
```

- 네트워크 상세 내용을 확인
```cmd
sudo docker network inspect sub1
```
- CIDR로 서브넷 배정된 것 확인
- 실제로 ip가 몇개인지 확인하려면 : https://www.ipaddressguide.com/cidr
- "Subnet": "172.18.0.0/16"은 172.18.0.0부터 172.18.255.255를 이 컨테이너의 내부 IP로 사용하겠다는 의미
![cidrtorange](..\assets\image\cidrtorange.png)



- container들의 이름 확인
```cmd
sudo docker ps
```
![sudo docker ps](..\assets\image\docker3.png)

- sub1에 내 컨테이너들을 연결
```cmd
sudo docker network connect sub1 redis-stack-server
sudo docker network connect sub1 sweet_elion
```
- 연결 후에 sub1 상태 확인
```cmd
sudo docker network inspect sub1
```
![sudo docker network inspect sub1](..\assets\image\docker4.png)
- Containers에 나의 컨테이너 두 개가 잘 연결되었다.

### 띄워보자.
- 이제 원래 띄우려던 컨테이너도 이 네트워크를 바라보도록 설정값을 주면서 띄워보려 했으나 에러 발생...
![error](..\assets\image\docker4.png)
```cmd
docker: Error response from daemon: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "--network": executable file not found in $PATH: unknown.
```
- 커맨드 키 순서를 변경해서 다시 실행
```
sudo docker run -p 8081:8081 --env-file env.list --network sub1 --name retrieval2 retrieval
```
[스택오버플로우 참고 내용](https://stackoverflow.com/questions/58581661/sql-server-in-docker-network-executable-file-not-found-in-path-unknown)


### 성공 후에 다시 에러.. 

![error](..\assets\image\docker5_1.png)
```
redis.exceptions.AuthenticationError: AUTH <password> called without any password configured for the default user. Are you sure your configuration is correct?
```
- env.list 파일에 비밀번호를 잘 넣어놨으나 redis를 실행할 때 비번 지정을 안했던 기억이.... 
- 혹시나 확인해보니 역시나다. 컨테이너에서 나던 에러가 redis client에서 정확하게 난다.
![error](..\assets\image\docker6.png)
- docker cluster 생성 시에 설정할 수 있었는데.. 뒤늦게라도 해주자..
- 나의 비밀번호는 ISMS걸리기 딱 좋은 `superpass`
- [Redis default user 이름은 `default`](https://redis.io/commands/auth/)
```
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) ""
127.0.0.1:6379> config set requirepass superpass
OK
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) "superpass"
127.0.0.1:6379> exit
```

### 정리하고 다시 실행
- 같은 컨테이너명으로 이제는 실행이 안된다. 이왕 이렇게 된 거 뜨다 만 실패작들을 정리하자.
```
sudo docker ps -a
```
- 위 명령어로 뜨다만 컨테이너들을 확인하고, 아래 명령어로 지워준다.
```
sudo docker rm ${CONTAINER ID 또는 NAME}
```
```
sudo docker run -p 8081:8081 --env-file env.list --network sub1 --name retrieval2 retrieval
```
- 정상 동작 완료.