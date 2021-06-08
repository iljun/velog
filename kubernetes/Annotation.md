# Annotation
Label과 비슷하게 Kubernetes Object에 annotation을 추가할 수 있다.
annotation은 label과 비슷해보이지만 식별하기 위한 key-value의 값이 아니다.
식별자가 존재하지 않기 때문에 label처럼 검색이 불가능하다.

## 역할
개발자가 아닌 library등에서 붙이거나 개발자가 특정 annotation을 붙여 library가 동작하게 하는 역할을 가진다. 즉, kubernetes의 새로운 기능을 추가할때 사용한다.

특정 annotation은 kubernetes에서 자동으로 붙이지만 대부분의 annotation은 개발자가 직접 붙여야한다.

kubernetes의 특정한 API도 이 annotaion을 이용해 활성화 시키고 사용되게 할 수 있다. sidecar pattern으로 envoy proxy를 주입할때도 사용된다.

로깅, 모니터링, 분석을 위한 pointer로도 이용이 된다. 
annotaion을 추가하여 특정한 Object에만 로깅, 모니터링, 분석을 활성화 시킬 수 있다.


Annotaion의 추가 
```kubectl annotate pod kubia-manual iljun.me/annotation="foo bar"```
--> iljun.me/annotaion을 foo bar라는 값과 함께 추가했다.
annotaion을 적용할때는 key값에 유의해야한다. 일반적으로 접두사를 추가하여 사용되며, 접두사가 없이 사용하다 다른곳에서 사용되는 annotaion과 중복이 되게되면 override하게 된다.
ex) `kubernetes.io/`와 `k8s.io/`는 kubernetes system에서 사용하는 접두사이다.


describe 명령어를 사용하면 pod의 자세한 정보와 annotation의 정보를 모두 볼 수 있다.
```kubectl describe pod {podName}```

