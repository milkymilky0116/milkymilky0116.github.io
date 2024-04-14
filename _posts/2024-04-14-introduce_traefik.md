---
title: Traefik으로 도커 컨테이너🐳 를 멋지게 배포해보기
date: 2024-04-14 22:04:43 +/-TTTT
categories: [traefik]
tags: [traefik]
mermaid: true # TAG names should always be lowercase
---

이전까지는 서버에 배포해 도메인을 등록할때 Nginx를 줄곧 쓰곤 했습니다. 하지만 배포해야할 컨테이너가 늘어나고 Nginx 만으로는 한계가 있다고 느끼던 있던 차에 Traefik 이라는 오픈소스 프로젝트를 알게 되었습니다. 오늘 글에서는 Traefik의 작동방식과 Traefik으로 멋지게 도커 컨테이너를 배포하는 방법에 대해서 알아보겠습니다.

![Pasted image 20240414222840](https://github.com/milkymilky0116/milkymilky0116.github.io/assets/84823612/27a26c1e-c886-442e-a7be-8a9f2c081236)

# Traefik과 여러가지 컨셉

Traefik을 쉽게 설명하자면 진보한 Nginx 입니다. 단일의 모놀리식 구성으로 되어있던 과거의 아키텍처와 달리, 요즘의 서비스 아키텍처는 여러개의 컨테이너, 혹은 클러스터로 이루어진 작은 단위의 Microservice 들이 모여 하나의 서비스를 이루는 구조로 되어 있습니다. 이러한 상황에서 유저가 각각의 Microservice에 접속하게끔 만들고 싶습니다. 그리고 우리에게는 Reverse Proxy가 필요한 상황입니다.

Nginx의 번거로운 점은 Configuration을 직접 작성해야 한다는 점입니다. 각각의 Microservice 대해서 직접 Route를 설정해줘야 하고, 이는 매우 귀찮은 작업이 될 수 있습니다. 만일 Microservice가 자주 추가되고, 업데이트 되고, 삭제되는 상황에 놓인다면 어떻게 해야 할까요? 그럴때마다 설정을 직접 해줘야 한다면 서비스의 유지보수에 있어서 큰 어려움을 겪을 수도 있습니다.

Traefik은 이러한 상황에서 도움을 줄 수 있습니다. 왜냐하면 Traefik은 우리를 위해 이러한 작업들을 자동으로 해주기 때문입니다. 라우팅부터, 로드밸런싱, 심지어 TLS 인증까지도요! 또한 Traefik에서 제공해주는 Metrics, Log 등으로 각각의 Microservice들에 대해서 모니터링 또한 가능합니다.

Traefik에는 크게 네가지 컨셉이 있습니다.

1. Entrypoints : EntryPoint는 네트워크의 진입점을 의미합니다. 외부에 노출되는 Port나, TCP 혹은 UDP 또한 설정 가능합니다.
2. Routers : Router는 Service에 대한 Request에 대해 연결하는 역할을 하고 있습니다.
3. Middleware : Middleware는 각각의 Router에 대해 Request나 Response를 변경하는 역할을 수행합니다.
4. Service : Service는 요청이 들어올때 실제 서비스에 어떻게 연결될지 설정하는 Configuration을 담당합니다.

![스크린샷 2024-04-14 오후 10 48 01](https://github.com/milkymilky0116/milkymilky0116.github.io/assets/84823612/f1587162-63b6-48af-9cc5-4e45c4ecc704)

# Traefik으로 TLS 인증까지 자동화 하기

Traefik은 TLS 인증까지 자동으로 배포할 수 있는 강력한 기능을 제공하고 있습니다. Traefik을 직접 배포 해보면서 알아보도록 하겠습니다. 아래는 Traefik의 docker-compose 파일입니다.

```yaml
version: "3.8"

services:

traefik:
image: traefik:v2.10.1
restart: unless-stopped
command:
	- --entrypoints.web.address=:80
	- --entrypoints.web.http.redirections.entryPoint.to=websecure
	- --entrypoints.web.http.redirections.entryPoint.scheme=https
	- --entrypoints.websecure.address=:443
	- --providers.docker=true
	- --providers.docker.exposedByDefault=false
	- --api
	- --certificatesresolvers.letsencryptresolver.acme.email=${EMAIL}
	- --certificatesresolvers.letsencryptresolver.acme.storage=/acme.json
	- --certificatesresolvers.letsencryptresolver.acme.tlschallenge=true
ports:
	- 80:80
	- 443:443
volumes:
	- /etc/localtime:/etc/localtime:ro
	- /var/run/docker.sock:/var/run/docker.sock:ro
	- ${TRAEFIK_DIR}/acme.json:/acme.json
labels:
	- traefik.enable=true
	- traefik.http.middlewares.admin.basicauth.users=${HTTP_BASIC_USER}:${HTTP_BASIC_PWD}
	- traefik.http.routers.traefik.entrypoints=websecure
	- traefik.http.routers.traefik.rule=Host(`traefik.${DOMAINNAME}`)
	- traefik.http.routers.traefik.service=api@internal
	- traefik.http.routers.traefik.middlewares=admin
	- traefik.http.routers.traefik.tls.certresolver=letsencryptresolver
```

꽤 설정이 많아서 혼란스러워 보일 수 있습니다만, 자세히 살펴보면 그렇게 어렵지 않습니다. command 부분은 traefik 서비스가 시작될 때 설정하는 부분입니다.

```yaml
	- --entrypoints.web.address=:80
	- --entrypoints.web.http.redirections.entryPoint.to=websecure
	- --entrypoints.web.http.redirections.entryPoint.scheme=https
	- --entrypoints.websecure.address=:443
```

`entrypoints` 부분은 앞서 설명드린 Traefik의 Entrypoint를 설정하는 부분입니다. 해당 부분에서는 `web`과 `websecure` 각각 2개의 entry를 설정했는데요, `web`은 http를, `websecure`는 https를 의미합니다. 각각 80포트와 443포트에 할당되도록 설정했습니다.

우리는 모든 트래픽이 https로 redirection 되기를 원합니다. 그래서 `redirections` 플래그를 설정했습니다. `web`으로 오는 요청에 대해서 `websecure`로 https으로 redirection 됩니다.

```
- --providers.docker=true
- --providers.docker.exposedByDefault=false
```

Traefik을 deploy할때, 기본적으로 expose 되지 않도록 설정했습니다. 왜냐하면 내부적으로만 쓰일 수 있는 서비스가 존재할 수 있기 때문입니다. 가령 웹서버는 외부에 노출시키고 싶을 수 있지만, DB와 같은 민감한 서비스는 내부 트래픽에서만 오가게 하고 싶을수도 있기 때문입니다. 그래서 기본적으로 traefik을 노출시키지 않는 플래그를 설정했습니다.

```yaml
- --api
```

`api` 플래그는 traefik의 대시보드를 활성화 합니다. router, middleware, service가 어떻게 작동되는지 쉽게 확인 할 수 있습니다.

![스크린샷 2024-04-14 오후 11 05 22](https://github.com/milkymilky0116/milkymilky0116.github.io/assets/84823612/e0613f32-1d26-4d97-87d7-045cb40f5d73)

```yaml
	- --certificatesresolvers.letsencryptresolver.acme.email=${EMAIL}
	- --certificatesresolvers.letsencryptresolver.acme.storage=/acme.json
	- --certificatesresolvers.letsencryptresolver.acme.tlschallenge=true
```

`certificatesresolvers` 플래그로 https 인증을 자동으로 최신화 할 수 있습니다. 인증된 정보는 `/acme.json`에 저장됩니다.

```
labels:
	- traefik.enable=true
	- traefik.http.middlewares.admin.basicauth.users=${HTTP_BASIC_USER}:${HTTP_BASIC_PWD}
	- traefik.http.routers.traefik.entrypoints=websecure
	- traefik.http.routers.traefik.rule=Host(`traefik.${DOMAINNAME}`)
	- traefik.http.routers.traefik.service=api@internal
	- traefik.http.routers.traefik.middlewares=admin
	- traefik.http.routers.traefik.tls.certresolver=letsencryptresolver
```

labels는 traefik으로 deploy될 서비스들을 정의 할 수 있습니다. 위의 예제는 traefik의 대시보드를 deploy하는 설정입니다.

```yaml
- traefik.http.routers.traefik.entrypoints=websecure
- traefik.http.routers.traefik.rule=Host(`traefik.${DOMAINNAME}`)
- traefik.http.routers.traefik.service=api@internal
```

deploy될 서비스의 entrypoint와 router를 설정할 수 있습니다. entrypoint는 https인 websecure로 설정하고, 접속할 수 있는 도메인을 설정합니다. 서비스는 `api@internal`로 연결됩니다.

```yaml
- traefik.http.routers.traefik.middlewares=admin
- traefik.http.middlewares.admin.basicauth.users=${HTTP_BASIC_USER}:${HTTP_BASIC_PWD}
```

traefik의 대시보드에는 기본적으로 Authentication 기능이 제공되지 않습니다. 그래서 traefik에서 제공해주는 middleware로 간단한 인증기능을 생성하고 있습니다.

```yaml
- traefik.http.routers.traefik.tls.certresolver=letsencryptresolver
```

그리고 TLS 인증을 위해 앞서 정의했던 `letsencryptresolver`를 사용합니다. 이렇게 정의함으로써 TLS 인증을 자동화 할 수 있습니다.

이렇게 설정하고 `docker-compose up -d` 를 실행하면 traefik 서버가 배포되는 것을 확인 할 수 있습니다. 이제 설정해준 도메인으로 접속되면 traefik의 대시보드가 보이는 것을 확인 할 수 있습니다.

![스크린샷 2024-04-14 오후 11 16 05](https://github.com/milkymilky0116/milkymilky0116.github.io/assets/84823612/a8294ade-d5ad-431a-94e8-6202b2271de2)

# 마무리

Traefik은 Microservice로 전환되는 요즘의 서비스 아키텍처의 요구에 맞는 Reverse-proxy 기능을 제공하고 있습니다. 이전에 Nginx의 설정과 일일이 씨름하면서 서비스를 배포했던 것을 생각하면 Traefik이 제공해주는 기능은 상당히 매력적으로 보였습니다. Traefik은 이외에도 많은 기능을 제공해주고 있기에 더 깊이 파보고 싶다는 생각이 들었습니다.
