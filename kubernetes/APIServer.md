# API Server
kubernetes의 master node에서 동작하는 API Server이다.

API Server는 etcd라는 저장소에 직접 접근하는 역할을 담당하고, 실제 kubernetes object를 생성 및 수정 할 때의 validation과 다른 component들에서 etcd에 접근하기 위한 interface 제공
etcd 저장소에 동시성을 책임진다.

kubectl에서 주로 사용하는 중요한 Server이며, Restful API로 CRUD 인터페이스를 제공한다.

물론 이 API Server에서는 인증, 인가를 모두 담당하기에 kubernetes의 RBAC를 이용해 user별 권한을 제한 할 수 있다.

API Server의 존재로써 etcd 저장소에 동시성과 객체의 유효성을 검증한다.
etcd는 kubernetes의 object (deployment, configmap, pod의 정보...)을 저장하는 저장소이다.

이 API Server와 etcd는 여러 node에 걸쳐 성능 향상 및 장애 상황에 대응할 수 있게 구성해야한다.

여러대의 node에 걸쳐 API Server와 etcd를 구성하게 된다면 장애 상황에 대응 할 수 있지만
결국 etcd의 정보의 동기화가 매우 중요하다.

etcd는 RAFT 합의 알고리즘을 사용해 각 노드 상태가 대다수의 노드와 동일한 상태가 되어야 데이터의 생성, 변경이 가능합니다.

이 RAFT 합의 알고리즘을 효율적으로 사용하기 위해 ectd 인스턴스를 보통 홀 수로 구성한다.
3개의 인스턴스를 예를 들어보면 하나의 인스턴스에 장애가 발생했을때 다른 2개의 인스턴스에 정상적으로 반영이 되었다면 하나의 인스턴스가 장애 상황이라도 원활한 운영이 가능하다.
2개의 인스턴스를 예를 들어보면 하나의 인스턴스에 장애가 발생했다면 100% 확률로 전체 클러스터에 장애가 발생한다.

이 API Server의 중요한 부분은 etcd에 object를 저장하는 역할만 담당한다. 
실제 object가 변경되었을때 다른 client에 통보하지 않는다.
각각의 역할을 담당하는 clients는 API 서버에 connection을 맺어놓고 변경사항을 직접 감지한다.
그렇기 때문에 client는 각각의 역할만을 담당할 수 있다.


