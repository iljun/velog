# Namespace
Kubernetes는 하나의 Cluster를 기반 동작하는 가상 클러스터를 지원한다.
즉 하나의 Cluter 내부에서 여러 Cluster를 사용하는 것과 같은 효과를 지닐 수 있다.
이걸 가능하게 해주는 개념이 바로 `Namespace`이다.

## Namespace의 사용 이유 
- 여러가지 종류의 Object가 존재할때 이 Object를 작은 그룹으로 분리 할 수 있다.
- 분리된 Namespace에서는 동일한 Object의 Naming을 사용 할 수 있다.
  - Object의 name은 Namespace 내부에서 유일해야 한다.

물론 일부 Object는 Namespace에 속하지 않는다.
ex) Node Resource (Daemonset)

`kube-`로 시작하는 Namespace는 Kubernetes system에서 사용하는 namespace이다.


`kubectl get namespace`

kubernetes를 사용할때 기본적으로 4가지의 namespace가 제공된다.
- default
  - namespace가 존재하지 않는 Object을 위한 기본 namespace
- kube-system
  - Kubernetes System에서 생성한 Object들이 위치하는 namespace
- kube-public
  - Cluster에서 공개할 필요가 있는 Object들이 위치한다. 
  - required는 아니면 관례로 사용된다.
- kube-node-lease
  - scaling을 할 때 Node의 heartbeat의 성능을 향상 시키는 Node와 관련있는 Object들이 위치한다.
  
#### Namespace를 삭제하게 되면 삭제하는 Namespace에 속한 Object는 모두 삭제된다.