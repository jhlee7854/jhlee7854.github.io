---
draft: false
date: '2025-05-14T13:43:48+09:00'
title: 'Telepresence를 써보자'
summary: "Telepresence를 이용하여 Kubernetes에 배포된 애플리케이션을 로컬 PC에서 디버깅하는 것에 대해 간단히 설명하였다."
tags: ["post", "telepresence"]
searchHidden: false
disableShare: false
canonicalURL: 'https://jhlee7854.github.io/posts/lets-try-telepresence'
ShowCanonicalLink: true
CanonicalLinkText: "Originally published at"
cover:
    image: "https://raw.githubusercontent.com/telepresenceio/telepresence.io/master/src/assets/images/telepresence-edgy.svg" # image path/url
    alt: "telepresence logo" # alt text
    caption: "https://github.com/telepresenceio/telepresence" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false
---
## Telepresence란?

공식 문서에서는 다음과 같이 설명하고 있다.

>Telepresence는 개발자가 원격 쿠버네티스 클러스터에 있는 마이크로서비스를 로컬에서 코딩하고 테스트할 수 있게 해주는 오픈 소스 도구이다. Telepresence는 다른 서비스 종속성에 대한 염려를 덜어주면서 보다 효율적인 개발 흐름을 가져갈 수 있도록 돕는다.

간단하게 말하면 로컬 PC를 원격 쿠버네티스 클러스터와 네트워크로 연결하고 클러스터 내부의 특정 애플리케이션과 관련된 트래픽을 가로채서 로컬 PC의 애플리케이션으로 전달한다. 이러한 방식은 원격 쿠버네티스 클러스터 내부에서 실행되고 있는 애플리케이션을 로컬 PC에서 실행하고 있는 애플리케이션으로 대체하는 효과가 있다. 그러므로 로컬 PC에서 개발중인 애플리케이션을 배포하지 않고도 원격 쿠버네티스 클러스터에 있는 다른 애플리케이션과의 상호작용 및 변경 사항에 대한 즉각적인 확인 또는 디버깅이 가능하다.

**NOTE**: 본문서의 내용은 telepresence `v2.22.4`를 기준으로 작성되었다. 버전에 따라 명령어 및 동작이 상이할 수 있다.

### Inner dev loop

Inner dev loop는 한명의 개발자에 대한 업무 흐름을 나타낸다. 다음 그림은 전통적인 inner dev loop를 보여 준다. 그림은 한명의 개발자가 코드를 수정하고 결과를 확인하는 한번의 흐름을 보여주고 있고, 이러한 흐름을 업무 시간 동안 반복한다는 것이다. 공식 문서에서는 "developer tax"라고 말하는 그림의 빨간색 부분은 필연적으로 소요되는 시간을 나타낸다.

{{<figure
  src="https://telepresence.io/assets/images/trad-inner-dev-loop-9ae1f79473a581f556ad8884bd19e73e.png#devloop"
  link="https://telepresence.io/docs/concepts/devloop#what-is-the-inner-dev-loop"
  alt="Traditional Inner Dev Loop"
  caption="Traditional Inner Dev Loop (출처: telepresence.io)"
  align="center"
  target="_blank"
>}}

다음 그림은 컨테이너를 이용한 개발에 대한 inner dev loop를 보여 준다. 위 그림과 비교해 보면 컨테이너를 빌드하고 업로드 및 배포하는 시간이 developer tax에 추가 되었고 그 비율이 40%에 가깝다. 즉 그 만큼 필연적인 시간이 더 소요되어 개발 생산성이 저하 되었다고 볼 수 있다.

{{<figure
  src="https://telepresence.io/assets/images/container-inner-dev-loop-7a5db1e9e9b56edd0b3dacbe683224aa.png#devloop"
  link="https://telepresence.io/docs/concepts/devloop#in-search-of-lost-time-how-does-containerization-change-the-inner-dev-loop"
  alt="Container Inner Dev Loop"
  caption="Container Inner Dev Loop (출처: telepresence.io)"
  align="center"
  target="_blank"
>}}

위 그림의 추가된 developer tax는 애플리케이션이 완성된 후 팀 레벨의 최종 배포 단계에서 필요한 비용이라고 볼 수 있다. Telepresence는 container inner dev loop에서 한명의 개발자가 코드를 수정할 때 마다 소요되는 컨테이너를 빌드하고 업로드 및 배포하는 비용을 제거하여 개발자의 생산성을 향상시키기 위한 도구이다.

### Telepresence 구성요소

Telepresence는 다음 그림과 같은 아키텍처와 구성요소를 가지고 있다.

{{<figure
  src="https://telepresence.io/assets/images/TP_Architecture-4a754cc82947915d1f44ee582315e885.svg"
  link="https://telepresence.io/docs/reference/architecture"
  alt="Telepresence Architecture"
  caption="Telepresence Architecture (출처: telepresence.io)"
  align="center"
  target="_blank"
>}}

- Telepresence CLI: 개발자의 로컬 PC에 설치되어 `Telepresence Daemon`을 시작하고 사용자와 상호작용할 수 있는 인터페이스를 제공한다.
- Telepresence Daemon: 개발자는 자신의 로컬 PC에서 Telepresence Daemon을 실행하여 클러스터와 로컬 PC의 트래픽 및 네트워킹을 관리하고, 클러스터에 있는 `Traffic Manager`와 상호작용하여 replacement, ingest, intercept의 생성과 삭제를 담당한다.
- Traffic Manager: 클러스터의 `Traffic Agent`와 개발자 로컬 PC의 Telepresence Daemon 사이의 통신을 관리한다. 실행중인 pod의 Traffic Agent 사이드카 주입, 관련된 모든 인바운드 및 아웃바운드 트래픽에 대한 프록시, 활성화된 작업에 대한 추적을 담당한다.
- Traffic Agent: 대상 워크로드의 pod에 주입되는 사이드카 컨테이너다. `replace`, `ingest`, `intercept`가 처음 시작되면 워크로드의 pod에 Traffic Agent 컨테이너가 주입된다. replace 또는 intercept가 활성화 되어 있다면 Traffic Agent는 수신하는 요청을 개발자의 로컬 PC로 라우팅 한다.

**NOTE**: 요청 트래픽을 인터셉트하는 개념을 보여주는 그림은 {{<rawhtml>}}<a href="https://telepresence.io/docs/concepts/intercepts?intercept=regular" target="_blank">여기</a>{{</rawhtml>}}서 확인할 수 있다.

### Telepresence가 제안하는 로컬 개발 방식

Telepresence는 다음과 같이 애플리케이션을 로컬에서 개발하는 세가지 방식을 제안한다. 각 방식의 유즈 케이스를 확인하여 각 애플리케이션 마다 적합한 방식을 취하면 된다.

#### Replace

- 작동 방식:
  - 트래픽 에이전트를 이용하여 쿠버네티스 클러스터 안에 있는 컨테이너를 대체한다.
  - 대체된 컨테이너로 향하는 트래픽을 로컬 PC로 다시 라우팅 한다.
  - 대체된 컨테이너의 원격 환경을 로컬 PC가 사용할 수 있게 한다.
  - 대체된 컨테이너에 마운트된 볼륨에 읽기/쓰기 접근을 제공한다.
- 영향:
  - 트래픽 에이전트가 대상 워크로드의 파드에 주입된다.
  - 대체된 컨테이너는 대상 워크로드의 파드에서 제거 된다.
  - 대체된 컨테이너는 replace 명령이 종료되면 다시 복원된다.
- 유즈 케이스:
  - 메시지 큐 컨슈머 처럼 (중복 처리를 막기 위해) 로컬 PC의 애플리케이션이 원격 컨테이너를 중지하여 완전히 대체해야 하는 경우
  - 원격 컨테이너로 트래픽이 유입되지 않도록 설정해야 하는 경우

#### Intercept

- 작동 방식:
  - 특정 서비스 포트(들)로 향하는 요청을 가로채서 로컬 PC로 다시 라우팅 한다.
  - 인터셉트 대상 컨테이너의 원격 환경을 로컬 PC가 사용할 수 있게 한다.
  - 인터셉트 대상 컨테이너에 마운트된 볼륨에 읽기/쓰기 접근을 제공한다.
- 영향:
  - 트래픽 에이전트가 대상 워크로드의 파드에 주입된다.
  - 가로채진 트래픽은 로컬 PC로 다시 라우팅 되며, 원격 서비스에는 더이상 도달할 수 없다.
  - 모든 컨테이너들은 계속 실행 중 이다.
- 유즈 케이스:
  - 클러스의 파드나 컨테이너의 동작 보다 서비스 API의 동작에 중점을 둘 경우
  - 로컬 서비스의 특정 유입 트래픽 수신만 필요한 경우(다른 트래픽은 건드리지 않는다.)
  - 리모트 서비스가 다른 요청이나 백그라운드 작업을 계속 수행해야 하는 경우

#### Ingest

- 작동 방식:
  - 수집 대상 컨테이너의 원격 환경을 로컬 PC가 사용할 수 있게 한다.
  - 대체된 컨테이너에 마운트된 볼륨에 읽기 전용 접근을 제공한다.
- 영향:
  - 트래픽 에이전트가 대상 워크로드의 파드에 주입된다.
  - 트래픽을 다시 라우팅하는 일은 없으며, 모든 컨테이너들도 계속 실행 중이다.
- 유즈 케이스:
  - 로컬 개발 환경이 원격 서비스에 주는 영향을 최소하능로 유지하고 싶을 경우
  - 클러스터가 로컬 PC로 트래픽을 라우팅 할 필요가 없고 컨테이너의 볼륨에 읽기 전용 접근만 필요한 경우

## Commands

**NOTE**: `macOS`를 기준으로 최소한의 필요한 명령어만 알아본다. 그 외의 설치 및 자세한 명령어 사용법은 공식 문서를 참고하자.

### Client 설치

개발자의 로컬 PC에 클라이언트를 설치한다. 클라이언트는 원격 쿠버네티스 클러스터와 상호작용을 위한 CLI 명령 및 데몬 프로세스를 제공한다.

```sh
brew install telepresenceio/telepresence/telepresence-oss
```

### Client 설정 확인

클라이언트 설정을 확인한다. 연결된 클러스터 네임스페이스, dns, routing 정보 등을 확인할 수 있다.

```sh
telepresence config view
```

### Traffic Manager 설치

원격 쿠버네티스 클러스터에 `ambassador` 네임스페이스를 생성하고 트래픽 매니저를 설치한다. 트래픽 매니저는 대상 워크로드에 트래픽 에이전트 사이드카 주입 및 트래픽 라우팅 등 클러스터 측면의 작업을 담당한다.

**NOTE**: 트래픽 매니저의 설치는 telepresence 클라이언트와 `helm`을 사용하므로 사전에 telepresence 클라이언트와 helm이 설치되어 있거나 직접 해당 명령어를 사용할 수 있어야 한다. 또한 트래픽 매니저를 설치하려는 클러스터에 대해 관리자 권한이 필요하다.

```sh
telepresence helm install
```

### Telepresence 상태 확인

클라이언트 단의 데몬 실행 여부 및 클러스터의 트래픽 매니저와의 연결 상태를 확인한다.

```sh
telepresence status
```

### Cluster 연결

로컬 PC에서 telepresence daemon을 실행하여 (kubeconfig에 설정되어 있는) 대상 클러스터의 네임스페이스와 연결한다. 클러스터의 네임스페이스와 연결이 되면 해당 네임스페이스의 워크로드를 대상으로 replace, intercept, ingest가 가능하다.

**NOTE** 다음 명령어 실행 시 로컬 PC의 root 권한이 필요하다고 하면서 PC 관리자 비밀번호을 요구할 수 있다.

```sh
telepresence connect
```

만약 특정 네임스페이스로 연결하거나, traffic manager를 별도의 네임스페이스에 설치했다면 다음과 같이 추가적인 플래그를 지정할 수 있다.

```sh
# telepresence connect --namespace ${TARGET_NAMESPACE} --manager-namespace ${TRAFFIC_MANAGER_NAMESPACE}
telepresence connect --namespace default --manager-namespace ambassador
```

### Workload 조회

workload 목록을 조회한다.

```sh
telepresence list
```

traffic agent가 설치된 workload만 조회한다.

```sh
telepresence list -a
```

특정 네임스페이스의 workload만 조회한다.

```sh
# telepresence list -n ${TARGET_NAMESPACE}
telepresence list -n default
```

replace 대상 workload만 조회한다.

```sh
telepresence list -r
```

ingest 대상 workload만 조회한다.

```sh
telepresence list -g
```

intercept 대상 workload만 조회한다.

```sh
telepresence list -i
```

### Replace

replcae를 실행하면 대상 워크로드의 지정된 컨테이너를 중지하고 해당 컨테이너의 선언된 모든 포트를 localhost의 대응되는 포트로 매핑하여 대상 워크로드를 대체한다. 만약 포트 매핑을 변경하고 싶다면 `--port` 플래그를 사용한다. `--port ${LOCAL_PORT}:${REMOTE_CONTAINER_PORT}` 형식으로 사용한다.

**NOTE**: 대상 컨테이너의 볼륨을 마운트하기 위해서는 `macFUSE`와 `SSHFS`가 필요하며, 해당 도구를 설치 후 보안관련 설정의 변경이 필요하다. 이와 관련된 내용은 [macFUSE Getting Started](https://github.com/macfuse/macfuse/wiki/Getting-Started)를 참고 하자.

```sh
# telepresence replace ${WORKLOAD_NAME} --container ${CONTAINER_NAME} --env-file ${ENVIRONMENT_FILE_PATH} --mount ${MOUNT_POINT_PATH}
telepresence replace example-app --container echo-server --env-file /tmp/example-app.env --mount /tmp/example-app-mounts
```

**NOTE**: 위 명령을 수행하면 `--evn-file` 플래그에서 지정한 경로의 파일에 대상 컨테이너의 환경 변수 정보를 가져오고, `--mount` 플래그에서 지정한 경로에 대상 컨테이너의 볼륨을 마운트 한다. 그러므로 로컬 PC에서 실행하는 애플리케이션은 위 환경 변수 정보 및 마운트된 볼륨을 이용하면 클러스터와 동일한 환경에서 개발 및 디버깅이 가능하다.

**NOTE**: `--mount` 플래그를 생략하더라도 별도의 지정된 경로에 볼륨을 마운트한다.

### Intercept

intercept 명령은 특정 서비스 트래픽을 가로채서 로컬 PC로 라우팅 한다. replace는 트래픽을 가로채는 것과 더불어 대상 워크로드에서 지정된 컨테이너를 제거하는 반면 intercept는 트래픽만 가로채서 로컬 PC로 전달하므로 대상 워크로드가 트래픽과 관련없는 작업을 지속해서 수행할 수 있다.

```sh
# telepresence intercept ${WORKLOAD_NAME} --port ${LOCAL_PORT}:${REMOTE_CONTAINER_PORT} --env-file ${ENVIRONMENT_FILE_PATH} --mount ${MOUNT_POINT_PATH}
telepresence intercept example-app --port 8080:http --env-file ~/example-app-intercept.env --mount /tmp/example-app-mounts
```

**NOTE**: 대상 워크로드의 포트가 하나 뿐이라면 생략 가능하며 대상 워크로드와 동일한 포트로 로컬 PC에 매핑한다. 하지만 대상 워크로드가 여러개의 포트를 노출하고 있다면 가로채려고 하는 트래픽과 관련된 포트를 명시해야 한다. `--port` 플래그 대신 `--service` 플래그를 사용하여 특정 서비스의 트래픽을 가로챌 수도 있다.

하나 이상의 포트를 intercept 해야 할 경우 다음과 같이 intercept name을 다르게 주고 `--workload` 플래그에 intercept할 워크로드를 지정한다.

```sh
# telepresence intercept ${INTERCEPT_NAME} --workload ${WORKLOAD_NAME} --port ${LOCAL_PORT}:${REMOTE_CONTAINER_PORT}
telepresence intercept multi-echo-http --workload multi-echo --port 8080:http
telepresence intercept multi-echo-grpc --workload multi-echo --port 50051:grpc --mechanism tcp
```

### Ingest

ingest 명령은 대상 워크로드의 실행에는 영향을 미치지 않고 환경 변수 및 볼륨 마운트 정보만 로컬 PC로 가져와 로컬 PC에서 코드를 실행하고 디버깅할 수 있다. 대상 워크로드의 환경 변수 및 볼륨 마운트를 로컬 PC에서 사용할 수 있지만 트래픽을 가로채거나 대상 워크로드를 대체(제거)하지 않는다.

```sh
# telepresence ingest ${WORKLOAD_NAME} --container ${CONTAINER_NAME} --env-file ${ENVIRONMENT_FILE_PATH} --mount ${MOUNT_POINT_PATH}
telepresence ingest example-app --container echo-server --env-file /tmp/example-app.env --mount /tmp/example-app-mounts
```

**NOTE**: `--env-file` 및 `--mount` 플래그는 replace와 동일한 기능을 한다.

### Traffic Agent 제거

telepresence의 replace, intercept, ingest가 활성화 되었던 워크로드는 traffic agent가 사이드카로 주입된 상태로 유지된다. 이미 사이드카로 주입되었던 traffic agent를 제거해야 할 필요가 있다면 다음 명령을 사용한다.

```sh
# telepresence uninstall ${WORKLOAD_NAME}
telepresence uninstall example-app
```

다음 플래그를 사용하여 모든 워크로드의 모든 traffic agent를 한번에 제거할 수 있다.

```sh
telepresence uninstall --all-agents
```

### 명령 중지 및 연결 종료

대상 워크로드에 대해 활성화된 명령(replcae, ingest, intercept)을 종료한다.

```sh
# telepresence leave ${WORKLOAD_NAME} --container ${CONTAINER_NAME}
telepresence leave example-app --container echo-server
```

클러스터의 traffic manager와 연결을 종료한다.

```sh
telepresence quit
```

클러스터의 traffic manager와 연결을 종료할 때 로컬 PC의 telepresence daemon도 같이 종료한다.

```sh
telepresence quit -s
```

**NOTE**: telepresence를 통해 클러스터의 워크로드와 로컬 PC가 연결되어 replcae 또는 intercept가 활성화된 경우 leave 또는 quit을 하지 않고 로컬 PC의 애플리케이션을 종료할 경우 쿠버네티스는 해당 애플리케이션이 클러스터 내에서 종료된 것으로 인지한다. 이로 인해 (관련 트래픽이 모두 로컬 PC로 라우팅 되는 상황이므로 쿠버네티스는 해당 애플리케이션에 대해 readiness 및 liveness probe를 할수 없어) 클러스터에 배포된 애플리케이션이 지속적으로 재시작 되는 것과 같은 문제 발생할 수 있으므로 워크로드와 로컬 PC 애플리케이션 간의 활성화 종료(leave) 또는 연결 종료(quit)를 먼저 수행하고 애플리케이션을 종료하는 것이 바람직하다.

## References

- [telepresence.io](https://telepresence.io)
- [Telepresence Quickstart](https://telepresence.io/docs/quick-start)
- [macFUSE Getting Started](https://github.com/macfuse/macfuse/wiki/Getting-Started)
