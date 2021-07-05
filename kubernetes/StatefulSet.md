# StatefulSet
StatefulSet은 언제나 동일한 network와 host, 지속성 있는 volume, 순차적인 배포가 필요한 Pod을 관리하기 위한 Resource이다.

그렇다면 ReplicaSet과 다른점은 무엇일까?
ReplicaSet은 언제나 대체될 수 있는 Pod을 생성한다. 즉 언제나 동일한 network, host가 아니며, volume의 지속성을 가지기 어렵고, 순차적인 배포가 일어나지 않을 수 있다.

한가지 예시를 들어보자면 영속성이 있는 volume을 계속 유지해야하는 Pod을 ReplicaSet으로 구성한다면 어떻게 구성해야할까?
ReplicaSet의 replica를 1개로 설정하고, 언제나 동일한 pvc를 설정해야한다.
그렇다면 scale out은 어떻게 해야할까? ReplicaSet의 개수 자체를 늘리는 방법만 존재한다.
이러한 복잡하고 scale out이 어려운 상태로 쉬운 관리는 불가능하다.

statfulSet의 특징으로 관리되는 pod은 순차적인 번호를 가진다.
scale out을 진행하면 순차적으로 pod의 번호가 증가되며, scale down이 발생할때는 가장 큰 번호를 가진 pod이 삭제된다.

statefulSet의 pod template을 변경해도 기존에 존재하던 pod은 새로운 pod으로 변경되지 않는다.
그렇다면 pod template이 변경 되 었을때 updateStrategy를 조정하여 pod template이 변경되면 교체할 수 있도록 설정 가능하다.

## VolumeClaim
StatefulSet은 지속성 있는 volume을 유지한다.
그렇다면 어떻게 지속성 있는 volume을 유지할까?
바로 StatefulSet을 생성할때 pvc을 생성하여 binding한다.
이때 유의할 점은 pvc binding을 위해서 pv의 동적 provisioning이 필요하다.

그리고 pod이 scale down 될 때 pvc는 삭제되지 않는다. 이 pvc가 같이 삭제될 시 데이터의 유실이 발생할 수 있기 때문이다.

## governing headless service
동일한 network를 제공하기 위해서 governing headless service를 생성해 각 pod에 실제 network를 바인딩 해야한다.
이 governing headless service의 역할은 각 pod이 자체 DNS entry를 가지게 하여, cluster 내부 또는 외부에서 host를 통해 pod에 접근하기 위해서이다.

이 의미는 statefuleSet으로 관리되는 pod은 각각 서로 다른 pod으로 판단을 해야한다. 같은 pod으로 구분하지 않기 때문에 상황에 맞게 접근 할 수 있는 방법이 필요하다.

governing headless service는 ClusterIP를 None으로 설정하여 생성한다.


StatefuleSet에서 더 중요한 점은 바로 pod이 완벽하게 실행중이지 않은지 판단해야한다는 점이다.
Pod이 완벽하게 실행중이지 않은 경우에 새로운 Pod을 생성해야 동일 pvc에 binding 하지 않게된다.
그렇기 때문에 statefulSet의 Pod에 대한 재생성은 pod을 수동 삭제 또는 생성된 node를 삭제해야한다.
