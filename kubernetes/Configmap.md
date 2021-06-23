# Configmap
Container가 실행될 때 설정 정보가 필요한 경우가 대부분이다.
외부 Container와 통신하기 위한 endpoint, DB connection의 정보 등등 다양한 정보가 포함 될 수 있다.

Container가 실행될때 인자를 주입하는 방법에는 실행 command에 환경 변수를 주입하는 방법이나, Dockerfile을 이용해 image를 빌드할때 밀어넣는 방법이 있다. 하지만 후자는 추천하지 않는다. 환경변수가 변경될때마다 새롭게 image를 빌드해야하기 때문이다.

Kubernetes에서는 pod에 환경 변수를 전달하기 위해 Configmap을 이용한다.
Configmap을 이용하면 환경변수의 하드코딩이나 Container 실행에 일일이 환경변수를 전달할 필요가 없어진다.

이 configmap은 단순하게 key/value 형태의 값을 가지는 Object이다.
Pod은 이 configmap을 직접 읽는것이 아니라, configmap을 환경변수 또는 volume 파일으로 전달되어 사용된다.
configmap을 이용함으로써 얻을 수 있는 장점은 하나의 image로 다양한 환경에 대응 가능한 Container를 구성가능하며, 설정 정보의 관리가 편리해진다.
파일 전체를 configmap에 지정하여, json 파일로 관리되는 설정도 한번에 주입이 가능하다.

그러면 pod에서 configmap을 사용하려면 어떻게 해야할까?
#### valueFrom
특정 configmap의 값을 key로 추출해 직접 주입하는 방법이다.
```
apiVersion: v1
kind: pod
metadata:
  name: ...
spec:
  containers:
  - image: ...
    env:
    - name: ...
      valueFrom:
        configmapKeyRef:
          name: 		-- 참조하는 configmap의 
          key: 			-- configmap 내부에 존재하는 key값
```

#### envFrom
configmap의 모든 값을 pod에 주입하는 방법이다.
```
spec:
  containers:
  - image: ...
    envFrom:
    - prefix: ...		-- 환경변수가 prefix를 가질 경우 필요 
      configMapRef:
        name: 			-- configmap의 이름
```

#### mount
configmap의 내용을 volume에 mount하여 사용하는 방법이다.
```
spec:
  containers:
  - image: ...
    name: ...
    volumeMounts:
    - name: config		-- configmap이 mount된 경로를 사용한다.
      mountPath: ...	
      readOnly: true
  volumes:
  - name: config
    configMap:			-- 이 volume은 configmap을 참조한다.
      name: ...
```

이 configmap은 application의 재시작 없이 설정값이 수정 가능하다.
그러나 application에서 설정정보를 꾸준히 읽어오지 않는다면 변경이 되지 않는점은 파악해야한다.
