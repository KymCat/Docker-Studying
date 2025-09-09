# 🐋 도커 컨테이너 명령어
### 🐟 컨테이너는 프로세스다
도커 컨테이너는 도커 이미지를 기반으로 만들어지는 스냅숏이다. 이 스냅숏은 읽기 전용의 도커 이미지 레이어를 복제한 것이고, 그 위에 읽고 쓰기가 가능한 컨테이너 레이어를 결합하면 컨테이너가 된다. `컨테이너는 격리된 공간에서 프로세스가 동작하는 기술이다.`라는 말을 들어 보았을 것이다. 명령어 `dokcer run`을 사용하면 컨테이너가 동작하게 되고, 가상의 격리 환경에 독립된 프로세스가 동작한다. 마치 서버 호스트 운영체제가 독립적으로 동작하는 것과 유사하다.  

전통적인 리눅스 호스트 운영체제를 부팅하면 PID 1번은 init(현대는 systemd) 프로세스가 동작하며 이 포로세스는 나머지 모든 시슽메 프로세스의 부모 프로세스가 된다. 그런데 도커 컨테이너에서도 PID 1번 프로세스가 init일까? 

```bash
# 현재 호스트에서 실행 중인 쉘 프로세스 PID
$ echo $$
2957

# centos 8버전 이미지 다운로드 컨테이너 생성 후 bash 모드로 실행
$ docker run -it centos:8 bash
[root@컨테이너ID /]$ echo $$
1 # 컨테이너 첫 쉘은 PID 1번임 : 1번이 반드시 init은 아니다 !
[root@컨테이너ID /]$ exit
exit
```

`docker run -it 이미지 bash`로 컨테이너를 생성 후 bash 모드로 실행시킨 뒤 프로세스 ID를 확인 할 결과 1번이 출력되었다. 이는 도커 컨테이너의 1번 프로세스는 반드시 init이 아닐 수도 있다는 결과에 도달했다.

##
### 🐟 컨테이너 실행
컨테이너 실행을 위해 `docker run` 명령을 사용하면 해당 도커 이미지 스냅숏 레이어 위에 읽고 쓰기가 가능한 컨테이너 레이어를 추가한 뒤 `docker start` 명령으로 컨테이너를 시작한다. 이렇게 실행된 컨테이너를 조회하는 방법은 `docker ps` 명령을 사용하는 것이다.
<details>
    <summary>[실습]</summary>

```bash
# docker create : 컨테이너 생성 명령어
# --name > : 생성 할 컨테이너 이름
$ docker create -it --name test ubuntu:14.04

# test 컨테이너 실행
$ docker start test

# 실행 중인 컨테이너 조회
$ docker ps
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS         PORTS     NAMES
b50a90ab3cd4   ubuntu:14.04   "/bin/bash"   33 seconds ago   Up 4 seconds             test

# 컨테이너 접속
# docker attach container : 실행중인 컨테이너에 내 터미널을 attach 하는 명령어
# exit를 하면 컨테이너도 종료된다.
$ docker attach test
root@컨테이너ID:/$ ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

root@컨테이너ID:/$ exit
exit
```

위의 작업을 docker run으로 수행하면 다음과 같다.
```bash
$ docker run -it --name test ubuntu:14.04 bash
root@d116649d5f60:/# exit
exit
```
docker run의 특징은 호스트 서버에 이미지가 다운로드 되어 있지 않아도 로컬에 존재하는 이미지를 도커 허브에서 자동으로 다운로드를 해주고 컨테이너 생성 후 실행까지 함께 처리된다는 점이다.
> docker run = [pull] + create + start + [command]

</details>

##
### 🐟 컨테이너 재시작 및 일시정지
```bash
docker pause 컨테이너   # 컨테이너 일시정지
docker unpause 컨테이너 # 컨테이너 일시정지 해제
docker restart 컨테이너 # 컨테이너 재시작
```
- ***docker pause/unpause***  
    docker pause 명령은 지정된 컨테이너의 모든 프로세스를 일시 중단한다. 프로세스를 중지할 때 SIGSTOP 신호가 사용된다.

- ***docker restart***  
    컨테이너 재시작은 기존 컨테이너 프로세스를 정지하고 새로운 컨테이너 프로세스를 시작하는 것이다. 컨테이너 동작에는 영향이 없고 호스트의 PID만 변경

<details>
    <summary>[실습]</summary>

```bash
# webserver1 컨테이너 일시정지
$ docker pause webserver1
webserver1

# STATUS 열에 (Paused) 됨
$ docker ps
CONTAINER ID   IMAGE        COMMAND                  CREATED          STATUS                   PORTS                  NAMES
aa922c39f991   nginx:1.18   "/docker-entrypoint.…"   36 minutes ago   Up 17 seconds (Paused)   0.0.0.0:8081->80/tcp   webserver1

# webserver1 컨테이너 일시정지 해제
$ docker unpause webserver1
webserver1

# (Paused) 사라짐
$ docker ps
CONTAINER ID   IMAGE        COMMAND                  CREATED          STATUS          PORTS                                     NAMES
aa922c39f991   nginx:1.18   "/docker-entrypoint.…"   37 minutes ago   Up 46 seconds   0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   webserver1

# HOST PID 4273 / 
$ ps -ef | grep 8081
com       4273  1528  0 19:04 pts/5    00:00:00 grep --color=auto 8081

# webserver1 컨테이너 재시작
$ docker restart webserver1
webserver1

# HOST PID 4273 -> 4290
$ ps -ef | grep 8081
com       4290  1528  0 19:04 pts/5    00:00:00 grep --color=auto 8081
```
</details>

##
### 🐟 실습 : Ngnix 컨테이너 실행
<details>
    <summary>[Ngnix 실습]</summary>
    
```bash
# nginx 1.18 버전 다운로드
$ docker pull nginx:1.18
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
nginx        1.18      e90ac5331fe0   4 years ago     200MB

# 컨테이너 실행
# -d: 컨테이너를 백그라운드에서 실행하고 컨테이너 ID 출력
# -p : 컨테이너의 해당 포트를 HOST 해당 포트로 오픈 (호스트:컨테이너)
$ docker run --name webserver1 -d -p 8081:80 nginx:1.18
aa922c39f991206049030dc5d18770d08d67067af91de41779532f650171c1d2

# 실행 중인 컨테이너 조회
$ docker ps
CONTAINER ID   IMAGE        COMMAND                  CREATED         STATUS         PORTS                                     NAMES
aa922c39f991   nginx:1.18   "/docker-entrypoint.…"   4 seconds ago   Up 3 seconds   0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   webserver1

# localhost:8081 HTTP 요청
$ curl localhost:8081
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

# nginx 컨테이너의 접근 로그 확인 (-f : 실시간, -t: 마지막 로그까지)
$ docker logs -f webserver1
...
172.17.0.1 - - [08/Sep/2025:09:29:49 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/8.5.0" "-"
172.17.0.1 - - [08/Sep/2025:09:31:16 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36" "-"

# 컨테이너 정지
$ docker stop webserver1
webserver1

# localhost:8081 HTTP 요청 : 컨테이너를 정지했으므로 실패
$ curl localhost:8081
curl: (7) Failed to connect to localhost port 8081 after 0 ms: Could not connect to server

# nginx 컨테이너 실행
$ docker start webserver1
webserver1

# vi 편집기로  index.html 내용을 변경
$ vi index.html
<h1>Hello world!</h1>

# 도커 cp 명령을 통해 컨테이너 내부 index.html 파일 경로에 복사
$ docker cp index.html webserver1:/usr/share/nginx/html/index.html

# 변경된 내용 확인
$ curl localhost:8081
<h1>Hello World!</h1>
```

</details>

##
### 🐟 도커 볼륨
도커는 유니언 파일 시스템을 사용하여 하나의 이미지로부터 여러 컨테이너를 만들 수 있는 방법을 제공하고, 이미지에 변경된 내용을 저장할 수 있도록 해준다. 데이터베이스, 웹 프로그램 등 업무에서 사용하는 애플리케이션에서 발생하는 데이터에 접근하고 이것을 공유하기 위해서 `도커 볼륨` 기능을 사용할 수 있다.  
  
일반적으로 컨테이너 내부의 데이터는 컨테이너의 라이프사이클과 연관되어 컨테이너 종료시 삭제된다. 이를 영속적으로 유지하기 위한 방법으로 도커 볼륨을 사용하면 컨테이너가 삭제되어로 볼륨은 독립적으로 운영되기 때문에 함께 삭제되지 않는 특징이 있다.

#### **도커 볼륨 타입**
- ***volume***  
    도커에서 권장하는 방법으로 `docker volume create 볼륨이름`을 통해 볼륨을 생성하고 여러 컨테이너 간에 안정하게 공유가능  
    <details>
        <summary>[volume 실습]</summary>  
      
    ```bash
    # 볼륨 생성
    $ docker volume create my-vol
    my-vol

    # 생성된 볼륨 조회
    $ docker volume ls
    DRIVER    VOLUME NAME
    ...
    local     my-vol

    # 볼륨 검사, 볼륨이 올바르게 생성되고 마운트 되었는지 확인하는 데 사용
    $ docker volume inspect my-vol
    [
        {
            "CreatedAt": "2025-09-09T05:31:13Z",
            "Driver": "local",
            "Labels": null,
            "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
            "Name": "my-vol",
            "Options": null,
            "Scope": "local"
        }
    ]

    # --mount 옵션을 이용한 볼륨 지정
    # type : [volume, bind, tmpfs] 타입 지정
    # source : 볼륨이름, 경로(bind)
    # target : 컨테이너 안에서 마운트될 경로
    $ docker run -d --name vol-test1 \
    > --mount source=my-vol,target=/app \ # 띄어쓰기 금지
    > ubuntu:20.04
    Unable to find image 'ubuntu:20.04' locally
    ...

    # -v 옵션을 이용한 볼륨 지정
    # -v [경로,볼륨이름]:[컨테이너 마운트 경로]:[OPT]
    # [OPT] : ro,rw (읽기, 읽기쓰기<디폴트>)
    $ docker run -d --name vol-test2 \
    > -v my-vol:/var/log \ # my-vol 볼륨을 /var/log에 연결
    > ubuntu:20.04

    # docker volume create를 하지 않아도 호스트 볼륨 이름을 쓰면 자동 생성
    docker run -d --name vol-test-3 \
    > --mount source=my-vol-2,target=/var/log \
    > ubuntu:20.04

    $ docker volume ls
    DRIVER    VOLUME NAME
    ...
    local     my-vol
    local     my-vol-2 # docker volume create 하지 않은 볼륨

    # 컨테이너 Mount 필드 상세정보만 출력
    # [{볼륨타입 | 볼륨이름 | 호스트내 데이터 저장 경로 | 컨테이너내 마운트 경로 | 볼륨 드라이버 | SELinux 옵션 | 읽기쓰기 여부}]
    docker inspect --format="{{.Mounts}}" vol-test1
    [{volume my-vol /var/lib/docker/volumes/my-vol/_data /app local z true }]

    # 볼륨 제거, 연결된 컨테이너가 있으면 아래와 같은 에러 발생
    $ docker volume rm my-vol
    Error response from daemon: remove my-vol: volume is in use - ...

    # 컨테이너 중지
    $ docker stop vol-test1 vol-test2
    vol-test1
    vol-test2

    # 컨테이너 제거
    $ docker rm vol-test1 vol-test2
    vol-test1
    vol-test2

    # 볼륨 제거
    $ docker volume rm my-vol
    my-vol
    ```

    </details>  