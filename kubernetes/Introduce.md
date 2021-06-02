# Kubernetes
그럼 Kubernetes란 무엇인가?
컨테이너 환경으로 작성된 application을 자동으로 배포하며, application에 장애가 발생시 자동복구, auto sacle out을 제공하는 관리 시스템이다.

## 특징 
- 확장성 
	- 서로 독립적으로 확장해야 하는 Microservice에서는 특히 필요한 특징이다.
- 유연성 
	- 어떠한 환경이라도 일관된 방법으로 배포, 운영, 테스트가 가능한 유연성을 가지고 있다.
- 이식성
	- cloud, on-premise, cloud + on-premise 등 다양한 환경에서도 동작 가능한 이식성을 가지고 있다.
    
그렇다면 Kubernetes를 왜 써야할까?
우선 하나의 가정은 수 많은 MicroService를 개발 및 운영한다고 가정한다.
MicroService의 특징 중 하나로 application이 각각 확장하고, 복잡한 거미줄 처럼 서로 얽혀있다는 점이다.

MicroService architecture를 사용하는 입장에서 사용자의 요구가 하나씩 늘어나고 유지보수를 진행하면서 전체 architecture는 당연히 복잡해지며, 많은 수의 서비스가 동작하면서 복잡해지는 배포 과정, 서로간의 네트워크 통신이 필요한 구조에서 개발자 및 데브옵스는 너무나 할게 많아진다. 또한 서비스의 개수가 많아지면서 관리할 서비스의 개수가 많아진다.

kubernetes는 이러한 모든걸 해결해준다.

kubernetes는 서로간의 통신에서 자체 DNS를 가지고 있어 application이 어느 node에서 동작하던지 관계없이 요청을 전달 할 수 있으며, 간단한 Object 정의(yaml, json)로 쉽게 application을 배포 가능하며, 자동으로 application의 장애를 감지해 새로운 application으로 대체하여 장애 상황에서 벗어날 수 있게한다.

