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

### 2️⃣ bind
- 도커 볼륨 기법에 비해 사용이 제한적
- `호스트 파일 시스템 절대경로:컨테이너 내부경로`를 직접 마운트하여 사용
- 사용자가 파일 또는 디렉토리를 생성하면 해당 호스트 파일 시스템의 소유자 권한으로 연결
- 존재하지 않는 경우 자동 생성, 자동 생성된 디렉토리의 소유자는 root
- 컨테이너 실행 시 지정하여 사용하고, 컨테이너 제거 시 바인드 마운트는 해제되지만 호스트 디렉터리는 유지
    <details>
        <summary>[bind 실습]</summary>  

    ```bash
    # 현재 디렉토리 아래에 target 디렉토리 생성
    $ mkdir $(pwd)/target

    # 사전 생성한 디렉토리와 bind 마운트 지정
    $ docker run -d -it --name bind-test1 \
    > --mount type=bind,source="$(pwd)"/target,target=/var/log \
    > centos:8
    Unable to find image 'centos:8' locally
    ...

    # 사전 생성하지 않은 디렉토리와 bind 마운트 지정
    $ docker run -d -it --name bind-test2 \
    > -v "$(pwd)"/target2:/var/log \
    > centos:8
    
    # 사전 생성 target 소유자 : 호스트
    # 자동 생성 target2 소유자 : root
    $ ls -l
    ...
    drwxr-xr-x 2 com  com       4096 Sep  9 16:11 target
    drwxr-xr-x 2 root root      4096 Sep  9 16:16 target2
    ...
    ```
    </details>

### 3️⃣ tmpfs
- 컨테이너 실행시 메모리(RAM)위에 임시 파일시스템을 만들어 사용되는 방법
- 임시적으로 만들기 때문에 컨테이너가 제거되면 자동으로 메모리에서 해제
- 호스트 메모리에서만 지속되므로 내부 기록된 파일은 유지되지 않음
- 호스트 또는 컨테이너 쓰기 가능 계층에서 지속하지 않지만 중요한 파일들 임시로 사용하는 방법에 유용
    <details>
        <summary>[bind 실습]</summary>  

    ```bash
    # --mount 옵션으로 tmpfs 마운트
    # 메모리기반 임시 파일 시스템이라서 source가 없음
    $ docker run -d -it --name tmpfs-test1 \
    > --mount type=tmpfs,destination=/var/www/html \
    > httpd:2

    # --tmpfs 옵션으로 tmpfs 마운트
    $ docker run -d -it --name tmpfs-test2 \
    > --tmpfs /var/www/html \
    > httpd:2

    # tmpfs로 마운트된 컨테이너 상세정보 조회
    $ docker inspect --format="{{.HostConfig.Tmpfs}}" tmpfs-test2
    map[/var/www/html:]
    ```
    </details>