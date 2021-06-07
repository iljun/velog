# Label
kubernetes Object에 key-value 형태로 값을 붙이는게 label이다.
Object의 특징을 추가하여 사용자에게 노출되게 하는게 label의 목적이다.

label로 Object의 집합을 구성하고, 특정 label로 Object들을 한번에 조회도 가능하다.
label은 언제든지 생성, 수정, 삭제가 가능하다.

microservice를 적용하는 경우 구성하는 서비스가 매우 많은 숫자가 된다.
게다가 여러 버전이 동시에 동작하는 경우도 있다.(stable, beta, canary)
이러한 경우는 관리가 매우 어려워지기 때문에 label을 이용해서 개발자의 관리 용이성을 높일 수 있다.

라벨을 지정할때는 Object yaml에 정의해 쉽게 관리가 가능하다.

```
apiVersion: v1
kind: Pod
metadata:
	name: kubia-manual-v2
    labels: 
    	creation_method: manual		-- key: creation_method value: manual
        env: prod			-- key: env value: manual
spec:
	containers:
    - image: luksa/kubia
      name: kubia
      ports:
      - containerPort: 8080
        protocol: TCP
```

현재 pod에 붙은 label을 보는 명령어는 간단하다.
```kubectl get pod --show-labels```

label의 특정한 값으로 노출하려면 -L 스위치를 지정해주면 된다.
```kubectl get pod -L creation_method,env```

이미 동작중인 Object에 label을 추가하려면
```kubectl label pod kubia-manual creation_method=menual```

label의 값을 수정하려면 --overwrite 명령어를 추가하면된다.
```kubectl label pod kubia-manual env=debug --overwrite```

label을 이용하여 부분집합을 조회하는 방법
```kubectl get pod -l creation_method=manual```

label을 가지고 있지만 값은 무엇이든 상관없는 Object를 조회하려면
```kubectl get pod -l creation_method```

표현식을 이용해 조회도 가능하다.
```kubectl get pod -l '!creation_method'```

## Label로 스케쥴링 제한하기
일반적인 상황에서는 개발자는 스케쥴링을 신경쓰지 않아도 된다.
yaml에 명시해놓은 resource와 CPU 사용량만으로 kubernetes가 적절히 각 node에 배치한다.

하지만 node의 spce이 약간씩 다른 경우도 존재할 수 있다.
추가로 특별한 pod은 특정 node에만 배치되어야 하는 경우도 있을 수 있다.
이러한 경우에 label을 이용해 스케쥴링을 제한 할 수도 있다.

이때는 node에 label을 추가하고, `nodeSelector`라는 option을 이용해 배치된 node를 선택 할 수 있다.

```
apiVersion: v1
kind: Pod
metadata:
	name: kubia-manual-v3
spec:
	nodeSelector:
    	gpu: "true"		-- gpu=true라는 label을 가진 node에 배포된다.
	containers:
    - image: luksa/kubia
      name: kubia
      ports:
      - containerPort: 8080
        protocol: TCP
```