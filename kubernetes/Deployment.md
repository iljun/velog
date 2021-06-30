# Deployment
ReplicaSet을 이용하여 어플리케이션을 배포하고 선언적으로 업데이트가 가능한 high-level 리소스이다.
Deployment를 생성하면 ReplicaSet이 생성되며, ReplicaSet은 pod을 생성하게된다.

Pod의 관리는 ReplicaSet이 담당하는데 Deployment는 왜 필요할까?
개발자가 직접 rolling update등의 작업을 실행하지 않게하며, kubernetes가 이러한 일을 모두 처리하게 하기 위함이다.

Deployment를 사용하지 않고 pod의 업데이트 및 배포를 관리하려면 손이 많이 가는 작업이며, 언제든지 실수가 발생할 수 있다.

Pod의 업데이트 방법 
- 기존 pod을 모두 삭제하고 새로운 pod을 배포(Recreate)
- 순차적으로 하나의 pod을 삭제하고, 새로운 pod을 배포하는 형태로 모든 pod을 교체 (RollingUpdate)

pod의 template을 변경하고 기존 pod을 하나씩 삭제하여 업데이트 할 수 있다. 이 과정에서 언제든지 실수가 발생 할 수 있고, 이 과정은 어찌되었든 순간 downtime이 발생하게 된다.
또 다른 방법으로는 대체할 새로운 pod의 그룹을 먼저 생성하고 service의 설정을 변경하여 트래픽을 전환하는 방법도 존재한다. 

Deployment는 이러한 과정을 모두 처리해주며, 개발자의 실수 없이 Pod의 업데이트가 가능하게 한다.
Deployment 내부의 pod template을 수정하여 yaml 파일을 kuberentes에 적용하기만 하면 변경된 내용으로 pod이 업데이트가 된다.

Deployment는 Pod 업데이트시 새롭게 배포된 Pod이 문제가 생겨 정상적으로 실행되지 않으면 자동으로 rollback하는 기능도 가지고 있다.

rollingUpdate의 전략도 선언이 가능하다. 
- maxSurge
  - Deployment에 의해 생성된 Replica보다 얼마나 많은 수의 pod을 유지할지에 대한 설정이다.
  - Default는 25%로 설정되며, rollingUpdate를 진행하면서 미리 실행할 Pod이 어느정도 비율로 늘어날지 설정하는 값이다.
- maxUnavailable
  - 업데이트 도중 사용할 수 없는 Pod의 수를 결정하는 Field이다.
  - 즉 업데이트 도중에 몇개의 Pod이 응답을 처리할 수 있게할지에 대한 설정이다.

이 maxSurge와 maxUnavailable을 통해 배포 전략을 다양하게 세울 수 있다.
