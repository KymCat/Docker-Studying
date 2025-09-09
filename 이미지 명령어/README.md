# 🐋 도커 이미지 명령어
## 🐟 도커 이미지 내려받기
- ***docker pull***  
    도커 허브에서 로컬로 도커 이미지 내려받기
- ***docker push***  
    로컬에 있는 도커 이미지를 도커 허브에 업로드
- ***docker login***  
    업로드를 하기 전 도커 허브 계정으로 로그인 수행
- ***docker logout***  
    도커 허브에서 로그아웃
    
    ```bash
    # 배포판 리눅스 이미지 debian 다운로드
    # docker [image] pull [OPT] name[:TAG | @IMAGE_DIGEST]
    $ docker pull debian # [:태그]를 포함하지 않으면 최신버전 다운(latest)
    Using default tag: latest
    ... 생략

    # 다운로드한 이미지 조회 방법들
    $ docker images
    # [이미지명]  [태그]     [이미지 아이디]  [이미지 작성일] [이미지 크기]
    REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
    debian       latest    6d8737501634   10 days ago   183MB

    $ docker image ls
    REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
    debian       latest    6d8737501634   10 days ago   183MB
    ```

## 🐟 도커 이미지 세부 정보 조회 [ ***docker image inspect*** ]
```bash
$ docker image inspect [OPT] IMAGE [IMAGE...]
```
- docker image inspect 명령 옵션
    - ***--format, -f***  
        JSON 형식의 정보 중 지정한 형식의 정보만 출력 가능  
        중괄호`{}` 형식과 대소문자에 유의
    
```bash
# docker search 수행 전 도커 허브 계정으로 docker login 필요
$ docker search httpd
NAME                     DESCRIPTION                                     STARS     OFFICIAL
httpd                    The Apache HTTP Server Project                  4898      [OK]
... 생략

# httpd 이미지 다운로드
$ docker pull httpd # [:태그] 지정이 없어 latest 버전 다운로드
Using default tag: latest 
... 생략

# 도커 이미지 조회
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
httpd        latest    3198c1839e1a   12 days ago   174MB

# 도커 이미지 세부 정보 조회
$ docker image inspect httpd
[
    {
        "Id": "sha256:3198c1839e1a875f8b83803083758a7635f1ae999f0601f30f2f3b8ce2ac99e3",
        "RepoTags": [
            "httpd:latest"
        ],
        "RepoDigests": [
            "httpd@sha256:3198c1839e1a875f8b83803083758a7635f1ae999f0601f30f2f3b8ce2ac99e3"
        ],
        "Parent": "",
        "Comment": "buildkit.dockerfile.v0",
        "Created": "2025-08-08T19:04:23Z",

        ... 생략

    }
]

# httpd 세부 정보에서 RepoTags필드 값 조회
$ docker image inspect --format="{{.RepoTags}}" httpd
[httpd:latest]
```

## 🐟 도커 이미지 세부 정보 조회 [ ***docker image history*** ]
```bash
$ docker image history [OPT] IMAGE
```
현재 이미지 구성을 위해 사용된 레이블 정보와 각 레이어의 수행 명령, 크기 등을 조회  
한 마디로 해당 이미지가 어떤 빌드 명령어를 통해 구성 되었는지 보여주는 기록  
  
```bash
$ docker image history httpd
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
3198c1839e1a   12 days ago   CMD ["httpd-foreground"]                        0B        buildkit.dockerfile.v0
<missing>      12 days ago   EXPOSE map[80/tcp:{}]                           0B        buildkit.dockerfile.v0
<missing>      12 days ago   COPY httpd-foreground /usr/local/bin/ # buil…   20.5kB    buildkit.dockerfile.v0
<missing>      12 days ago   STOPSIGNAL SIGWINCH                             0B        buildkit.dockerfile.v0

... 생략
```

## 🐟 도커 이미지 태그 설정
```bash
$ docker tag 원본 이미지[:태그] 참조 이미지[:태그]
```
태그 설정은 새로운 참조명(이름)을 붙이는 작업이므로 이미지 ID는 변경되지 않음

```bash
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
httpd        latest    3198c1839e1a   12 days ago   174MB

# httpd[:latest] -> my-httpd[:1.0] 변경
$ docker image tag httpd my-httpd:1.0 #[:태그] 생략 시 latest 지정
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
httpd        latest    3198c1839e1a   12 days ago   174MB
my-httpd     1.0       3198c1839e1a   12 days ago   174MB

# 도커 허브에 저장하기 위해서는 [본인아이디]/참조 이미지[:태그] 형태로 저장
$ docker image tag httpd:latest kymcat/hub-httpd:1.0
$ docker images
REPOSITORY         TAG       IMAGE ID       CREATED       SIZE
kymcat/hub-httpd   1.0       3198c1839e1a   12 days ago   174MB
httpd              latest    3198c1839e1a   12 days ago   174MB
my-httpd           1.0       3198c1839e1a   12 days ago   174MB
```

## 🐟 도커 허브 업로드 및 내려받기
```bash
$ docker push [본인아이디]/참조이미지[:태그] # 업로드
$ docker pull [본인아이디]/참조이미지[:태그] # 내려받기
```
도커 허브에 업로드, 내려받기 전에는 반드시 `$ docker login` 으로 도커 허브에 원격 접속 필요  


```bash
# 이미지 조회
$ docker images
REPOSITORY         TAG       IMAGE ID       CREATED       SIZE
kymcat/hub-httpd   1.0       3198c1839e1a   12 days ago   174MB
httpd              latest    3198c1839e1a   12 days ago   174MB
my-httpd           1.0       3198c1839e1a   12 days ago   174MB

# 도커 허브에 kymcat/hub-httpd:1.0 업로드
$ docker push kymcat/hub-httpd:1.0
The push refers to repository [docker.io/kymcat/hub-httpd]
d2b1a5ae8cd3: Mounted from library/httpd
4f4fb700ef54: Mounted from library/httpd
7e8bbac53823: Mounted from library/httpd
779ccd583397: Mounted from library/httpd
... 생략

# kymcat/hub-httpd:1.0 이미지 제거
$ docker rmi kymcat/hub-httpd:1.0
Untagged: kymcat/hub-httpd:1.0

# 이미지 조회 -> kymcat/hub-httpd:1.0 제거됨
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
httpd        latest    3198c1839e1a   12 days ago   174MB
my-httpd     1.0       3198c1839e1a   12 days ago   174MB

# 도커 허브에서 kymcat/hub-httpd:1.0 내려받기
$ docker pull kymcat/hub-httpd:1.0
1.0: Pulling from kymcat/hub-httpd
Digest: sha256:7bc639e75649411beb3f0a2cfd74713c3b9ed2ee9dde4a65bf41c89cb408f895
Status: Downloaded newer image for kymcat/hub-httpd:1.0
docker.io/kymcat/hub-httpd:1.0

# 이미지 조회 -> kymcat/hub-httpd:1.0 생김
$ docker images
REPOSITORY         TAG       IMAGE ID       CREATED       SIZE
kymcat/hub-httpd   1.0       7bc639e75649   12 days ago   174MB
httpd              latest    3198c1839e1a   12 days ago   174MB
my-httpd           1.0       3198c1839e1a   12 days ago   174MB
```

## 🐟 도커 이미지를 파일로 관리
```bash
$ docker image save [OPT] 파일명 [IMAGE] # 파일 저장
$ docker image load [OPT] # 파일 로드

# 리눅스 리다이렉션 활용
$ docker image save 참조이미지[:태그] > 파일명.확장자
$ docker image load < 파일명.확장자

# PowerShell
$ docker save -o 파일이름.확장자 이미지[:태그]
$ docker load -i 파일이름.확장자
```
- ***docker image save***  
    도커 원본 이미지의 레이저 구조까지 포함한 복제를 수행하여 `.tar` 파일로 이미지 저장

- ***.tar*** 파일  
    여러개의 파일을 하나의 파일로 묶거나 풀 때 사용하는 파일

- ***파일로 저장하는 이유***
    - 도커 허브로부터 이미지를 내려받아 내부망으로 이전하는 경우
    - 신규 애플리케이션 서비스를 위해 Dockfile로 새롭게 생성한 이미지를 저장 및 배포해야 하는 경우
    - 컨테이너를 commit 하여 생성한 이미지를 저장 및 배포해야 하는 경우
    - 개발 및 수정한 이미지 등

```bash
# mysql 5.7 version 다운로드
$ docker pull mysql:5.7
5.7: Pulling from library/mysql
... 생략

# 이미지 조회
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
mysql        5.7       4bc6bc963e6d   20 months ago   689MB

# mysql:5.7 이미지 -> save-mysql.tar 파일로 저장
$ docker save mysql:5.7 > save-mysql.tar

# 파일 조회
$ ls 
save-mysql.tar

# 이미지에서 mysql 5.7 제거
$ docker rmi mysql:5.7
Untagged: mysql:5.7
Deleted: sha256:4bc6bc963e6d8443453676cae56536f4b8156d78bae03c0145cbe47c2aad73bb

# 이미지 조회 -> mysql 5.7 제거
$ docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE

# save-mysql.tar 파일 -> 이미지
$ docker load < save-mysql.tar
Loaded image: mysql:5.7

# 이미지 조회 -> mysql 5.7 생김
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
mysql        5.7       4bc6bc963e6d   20 months ago   689MB

# 파일을 로드하면서 태그 변경하는 방법
$ cat save-mysql.tar | docker import - load-sql:1.0 # load-sql 1.0 태그 변경
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
load-sql     1.0       240f178eecce   6 seconds ago   274MB # 변경되면서 새롭게 저장
mysql        5.7       4bc6bc963e6d   20 months ago   689MB
```

## 🐟 도커 이미지 삭제
```bash
docker image rm [OPT] {이미지 이름[:태그] | 이미지ID} # 정식 명령
docker rmi [OPT] {이미지 이름[:태그] | 이미지ID} # 단축 명령
```

도커 이미지는 현재 사용 중인 컨테이너가 없으면 바로 삭제된다. 하지만 컨테이너가 실행 중인 이미지를 삭제한다면 에러가 발생하기에 구동중인 컨테이너를 stop한 뒤 rm을 통해 제거한 후 이미지 삭제가 가능하다.

```bash
# 이미지 조회
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
debian       latest    6d8737501634   2 weeks ago   183MB
ubuntu       14.04    7c06e91f61fa   3 weeks ago   117MB

# 이미지 삭제 시 latest 버전을 제외하면 나머지는 태그를 반드시 명시
$ docker rmi ubuntu
Error response from daemon: No such image: ubuntu:latest

$ docker rmi ubuntu:14.04 # 태그 명시 :14.04
Untagged: ubuntu:14.04
Deleted: sha256:64483f3496c1373bfd55348e88694d1c4d0c9b660dee6bfef5e12f43b9933b30

# 이미지 조회 -> ubuntu:14.04 제거 완료
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
debian       latest    6d8737501634   2 weeks ago   183MB

# -q 옵션은 이미지 ID만 보기
$ docker images -q
6d8737501634

# 리눅스 셸 스크립트의 변수 활용 방식으로 모든 이미지 제거
$ docker rmi $(docker images -q)
Untagged: debian:latest
Deleted: sha256:6d87375016340817ac2391e670971725a9981cfc24e221c47734681ed0f6c0f5

$ docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE

# 이미지 다운
$ docker pull ubuntu:14.04
$ docker pull ubuntu
$ docker pull debian

# ubuntu 이름이 포함된 이미지들 제거
$ docker rmi $(docker images | grep ubuntu)
Untagged: ubuntu:latest
Deleted: sha256:7c06e91f61fa88c08cc74f7e1b7c69ae24910d745357e0dfe1d2c0322aaf20f9
Untagged: ubuntu:14.04
Deleted: sha256:64483f3496c1373bfd55348e88694d1c4d0c9b660dee6bfef5e12f43b9933b30
...생략

# 이미지 조회 -> ubuntu 제거 완료
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
debian       latest    6d8737501634   2 weeks ago   183MB

# -a 옵션 : 컨테이너 사용 중이 아닌 모든 이미지 제거
$ docker image prune -a
WARNING! This will remove all images without at least one container associated to them.
Are you sure you want to continue? [y/N] y
... 생략

# 이미지 조회 -> 전부 삭제 완료
$ docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```