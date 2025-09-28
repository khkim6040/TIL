## 도커 컨테이너를 중지해도 데이터는 정말 사라지지 않을까?

도커 컨테이너를 실행하고 내부에서 파일을 생성하거나 수정하는 작업을 하다가 `docker stop` 명령어로 컨테이너를 중지해도, 나중에 `docker start로` 다시 실행하면 작업 내용이 그대로 남아있는 것을 볼 수 있다. 컨테이너를 `docker rm`으로 완전히 삭제하지 않는 한 데이터는 보존된다.

난 항상 컨테이너의 데이터를 보존하려면 무조건 볼륨(Volume)을 잡아줘야 한다고 생각했다. 그래서 중요한 작업 내용이 날아갈까 봐 컨테이너를 일부러 끄지 않은 적도 있었다 ㅋ. 근데, 컨테이너를 종료해도 데이터가 유지된다면, 볼륨은 왜 필요한 걸까? 하나하나 알아가 보자.

### 저장 경로
결론부터 말하면, **컨테이너에서 변경된 내용은 호스트 머신의 디스크에 저장된다.** 그래서 그 컨테이너를 지우지 않는 한 도커 파일시스템에서 작업 내용을 유지해 컨테이너에서 작업을 계속할 수 있다. Mac의 경우 아래 경로에서 도커 파일시스템을 볼 수 있다. 컨테이너 변경 데이터가 여기에 보존되는 것이다.

```sh
admin@Gwanhos-MacBook-Air data % pwd
/Users/admin/Library/Containers/com.docker.docker/Data/vms/0/data
admin@Gwanhos-MacBook-Air data % ls
Docker.raw

```

하지만 볼 수 있듯, 우리가 일반적으로 생각하는 폴더나 파일 형태가 아닌, 계층형 파일 시스템이라는 특별한 방식으로 저장된다.

### 도커의 계층형 파일 시스템(Layered Filesystem)
크게 이미지 레이어, 컨테이너 레이어 두 가지로 구성된다.
1. 이미지 레이어
    - 도커 이미지는 여러 개의 읽기 전용(read-only) 레이어로 구성된다. 당연히 이미지는 구워져 나온 것이므로 읽기 전용이어야 한다. 
    - 흥미로운 점은 Dockerfile의 `FROM`, `RUN`, `COPY` 등 각 명령어마다 새로운 레이어가 쌓이는 구조다. **여러 컨테이너가 하나의 레이어를 공유**할 수 있게 되어 저장 공간을 꽤 절약할 수 있다. DLL(Dinamic Linked Library)와 비슷하다.
2. 컨테이너 레이어
    - 컨테이너를 실행하면 이미지 레이어들 위해 이제 쓰기 가능한 레이어가 하나 더 생기고, 이게 컨테이너 레이어다.
    - 컨테이너 내부에서 파일 생성, 수정, 삭제 등 모든 작업은 이 컨테이너 레이어에 기록된다.

`docker stop`은 이 컨테이너 레이어를 보존한 채로 컨테이너의 실행만 멈추기 때문에, `docker start`로 다시 컨테이너를 시작하면 이전 작업이 그대로 남아있게 되는 것이다.


### 볼륨을 굳이 잡아야 하나?
이렇게 호스트 머신에서 작업 내용을 볼 수 있다면 처음에 가졌던 의문처럼 volumn을 잡을 필요가 없지 않을까? 어차피 이곳에서 작업한 내용을 볼 수 있으니까, 굳이 디스크 용량을 더 쓸 필요가 있을까? 

위에서 봤던 도커 파일 시스템의 파일과 폴더들은 해시되어있어 알아볼 수 없다. 내용도 알 수 없다. 그래서 호스트에서 파일에 접근하고 싶다면 볼륨을 잡아야 한다.
```sh
admin@Gwanhos-MacBook-Air data % cat Docker.raw
...
q???\̀O? ???h???	?A???h4??h4??h
??~??
     Q
?A???h???h???h???d?D?
?A???h???h???h
?A???h???h???h?
?A???h???h???h?2?M?
...
```

### 왜 외계어를 썼을까
그렇다면 왜 도커 파일시스템은 컨테이너 리소스 관리를 이렇게 개판(?)으로 해놓은 것일까? 볼륨을 잡았듯이 그냥 컨테이너별로 내부 구조를 그대로 옮겨놓으면 편하지 않을까? 

그 이유는 짐작하겠지만 효율성 때문인데, [도커 파일시스템 원리](docker-filesystem-mechanism.md)에서 알아본다.

[Docker image layer caching](https://blog.macellan.net/docker-image-layer-caching-c471b1bd1f50)
> Docker uses a content-addressable storage system to store and manage the image layers, which means that each layer is identified by its content, rather than its name or location. This allows Docker to reuse layers across different images and different builds, even if the images have different names or are stored in different locations.


### 결론
컨테이너를 `stop`해도 작업 내역은 컨테이너 레이어에 안전하게 보존된다. 컨테이너를 완전히 `rm` 하기 전까지는 데이터가 사라질 걱정 없이 종료해도 괜찮음

데이터가 컨테이너의 생명주기를 넘어 영속적으로 보존되어야 하거나, 호스트에서 쉽게 접근하고 관리해야 한다면 볼륨(Volume)을 잡아주자