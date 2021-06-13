# Daemonset
Cluster에 존재하는 모든 Node마다 하나의 Pod을 실행하기 위해 사용하는것이 Daemonset이다.
주로 시스템 수준의 작업을 수행하는 Pod이다. 로그 수집기, 리소스 모니터링 등이 주로 Daemonset으로 사용된다.

Kubernetes에서 제공하는 Daemonset은 kube-proxy가 존재한다. 이 kube-proxy는 모든 노드에서 서비스를 작동시키기 위해 배포된다.
일반적으로 systemd를 이용해 각 노드마다 동일한 application을 실행하거나 script를 실행한다. kuberntes에서도 이렇게 실행이 가능하나 이러한 경우의 kubernetes의 모든 기능을 사용하기 어렵다.

Daemonset Object를 생성하고 배포될 PodTemplate을 작성하면, 노드가 추가 및 제거될때에도 자동으로 Node에 Pod이 배포된다. 

Daemonset으로 배포되는 Pod은 노드가 이미 지정되어있고, kubernetes scheduler를 건너뛴다는것을 제외하고는 일반적인 Pod과 동일하다.

다만 Daemonset은 replicaset의 개념이 존재하지 않는다. Pod Selector가 일치하는 Pod이 각 노드에서 실행중인지 판단하는게 Daemonset의 역할이다. 그러므로 replica의 개념은 필요하지 않다.

특정 Node에만 배포되는 Daemonset을 생성하려면 `Node-Selector` 속성을 지정하면 된다.
Node-Selector가 동일한 Daemonset에만 Daemonset이 배포된다.

```
apiVersion: apps/v1beta2
kind: Daemonset
metadata:
  name: ssd-monitor
spec:
  selector:
    matchLabels:
      app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
    spec:
      nodeSelector:			-- disk=ssd label이 존재하는 Node에만 배포되는 Daemonset이다.
        disk: ssd
      containers:
      - name: main
        image: luksa/ssd-monitor
```

node에 label을 붙이는 방법은 간단한다.
```kubectl label node {nodeName} disk=ssd```

