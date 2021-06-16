# Service
Service란 Pod을 네트워크 서비스로 노출시켜 외부에서 Pod에 Request를 전달할 수 있게하는 Object이다.

이 Service 대신에 Pod의 IP주소를 사용하면 되지 않을까라는 생각을 할 수 있지만, 이러한 방법은 운영단계에서 매우 비효율적이고 관리가 되지 않는 방법이다.
- Pod은 언제나 삭제되고 재실행 될 수 있다.
  - 즉 IP주소가 언제나 변경될 수 있다는 점이다.
- Pod의 IP 주소는 노드에 Pod이 스케쥴링 된 다음 할당된다.
  - 재시작된 Pod의 IP 주소를 정확하게 알 수 없다.
- loadbalancing을 지원해야한다.
  - 대부분의 서비스와 동일하게 호출하는 client에서는 하나의 단일 endpoint로 호출해야한다.

이러한 이유 때문에 Kubernetes에서는 Service란 Object를 제공한다.
이 Service Object는 외부에 노출될 IP를 할당받게되고, 이 IP로 request를 전달하면 동일한 Service에서 관리되는 Pod에 Request가 분배된다.
(Service에서 관리된다는 의미는 label selector로 지정된 pod이다.)
Service가 유지되는 동안 한번 생성된 IP주소는 변경되지 않는다.

```
apiVersion: v1
kind: Service
metadata:
  name: kubia
spec:
  ports:
  - port: 80			-- service가 사용할 port
    targetPort: 8080		-- service가 request를 forward할 pod의 port
 selector:
   app: kubia
```

##### sessionAffinity
특정 client의 모든 요청을 동일한 pod으로 전달 하기 위해서 사용하는 속성이다.
`sessionAffinity: ClusterIP`로 지정하면 동일한 client의 request는 동일한 pod에 전달된다.
client와 관계없이 부하를 분산하려면 `sessionAffinity: None` 속성을 사용해야한다.

sessionAffinity의 속성을 보면 cookie 기반으로도 routing이 가능하지 않을까라는 의문점이 들 수 있지만, service 객체는 HTTP 수준에서 동작하는 Object가 아니기 때문에 cookie는 지원하지 않는다. 실제 Service는 TCP, UDP의 packet을 전달하는 역할만하지 payload, header등은 전혀 신경쓰지 않는다.

### Service가 Pod의 IP 주소를 어떻게 알까?
Service의 IP는 바뀌지 않지만, Pod의 IP 주소는 언제든지 바뀔 수 있다.
그렇다면 Service는 request를 전달할 Pod의 IP 주소를 어떻게 파악할까?
Pod이 실행되면 kubernetes는 해당 시점에 존재하는 각 service를 지칭하는 환경 변수를 초기화한다.
service는 이 환경변수를 참조해 Pod의 IP를 획득한다.

### DNS
kubernetes의 DNS 서비스를 실행하면 cluster 내부에서 실행중인 모든 pod은 자동으로 DNS 서버를 사용할 수 있게 된다.

pod에서 DNS 서비스를 사용하게 되면 이는 kubernetes 내부 DNS를 먼저 사용하게 되므로, cluster 내부에서도 DNS를 사용할 수 있게 된다.

#### FQDN
cluster 내부에서 사용할 수 있는 DNS이다.
도메인의 pattern은 `<serviceName>.<namespace>.svc.cluster.local`의 형태이다.
`svc.cluster.local`은 clueter의 local service에 사용되는 접미사이다.

### Service를 외부로 노출
Service 객체와 Endpoint 객체를 구성하면 외부에서 접근 가능한 endpoint를 만들 수 있다.
```
apiVersion: v1
kind: Endpoints
metadata:
  name: external-service	-- service의 이름과 동일해야한다.
subsets:
  - addresses:
    - ip: 11.11.11.11		-- 서비스에 연결될 IP의 항목
    - ip: 22.22.22.22		-- 서비스에 연결될 IP의 항목
  ports:
  - port: 80
```

endpoint Object를 구성하면 외부에서 public IP에 domain을 설정하고, kubernetes로 연결이 가능하다.

Service에 IP가 아닌 FQDN을 이용하면 IP의 주소가 아니라 cluster의 DNS를 이용해 mapping도 가능하다.
```
spec:
  type: ExternalName
  externalName: <FQDN>
  ports:
  - port: 80
```

### ServiceType
외부에서 접근 가능한 Service를 구성하는 방법은 3가지가 존재한다.
#### ClusterIP
서비스를 클러스터-내부 IP에 노출시켜 cluter 내부에서만 접근 가능하게한다.
#### NodePort
각 Node에서 port를 열고, 해당 port로 전달되는 request를 각 서비스로 전달하는 방식이다.
#### LoadBalancer
cloud 환경에서 사용되는 방법으로 cloud에서 제공하는 loadbalancer를 이용해 Service로 access할 수 있게 한다. 이때 LoadBalancer를 통한 request는 NodePort의 방식과 동일하게 Node의 Port로 전달된다. cloud 환경에서 제공하는 kubernetes(EKS, AKS)는 Service 객체를 생성하면 자동으로 loadbalancer를 생성하며 mapping한다.
#### Ingress 
HTTP Level에서 동작하는 Object이다. ingress의 중요한점은 public IP를 하나만 가지고 모든 request를 받아 전달한다는 점이다. loadbalancer를 사용할 경우 각 endpoint마다 public IP가 할당이 되어야한다. 하나의 public IP가 할당되고 이 ingress는 host와 path를 해석해 어느 Service로 request를 전달할지 판단한다. 이 Ingress가 HTTPS를 지원하기 위해서는 TLS 인증서를 cluster 내부에 저장하기만 하면된다.
#### ExternalName
CNAME 레코드를 리턴하여, 서비스를 externalName 필드에 매핑한다.
### readiness probe
service가 pod으로 언제나 request를 전달 할 수 있을까? 아직 pod이 완벽하게 실행된 경우가 아니면 request를 전달하면 안된다. 이러한 경우를 방지하기 위한 개념이 `readiness probe`이다.

즉 container가 request를 처리할 준비가 되었는지 또는 request를 계속 처리할 수 있는지 파악하는 endpoint이다. readiness probe는 liveness probe와 동일하게 TCP, HTTP, script EXEC를 지원한다. 

readiness probe가 실패하게 되면, service에서 pod의 목록을 제거해 request가 전달되지 않도록 한다.