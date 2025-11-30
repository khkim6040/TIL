# 도커 파일시스템 원리

도커 파일시스템은 일반적인 파일 시스템 경로가 아니라 알아보기 힘들게 되어있다.

```sh
admin@Gwanhos-MacBook-Air data % pwd
/Users/admin/Library/Containers/com.docker.docker/Data/vms/0/data
admin@Gwanhos-MacBook-Air data % ls
Docker.raw
admin@Gwanhos-MacBook-Air data % cd ..
admin@Gwanhos-MacBook-Air 0 % ls
00000002.000007cf	console.sock		log
00000002.00001003	data			macaddr-0
```

왜 이런 방식을 채택하는 것일까? 이는 **효율성**과 **데이터 무결성**을 위한 방법으로, '콘텐츠 주소 지정 방식'이라고 한다.

## 콘텐츠 주소 지정 방식(Content-Addressable Storage)
이름과 경로로 구성되는 일반적인 파일 시스템과 달리 도커는 파일의 **내용** 자체를 가지고 해시값을 만들고 이를 주소로 이용한다. 이 방법은 몇 가지 장점이 있는데,

### 1. 완벽한 데이터 중복 제거 (Perfect Data Deduplication)

동적 라이브러리의 원리와 비슷한데, 중복되어 쓰이는 데이터를 하나만 두는 것이다. 

두 개의 이미지가 똑같은 베이스 이미지인 `ubuntu:22.04`를 사용한다고 해보자. 여기서 똑같은 라이브러리 파일(`libcrypto.so`)을 각기 다른 경로에 추가했다고 가정해보자.

도커 파일시스템이 이름 기반으로 저장한다면 각 이미지 별로 똑같은 내용의 `libcrypto.so`이 디스크에 두 번 저장된다.

해시값 기반 방식을 사용한다면, `libcrypto.so`의 내용을 해시해 유일한 ID(`sha256:5f22...`)를 얻는다. 이 값을 이름으로, 실제 `libcrypto.so`는 단 한 번만 저장한다. 처음 만든 두 개의 이미지 레이어는 `libcrypto.so`의 실제 데이터를 저장하지 않고 이 파일의 포인터(`sha256:5f22...`)를 기록해 둔다. 나중에 이 포인터를 이용해 파일을 가져온다.

### 2. 강력한 데이터 무결성 보장 (Strong Data Integrity Guarantee)

해시값은 데이터의 '지문'으로, 파일 내용이 단 1비트라도 바뀌면 해시값은 완전히 달라진다. 이를 이용해 보안과 신뢰성을 보장할 수 있다.

- 보안: `docker pull`로 이미지를 받아올 때 도커는 각 데이터 조각(레이어)의 해시값을 계산해 이미지 Manifest에 기록된 값과 비교한다. 만약 값이 다르면 전송 중 데이터가 손상되었거나 누군가 악의적으로 변조했다는 것이 된다. 
```sh
admin@Gwanhos-MacBook-Air 0 % docker manifest inspect hello-world
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.oci.image.index.v1+json",
   "manifests": [
      {
         "mediaType": "application/vnd.oci.image.manifest.v1+json",
         "size": 1035,
         "digest": "sha256:2771e37a12b7bcb2902456ecf3f29bf9ee11ec348e66e8eb322d9780ad7fc2df", # 이 값을 보고 비교한다.
         "platform": {
            "architecture": "amd64",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.oci.image.manifest.v1+json",
         "size": 566,
         "digest": "sha256:6b75187531c5e9b6a85c8946d5d82e4ef3801e051fbff338f382f3edfa60e3d2",
         "platform": {
            "architecture": "unknown",
            "os": "unknown"
         }
      },
...
```

- 신뢰성: 이미지의 특정 버전(예: `ubuntu:22.04`의 이미지 ID `sha256:1234...`)은 그 내용을 구성하는 모든 레이어의 해시값으로 결정된다. 따라서 ID가 같다면 내용물도 100% 같다고 보장할 수 있다.


### 3. 불변성(Immutability)과 안정성 (Stability)

기존 레이어를 '수정'하는 것은 불가능하고, 변경 사항을 담은 새로운 레이어가 그 위에 생성된다.

이는 Git의 작동 방식과 유사한데, 커밋을 수정하는 것이 아니라 새로운 커밋을 만드는 것처럼, 도커 레이어도 한번 생성되면 절대 변하지 않는다. 이러한 불변성 덕분에 이미지의 버전을 안정적으로 관리하고, 어떤 환경에서도 동일한 컨테이너가 실행될 것을 보장할 수 있게 된다.

## 결론

도커 파일시스템은 사람이 읽기 좋은 구조 대신 자기가 읽기 좋게 만들어서 저장 공간 효율성, 데이터 무결성 및 안정성을 얻었다.

`docker images`, `docker history`, `docker inspect` 같은 API로 필요하다면 사람이 읽을 수 있게 정보를 제공해준다.

```sh
admin@Gwanhos-MacBook-Air 0 % docker images
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
jekyll-site   latest    e73b34001e00   4 days ago    1.85GB
ubuntu        latest    9cbed7541129   5 weeks ago   117MB
ubuntu        18.04     152dc042452c   2 years ago   97.5MB

admin@Gwanhos-MacBook-Air 0 % docker history ubuntu:latest
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
9cbed7541129   5 weeks ago   /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      5 weeks ago   /bin/sh -c #(nop) ADD file:e67907c77897d2719…   87.6MB    
<missing>      5 weeks ago   /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B        
<missing>      5 weeks ago   /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B        
<missing>      5 weeks ago   /bin/sh -c #(nop)  ARG LAUNCHPAD_BUILD_ARCH     0B        
<missing>      5 weeks ago   /bin/sh -c #(nop)  ARG RELEASE                  0B        

admin@Gwanhos-MacBook-Air 0 % docker inspect ubuntu:latest
[
    {
        "Id": "sha256:9cbed754112939e914291337b5e554b07ad7c392491dba6daf25eef1332a22e8",
        "RepoTags": [
            "ubuntu:latest"
        ],
        "RepoDigests": [
            "ubuntu@sha256:9cbed754112939e914291337b5e554b07ad7c392491dba6daf25eef1332a22e8"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2025-08-19T14:37:01.23577114Z",
        "DockerVersion": "",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "Image": "",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.opencontainers.image.ref.name": "ubuntu",
                "org.opencontainers.image.version": "24.04"
            }
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 7112,
        "GraphDriver": {
            "Data": null,
            "Name": "overlayfs"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:9d592720ced4a7a4ddf16adef8a126e4c8c49f22114de769343320b37674321e"
            ]
        },
        "Metadata": {
            "LastTagTime": "2025-09-08T11:16:28.614846091Z"
        },
        "Descriptor": {
            "mediaType": "application/vnd.oci.image.index.v1+json",
            "digest": "sha256:9cbed754112939e914291337b5e554b07ad7c392491dba6daf25eef1332a22e8",
            "size": 6688
        }
    }
]

```