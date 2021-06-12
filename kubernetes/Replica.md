# Replication Controller
Replication Controller는 Kubernetes의 Resource로 Pod이 항상 실행되도록 보장하는 Object이다. Pod에서 Crash가 발생하거나, 다른 이유로 삭제되면 Replication Controller에 정의된 대로 Pod을 새롭게 생성한다.

또한 Replication Controller는 Pod의 replica까지 관리가 가능하다.

Pod을 Object로 직접 생성하면 Pod에 문제가 생겨서 process가 죽어도 재시작 되지 않는다.

Replication Controller의 정확한 역할은 현재 동작중인 Pod이 몇개가 실행중인지를 감시하는 역할이다.
정상 동작중인 Pod이 Replication Controller에 선언된 replica와 동일한지를 감시한다.
(여기서 Replication Controller가 감지하는 Pod은 특정 label selector와 match 되는 pod이다.)

## Replication Controller의 3요소
- Label Selector
  - Replication Controller가 감지할 Pod을 결정하는 요소
- Replica Count
  - 실행할 Pod의 개수를 지정
- Pod Template
  - Pod의 개수가 달라져 새롭게 Pod을 생성할때 참조하는 정보
  
이미 실행중인 Replication Controller의 Label Selector을 수정한다면 Replication Controller는 이전에 동작중인 Pod을 더이상 관리하지 않게된다. 이미 동작중인 Pod은 계속 실행이 되겠지만, Crash등의 문제로 삭제가 되도 다시 실행되지 않는다.

Pod Template을 변경하게 된다면 이전에 관리되던 Pod은 모두 재시작이 될까? 아니다. 
Pod Template은 새롭게 생성하는 Pod의 참조 정보이므로, 이전에 유지되던 Pod은 영향을 받지 않고 새롭게 생성되는 Pod에만 변경된 Pod Template가 적용된다.

## Replication Controller가 Pod의 상태를 감지하는 방법
정확히 Replication Controller가 모든 Pod의 상태를 감시하는것은 아니다. 실제로는 kubelet이 Pod의 상태를 확인하고, 현재 상태를 kubernetes로 전달한다.

Pod이 정상 상태인건 어떻게 판단할까? crash는 명확하게 container가 중지하지만, OOM의 경우 container는 중지되지 않고 동작한다. Pod이 정상 상태라는 의미는 application이 요청을 처리할 수 있는 상태라는 의미이다.

#### liveness probe
개발자가 정의한 `livevess probe` 를 이용해 Container가 정상 상태인지 파악할 수 있다.
주기적으로 liveness probe를 실행해 Container의 상태를 파악한다.

Kubernetes에서는 3가지의 liveness probe를 제공한다.
- HTTP
  - HTTP GET으로 지정한 IP, port, path를 호출해 현재 application의 상태를 파악한다.
  - HTTP status가 2xx, 3xx이면 정상 상태라고 판단한다.
  - HTTP timeout은 정상 상태가 아니라고 판단한다.
- TCP
  - 지정된 port에 TCP 연결을 시도하고, TCP 연결이 성공한다면 정상 상태라고 판단한다.
- EXEC
  - 컨테이너 내부에서 임의의 명령을 실행하고, 종료 상태 코드를 판단해 Application의 상태를 확인한다.
  - 종료 상태 코드는 0 이외는 전부 실패로 판단한다.
  
liveness probe는 최대한 가볍게 유지해야한다.
liveness probe는 자주 실행되므로, liveness probe의 로직이 오래걸리고 무거우면 Container의 성능에 영향을 미친다.


## Horizontal Scale
Replication Controller로 동일한 Pod을 수평 확장도 가능하다.
Replication Controller의 replicas field를 수정하면 즉시 pod의 수를 조정가능하다.

Replication Controller의 template을 수정하는 방법도 존재하지만, kubectl의 명령어로도 가능하다.

`kubectl scale rc --replicas={count}`
지정된 replicas의 개수만큼 pod을 관리하게 된다.

# ReplicaSet
ReplicaSet은 Replication Controller를 대체하는 Object이다.
ReplicaSet와 Replication Controller는 거의 동일하기 때문에, 사용법과 변경에 문제가 거의 없을것이다.
Pod의 Replica를 관리해야 한다면 ReplicaSet을 사용하길 권장한다.

## 차이점
ReplicaSet은 Replication Controller보다 조금 더 풍부한 Pod Selector의 표현식을 가지고있다.
ReplicaionController는 label selector를 이용해 특정 label이 있는 pod만 관리가 가능하지만,
ReplicaSet은 특정 label의 key키를 가지는 Pod 모두를 관리할 수 있다.
하나의 ReplicaSet으로 다수의 Pod Group을 관리할 수 있다.

