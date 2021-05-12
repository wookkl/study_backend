# Channels란?

Channels란 웹 소켓, 챗 프로토콜 등 을 다루기 위한 HTTP 이상의 기능을 가지고 있는 프로젝트이다. ASGI에 내장되어 있는 프로젝트이다. Channels는 장고 3.0 버전이후로 네이티브 ASGI에 구축되었다. 장고는 기본적으로 전통 HTTP를 다루지만, Channels는 장고가 비동기 또는 동기를 자유롭게 선택하여 다룰 수 있도록 제공한다.

# Channels의 특징

- HTTP 뿐만 아니라 웹소켓같은 긴 연결을 요구하는 프로토콜들을 핸들링 가능
- 이벤트-드리븐 아키텍처
- 채팅 메세지, 알림등을 다룰 수 있도록하는 `consumer`를 제공 (장고의 view랑 약간 비슷)

# Scopes and Events

Channels 와 ASGI는 들어오는 연결을 `scope`와 `event`로 나눈다.

## Scope

- 하나의 들어오는 커넥션에 대한 정보들의 데이터 셋
- 예를 들어 어디서 요청이 들어왔는지의 path 또는 유저의 메세지 등의 정보
- HTTP에 대해서는 단 하나의 요청이고 웹소켓에 대해서는 소켓의 라이프타임동안 지속되는 등 프로토콜의 ASGI 스펙에 따라 다름

## Event

- 위의 scope의 라이프타임 동안에 이벤트(유저와 서버의 상호작용)가 발생

# Consumer

- Consumer란 Channels 코드의 기본적인 단위
- 위의 이벤트들을 소비하는 `consumer`라고 할 수 있음
- 요청이나 새로운 소켓이 들어오면 Channels는 라우팅 테이블을 보고 그에 맞는 `consumer`로 보냄

# 공식 문서

[Introduction - Channels 3.0.3 documentation](https://channels.readthedocs.io/en/stable/introduction.html)
