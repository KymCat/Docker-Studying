# 🐋 도커 볼륨 명령어
## 🐟 도커 볼륨
도커는 유니언 파일 시스템을 사용하여 하나의 이미지로부터 여러 컨테이너를 만들 수 있는 방법을 제공하고, 이미지에 변경된 내용을 저장할 수 있도록 해준다. 데이터베이스, 웹 프로그램 등 업무에서 사용하는 애플리케이션에서 발생하는 데이터에 접근하고 이것을 공유하기 위해서 `도커 볼륨` 기능을 사용할 수 있다.  
  
일반적으로 컨테이너 내부의 데이터는 컨테이너의 라이프사이클과 연관되어 컨테이너 종료시 삭제된다. 이를 영속적으로 유지하기 위한 방법으로 도커 볼륨을 사용하면 컨테이너가 삭제되어로 볼륨은 독립적으로 운영되기 때문에 함께 삭제되지 않는 특징이 있다.


## 🐟 도커 볼륨 타입
### 1️⃣ volume
- 도커에서 권장하는 방법 
- `docker volume create 볼륨이름`을 통해 볼륨을 생성
- 여러 컨테이너 간에 안전하게 공유가능  

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