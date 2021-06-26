# Docker

## Container vs Virtual Machines

가상 머산과 비교했을 때, 도커 플랫폼은 자원의 추상화를 하드웨어 레벨에서 운영체제 레벨로 옮긴다. 이 플랫폼은 많은 이점들을 가진다. (자가격리된 마이크로 서비스, 인프라의 분리, 애플리케이션 휴대성)

가상 머신을 하드웨어 서버 전체를 추상화 하는 반면에, 컨테이너는 운영체제 커널을 추상화한다. 이것은 가상화에 대해 완전히 다른 접근이며, 그 결과 더 가벼운 인스턴스와 더 높은 속도를 가질 수 있게 된다.

<figure>
<p align="center"><img src="https://wiki.aquasec.com/download/attachments/2854889/Container_VM_Implementation.png?version=1&modificationDate=1520172703952&api=v2"></p>
<figcaption align = "center"><i>Container vs Virtual Machine</i></figcaption>
</figure>

## Advantages

도커의 주요 이점은

1. 자원의 효율성: 프로세스 레벨의 격리와 컨테이너 호스트의 커널 사용은 하드웨어 서버 전체를 가상화 했을 때보다 자원의 효율이 더 좋다.
2. 휴대성: 모든 애플리케이션에 대한 의존성은 컨테이너에 번들되어 개발, 테스트, 프로덕션 환경으로 쉽게 옮겨갈 수 있다.
3. 지속적인 배포와 테스트: 도커는 일관된 환경을 가지게 해주며 패치시 유연하기 때문에 기존 waterfall 방식에서 현대 데브옵스 방식으로 옮기려는 팀에게 좋은 선택이다.

## Docker Engine

먼저, 도커 엔진과 컴포넌트들을 보면 어떻게 시스템이 동작하는지 알 수 있다. 도커 엔진을 사용하면 다음 컴포넌트들을 사용하여 애플리케이션을 개발, 통합, 실행할 수 있다.

1. Docker Daemon: 도커 이미지, 컨테이너, 네트워크 그리고 스토리지들을 관리하는 백그라운드 프로세스이다. 도커 데몬은 지속적으로 도커 API 요청들을 듣고 프로세스들을 관리한다.
2. Docker Enine REST API: 도커 엔진에서 상호작용하는 애플리케이션을 위한 API이다. HTTP Client에 의해 액세스된다.
3. Docker CLI: 도커 엔진과 상호작용하기 위한 커맨드라인 인터페이스 클라이언트이다. 이 CLI를 통해 컨테이너 인스턴스를 쉽게 관리할 수 있다. 많은 개발자들이 도커를 사랑하는 이유중 하나이다.

<figure>
<p align="center"><img src="https://wiki.aquasec.com/download/attachments/2854889/Docker_Engine.png?version=1&modificationDate=1520172702424&api=v2"></p>
<figcaption align = "center"><i>Docker Engine</i></figcaption>
</figure>

## Implementation

도커는 광법위한 플랫폼에서 구현될 수 있다.

1. Desktop: Mac OS, Window 10
2. Server: Window Server와 다양한 Linux 배포판들
3. Cloud: AWS, GCP, Azure, IBM Cloud 등

## Architecture

도커는 서버-클라이언트 모델을 사용하고 도커 클라이언트, 도커 호스트, 네트워크 그리고 스토리지 컴포넌트와 도커 레지스트리(도커 허브)로 이루어져있다.

<figure>
<p align="center">
<img src="https://wiki.aquasec.com/download/attachments/2854889/Docker_Architecture.png?version=1&modificationDate=1520172700553&api=v2">
</p>
<figcaption align = "center"><b>Docker Architecture</b></figcaption>
</figure>

### Docker Client

도커 클라이언트는 사용자가 도커와 상호작용할 수 있도록 해준다. 도커 클라이언트는 원격 호스트의 데몬과 연결할 수 있으며, 하나 이상의 데몬과 소통할 수 있다. 커맨드 라인 인터페이스를 제공함으로써 애플리케이션을 빌드, 실행, 중지할 수 있는 커맨드를 데몬에 전달할 수 있다. 도커 클라이언트의 주요 목적은 직접 레지스트리에서 이미지를 받아 호스트에서 실행할 수 있는 수단을 제공하는 것이다.

### Docker Host

도커 호스트는 애플리케이션을 실행하도록 환경을 제공한다. 도커 호스트는 도커 데몬, 이미지, 네트워크, 스토리지, 컨테이너로 구성되어 있다. 이전에 언급했듯이, 데몬은 모든 컨테이너와 관련된 액션과 REST API를 통한 CLI 커맨드를 받는다. 또한 서비스를 관리하기 위해 다른 데몬들과 커뮤니케이션이 가능하다. 데몬은 클라이언트가 요청한 컨테이너 이미지를 받고 빌드한다. 요청된 이미지를 받으면, 빌드 파일로 컨테이너를 빌드한다. 이 빌드파일은 또한 컨테이너를 실행하기 전에 다른 컴포넌트를 미리 실행하기 위해서 데몬을 위한 명령어나 한번 컨테이너가 빌드 되면 수행할 커맨드 라인도 포함할 수 있다.

### Docker Objects

**Images**
이미지는 컨테이너를 빌드하기 위한 read-only 바이너리 템플릿이다. 이미지는 또한 컨테이너에 대한 메타데이터를 포함하고 있다. 애플리케이션을 저장하고 옮기는데 사용된다. 이미지를 자체적으로 컨테이너를 빌드하거나 사용자가 이미지를 사용하여 현재의 구성을 확장시킬 수 있다. 컨테이너 이미지는 비공개 컨테이너 레지스트리를 사용하여 기업내에 있는 팀끼리 공유하거나 퍼블릭 레지스트리인 도커 허브를 사용하여 세계에 공유할 수 있다. 이미지는 전에는 가능하지 못했던 방식으로 개발자들 사이에 협업을 할 수 있게 해주기 떄문에 가장 중요한 부분이다.

**Containers**
컨테이너는 애플리케이션이 실행될 수 있는 환경에 캡슐화된다. 컨테이너는 오직 이미지에 정의된 리소스만 액세스할 수 있다. 현재 컨테이너 상태에 기반이 되는 새로운 이미지를 생성할 수 있다. 컨테이너는 가상머신보다 훨씬 가볍기 때문에 몇 초만에 띄울 수 있다.

**Networking**
**Storages**
