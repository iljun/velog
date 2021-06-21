# PersistentVolume
Pod에서 Container와 Lifecycle을 같이하는 Volume을 유지해야하는 경우가 아니라면 PersistentVolume을 사용해 저상소를 유지할 수 있다.

PersistentVolume을 이용해 개발자는 언제든지 필요한 storage를 요청해 사용 가능하다.

이 PersistentVolume이 존재하는 이유는 Pod에서 필요한 storage를 직접 작성한다면, kubernetes의 철학에 어긋나게 된다. Cloud 환경에서도 자유롭게 이동이 가능하며, onpremise 환경, cloud + onpremise 환경에서도 이식성이 좋게 작성되어야한다. 
Cloud 환경에 종속적인 volume, onpremise 환경에서 직접 관리되는 volume을 pod에 직접 작성하고 관리해야한다면, 개발자의 입장에서는 신경써야할 부분이 너무 많아지며, 이 volume은 cluster 관리자만 알고 있으면 충분한 정보이다.

이러한 문제점을 해결하기 위해서 추가된 Object가 `PersistentVolume`, `PersistentVolumeClaim`이다.

먼저 이 2가지의 동작 과정을 살펴보면
1. cluster 관리자는 storage 유형을 설정한다.
2. cluster 관리자는 이 storage를 PersistentVolume으로 등록한다.
3. 개발자는 PersistentVolumeClaim을 생성한다.
4. kubernetes는 PersistentVolumeClaim에 적당한 PersistentVolume을 찾고 바인딩한다.
5. 개발자는 배포하려는 pod에 PersistentVolumeClaim만 추가하여 배포한다.

즉 개발자의 입장에서는 어느 storage이건 관계없이 PersistentVolumeClaim을 이용하여 일관적인 pod을 생성 가능하며, 특정 환경에 종속적이지 않은 pod을 구성 가능하다. 추상화를 통해 특정 환경의 종속성을 줄인 방법이다.

## Provisioning PersistentVolume
cluster의 관리자는 계속 사용해야할 PersistentVolume을 미리 프로비저닝을 해야 다양한 scale out에 대처가 가능하다.
onpremise환경에서는 어쩔수 없이 관리자가 사전에 storage를 생성해야하지만, cloud 환경에서는 상태에 따라 동적으로 생성이 가능하다.

동적 provisioning을 위해서는 storageClass를 생성해야한다.
sotrageClass는 persistentVolumeClaim이 storageClass에 요청할 때 provisioner가 어떤 persistentVolume을 사용할지 결정하는 역할이다.

