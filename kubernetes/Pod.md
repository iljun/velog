Pod이란 kubernetes에서 생성, 관리, 배포 할 수 있는 가장 단위이다.
하나 이상의 컨테이너 그룹으로 하나의 pod 내부에 존재하는 컨테이너는 storage, network를 공유한다.

하나 이상의 컨테이너 그룹이란건 언제나 여러개의 container로 구성되어야하는 의미가 아니다. 일반적으로 하나의 pod은 하나의 컨테이너로 구성된다.

하나 이상의 컨테이너 그룹으로 구성된 pod은 내부에 존재하는 container는 언제나 동일한 Node에서 실행되도록 보장된다.

## 왜 Pod일까?
그냥 container를 사용하면 더 편하지 않을까? 왜 kubernetes에서는 pod이란 개념을 도입했을까?
- IPC, local file을 사용해 통신하는 여러 process로 구성된 application을 생각해보면, 같은 node에서 실행되어야한다.
- container는 단일 process를 실행하는 것을 목적으로 설계가 되었다. 단일 container에서 다른 process를 실행하는 경우 이 process를 관리하는건 모두 process를 실행한곳에서 관리해야한다.
	- 이 process 관리에는 log, process 종료 시 다른 process 종료등의 관리가 모두 포함된다.
    
이러한 케이스를 생각해 볼때는 process의 관리를 kubernets가 담당하는것이 더 좋으며, 각각의 개별 container로 구성한 다음 관리하는것이 더욱 좋은방법이다.

여러 process를 다중 container로 구성해 관리하기 때문에 pod이란 개념이 필요하다.
다중 container로 구성해도 언제나 동일한 환경(storage, network)처럼 동작이 가능하다.

pod 내부의 container들은 동일한 `linux namespace`에서 실행되므로, host와 network interface를 공유한다. 하지만 file system의 경우는 약간 다르다. kubernetes에서는 file system 공유를 위해 `volume`이란 개념을 사용한다.

또한 다른 주의점은 동일한 network interface를 공유하기 때문에 동일한 IP와 port 번호를 공유하기 때문에 container끼리는 port가 동일하면 안된다.

kubernetes에서 관리되는 pod은 라우팅 가능한 IP 주소를 가지고 있으며, 다른 pod에 IP를 이용하여 접근 가능하다.

이게 가능한 이유는 `flat network`가 있기 때문이다. docker의 `brige network`가 flat network의 한가지 예시이다.

## pod의 적절한 구성
개별 확장이 가능한 단위로 구성하는것이 pod을 구성하는 가장 적절한 방법이다.

서로 다른 host에서 실행 될 수 있는 container는 개별의 pod으로 구성하는것이 가장 적절하다.

## Pod 생성 방법
pod을 생성하기 위해서는 `JSON` 또는 `YAML`로 구성된 kubernetes Object를 kubernets REST API endpoint에 전송하면 가능하다.

`kubectl`의 명령어를 사용해서 배포가 가능하지만, 버전 관리가 어려워지며, 어떤 pod이 배포되었는지 명시적으로 파악하기 어렵기 때문에 추천하지 않는다.

kubia-manual.yaml
```
apiVersion: v1 					-- kubernetes의 api version
kind: pod					-- Object의 종류
metadata:
  name: kubia-manual  				-- Pod의 이름
spec:
  containers:
  - images: luksa/kubia				-- Container의 image
    name: kubia					-- Container의 이름
    ports:
    - containerPort: 8080			-- Container의 port
      protocol: TCP				-- Container가 수신할 protocol
```

`kubectl create -f kubia-manual.yaml`
이러한 yaml을 작성한 후 `kubectl create` 명령어를 이용해 pod의 배포가 가능하다.

## pod에 관련된 명령어
`kubectl get pod kubia-manual -o yaml`
- kubia-manual이란 이름을 가진 pod의 output을 yaml 형식으로 출력

`kubectl get pod kubia-manual -o json`
- kubia-manual이란 이름을 가진 pod의 output을 json 형식으로 출력

`kubectl get pods`
- 현재 namespace에서 동작되는 pod의 목록을 노출

`kubectl logs kubia-manual`
- kubia-manual의 log를 가져오기

`kubectl logs -f kubia-manual`
- kubia-manual의 log를 실시간으로 가져오기

`kubectl logs -f kubia-manual -c kubia`
- 다중 컨테이너로 구성된 경우 컨테이너를 지정하여 log 가져오기

## pod에 요청 보내기
pod을 배포했지만 아직 endpoint가 외부로 노출되어 있지 않다. service란 object가 추가로 필요하다.

하지만 pod에 요청을 보내는 방법은 존재한다.
`port-forward`명령어를 이용하면 로컬 네트워크의 port를 pod의 port로 forwarding이 가능하다.

`kubectl port-forward kubia-manual 8080:8080`
local의 8080 port로 들어온 request를 pod의 8080 port로 forwarding하는 명령어이다.


