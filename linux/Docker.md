# Docker

---

[https://www.youtube.com/watch?v=wW9CAH9nSLs](https://www.youtube.com/watch?v=wW9CAH9nSLs)

## 도커란?

- 2013년에 PyCon US에서 솔로몬 하익스가 발표한 오픈소스 프로젝트
- 개발자들을 위해 애플리케이션을 빠르고 가볍고 쉽게 개발, 실행하기 위한 플랫폼
- 거대한 서비스구조 안에 있는 하나의 애플리케이션을 떼어다가 쉽게 다른 곳에 붙일 수 있게 해준다. 쉽게 운송가능한 컨테이너. (컨테이너 가상화 기법을 통해 의존성 제거, 이식성이 뛰어남)

⇒ 비슷한 느낌의 솔루션으로 파이썬의 virtualenv

→ 가상화와 컨테이너?

---

## 가상화

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/50828aeb-3015-4de9-a2c9-11cb88aea169/virtualization.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/50828aeb-3015-4de9-a2c9-11cb88aea169/virtualization.png)

하이퍼바이저는? 호스트 시스템에서 다수의 게스트 OS를 구동할 수 있게 하는 소프트웨어. 중간 관리자 역할을 하는 것!(비슷한 역할을 하는 것이 컨테이너 엔진 <도커 엔진>)

1. 베어메탈 가상머신

   ex) 멀티 부팅

2. 호스트형 가상머신

ex) VM ware, Virtual box

3. 컨테이너

ex) Docker !

→ 컨테이너는 가상화 기술의 한 종류이며 도커에 종속되지 않는 용어이다!

### 컨테이너

- 운영 체제 수준의 컨테이너 기술로 리눅스의 커널을 공유하면서 프로세스를 격리된 환경에서 실행하는 기술
- 커널을 공유하는 방식이기 때문에 실행속도가 빠르고, 성능상의 손실이 없음 ⇒ 베어메탈과 비교했을 때 98%의 성능
- 컨테이너로 실행된 프로세스는 커널을 공유하지만 커널 기능(namespace, cgroup, chroot 등)을 활용해 격리

→ 따라서 호스트 OS 관점에서는 프로세스로 보이나 내부를 들여다 보면 하나의 가상환경임

리눅스 커널? 커널=운영체제가 아니라 커널은 운영체제의 핵심이 되는 프로그램이다. 시스템의 모든 것을 제어한다. User ↔ Shell ↔ Kernel

### 참고자료

- 서버 가상화 발전과정

  [가비아 라이브러리](https://library.gabia.com/contents/infrahosting/7426/)

- 리눅스 커널

  [Linux 커널이란 무엇일까요?](https://www.redhat.com/ko/topics/linux/what-is-the-linux-kernel)

- 컨테이너

  [컨테이너란? 리눅스의 프로세스 격리 기능](https://www.44bits.io/ko/keyword/linux-container#%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EB%9E%80)

---

## 특징(기존 컨테이너 가상화 기술과의 비교)

- 가볍고 빠른 실행 속도과 쉬운 배포
- 개발자들에게 편리한 클라이이언트 도구(Dockerfile, Docker cli, docker-compose)들을 많이제공
- 컨테이너 이미지 생성 및 공유 가능 (도커 허브를 통해)
- docker-compose라는 여러 Docker 컨테이너를 통합적으로 관리하는 cli 프로그램이 있어 컨테이너 구성에 용이

## 기본 구조

> 도커는 서버-클라이언트 모델을 사용하고 도커 클라이언트, 도커 호스트, 네트워크 그리고 스토리지와 도커 레지스트리(도커 허브)로 이루어져있다.

![https://wiki.aquasec.com/download/attachments/2854889/Docker_Architecture.png?version=1&modificationDate=1520172700553&api=v2](https://wiki.aquasec.com/download/attachments/2854889/Docker_Architecture.png?version=1&modificationDate=1520172700553&api=v2)

도커 구조

### Docker Daemon

도커 데몬(dockerd)는 도커 REST API 요청을 듣고 도커 오브젝트들(이미지, 컨테이너, 네트워크 그리고 볼륨)을 관리한다. 또한 도커 데몬은 도커 서비스를 관리하는 다른 도커 데몬과 커뮤니케이션할 수 있다.

→ 데몬이 두 개 이상이 될 수 있나? [https://docs.docker.com/engine/reference/commandline/dockerd/#run-multiple-daemons](https://docs.docker.com/engine/reference/commandline/dockerd/#run-multiple-daemons)

### Docker Client

도커 클라이언트는 사용자가 도커와 상호작용할 수 있도록 해준다. 도커 클라이언트는 호스트의 데몬과 연결할 수 있으며, 하나 이상의 데몬과 소통할 수 있다. CLI를 제공함으로써 애플리케이션을 빌드, 실행, 중지할 수 있는 커맨드를 데몬에 전달할 수 있다. 도커 클라이언트의 주요 목적은 직접 레지스트리에서 이미지를 받아 호스트에서 실행할 수 있게 하는 것이다.

### Docker Host

도커 호스트는 애플리케이션을 실행하도록 환경을 제공한다. 도커 호스트는 도커 데몬, 이미지, 네트워크, 스토리지, 컨테이너로 구성되어 있다. 이전에 언급했듯이, 데몬은 모든 컨테이너와 관련된 액션과 REST API를 통한 CLI 커맨드를 받는다. 또한 서비스를 관리하기 위해 다른 데몬들과 커뮤니케이션이 가능하다. 데몬은 클라이언트가 요청한 컨테이너 이미지를 받고 빌드한다. 요청된 이미지를 받으면, 빌드 파일로 컨테이너를 빌드한다.

### Docker Objects

- **Images**

이미지는 컨테이너를 빌드하기 위한 read-only 바이너리 파일이다. 이미지는 추가적인 구성과 함께 또 다른 이미지의 베이스가 될 수 있다. 컨테이너 이미지는 비공개 컨테이너 레지스트리를 사용하여 기업내에 있는 팀끼리 공유하거나 퍼블릭 레지스트리인 도커 허브를 사용하여 세계에 공유할 수 있다. 이미지는 전에는 가능하지 못했던 방식으로 개발자들 사이에 협업을 할 수 있게 해주기 때문에 가장 중요한 부분이다.

- **Container**

컨테이너는 실행가능한 이미지의 인스턴스이다. 사용자는 컨테이너를 Docker API를 통해 생성, 삭제, 수정, 이동할 수 있다. 하나 이상의 네트워크에 컨테이너를 연결할 수 있다. 또한 스토리지를 분리 할 수 있으며 심지어 현재 상태를 기반으로 이미지를 생성할 수도 있다.
