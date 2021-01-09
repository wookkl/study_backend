# Apline Python

_python:<version>-alpine_

이 이미지는 **알파인 리눅스 프로젝트 기반**으로 만들어진 파이썬 이미지이다. 이미지 크기가 5MB이하로 용량이 작은 것이 특징이다. 그래서 최대한 이미지의 용량을 작게해야 할 때 권장한다. glibc(GNU library C) 대신에 musl libc(임베디드 리눅스를 위한 신뢰성있는 C 라이브러리)를 사용하기 때문에 특정 패키지가 libc에 의존한다면 문제가 발생할 수 있다.

**정리하자면, 알파인 리눅스 프로젝트 기반으로 만들어진 5MB보다 작은 경량형 파이썬 도커 이미지 라는 것이다.**

최근에는 거의 모든 도커 이미지가 Alpine linux 기반으로 만들어진다고 한다.

## 그렇다면 alpine linux는 정확히 무엇일까? 

[alpinelinux.org/about/](https://alpinelinux.org/about/)

알파인 리눅스란 가볍고 간단한 보안성 목적으로 개발한 리눅스 배포판이다. 용량을 줄이기  위해서 C 런타임을 glibc 대신에 musl libc을 사용한다. 다양한 쉘 명령어는 GNU util 대신 busybox 를 사용한다. 용량이 80M로 가볍기 떄문에 임베디드 또는 네트워크 서버등에 유용하고 특히 도커에서 5M크기의 리눅스 이미지로 널리 배포되었다.

특징 중에 하나는 패키지 매니저는 apk를 사용한다.

apk 패키지 매니저 사용 예

apk add --update --no-cache postgresql-client
