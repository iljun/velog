# Volume
Pod은 내부에 process가 실행되고 CPU, RAM, Network Interface등의 리소스를 사용한다. 하지만 디스크는 조금 다르다. Pod 내부의 컨테이너들은 각각 분리된 파일 시스템을 가진다. 분리된 파일 시스템을 사용하는 이유는 파일시스템은 컨테이너 이미지에서 제공되기 때문이다.

컨테이너 내부의 파일은 컨테이너 이미지 빌드시의 추가한 파일들이 위치하며, 컨테이너가 재 시작할시 이전 컨테이너의 파일 시스템을 볼 수 있는 방법은 없다. 파일 시스템이 계속 유지가 될 필요가 없는 컨테이너라면 이러한 특성은 크게 문제가 되지 않지만, 파일 시스템을 계속 유지해야한다면 큰 문제가 생길 수 있다.

Kubernetes는 storage volume을 정의하는 방법으로 컨테이너가 특정한 파일 시스템을 계속 사용할 수 있도록 해주며, 여러개의 컨테이너가 이 storage를 공유할 수 있도록 해준다.

volume은 pod의 spec으로 정의되며, 자체적으로 생성, 삭제 할 수 없다. Pod의 내부 컨테이너에서 사용이 가능하지만 사용하려면 사용하려는 컨테이너에서 각각 mount 해야한다.

동일한 volume을 각각의 컨테이너에 mount하게되면 각각의 컨테이너는 이 volume을 공유해 사용하게 된다.

### Volume의 유형
- emptyDir: 일시적인 데이터를 저장할 때 사용하는 빈 directory이다.
- hostPath: node의 파일 시스템을 pod의 디렉토리로 mount할때 사용된다.
- gitRepo: git repository를 chekcout하여 초기화한 volume이다.
- nfs: NFS 공유 volume이다.
- gcePersistentDisk, awsElasticBlock, AzureDisk: cloud에서 제공하는 전용 volume이다.
- configMap, secret, downwardAPI: kubernetes Resource, cluster 정보를 pod에 mount하기 위해 사용하는 volume
- persistenceVolumeClaim: 동적으로 provisioning된 persistence storage를 사용하는 방법이다.


##### emptyDirs
Pod의 lifecycle과 같이 움직이며, Pod이 생성될 때 빈 디렉토리로 생성된다. 이후 pod내부에서 실행중인 컨테이너에서는 자유롭게 emptyDirs에 파일을 쓰고 읽을 수 있다. pod의 lifecycle과 묶여 있기 때문에 pod이 종료되면 같이 삭제된다.

emptyDirs는 주로 실행중인 컨테이너에서 파일을 공유할 때 사용된다. 
2개의 container로 구성된 pod을 만들어보자.
```
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  containers:
  - image: nginx:alpine
    name: nginx
    volumeMounts:				-- html이란 volume을 readonly 속성으로 /var/html에 mount한다.
    - name: html
      mountPath: /var/html
      readOnly: true
    ports:
    - containerPort: 80
      protocol: TCP
  - image: testImage
    name: testImage
    volumeMounts:				-- html이란 volume을 컨테이너의 /var/htmldoc에 mount한다.
    - name: html
      mountPath: /var/htmldoc
  volumes:
  - name: html
    emptyDir: {}
```

각 컨테이너에서는 mountPath로 지정된 경로를 이용하면 공유된 volume을 사용 가능하다.

##### gitRepo
emptyDirs + git checkout이라고 생각하면 편리하다.
컨테이너가 생성되기 전 gitRepository를 checkout하여 mount한다.
대신 초기에 volume이 생성된 후에는 sync가 되지 않는다.

```
apiVersion: v1
kind: pod
metadata:
  name: gitRepo
spec:
  containers:
  - image: ningx:alpine
    name: web
    volumeMounts:
    - name: html
      mountPath: /usr/html
      readOnly: true
    ports:
    - containerPort: 80
      protocol: TCP
  volumes:
  - name: html
    gitRepo:
      repository: ...
      revison: main
      directory: .
```


##### hostPath
대부분의 pod은 어느 node에 배치되어 서비스되는것에 관심이 없다. 그리고 일반적으로는 node의 파일 시스템에도 접근하지 않는것이 좋다. 하지만 daemonset과 같은 시스템 레벨의 pod에서는 접근해야할 일이 생긴다. 이럴때 사용하는게 hostPath이다.

hostPath는 node의 특정 파일, 디렉토리에 바로 mount를 하여 사용하는 방법이다. 동일한 Node의 container들이 동일한 path에 mount한다면 volume을 공유해 사용하게 된다. 이 hostPath는 pod이 종료되도 삭제되지 않고 남아있는다.

### Persistence Storage
pod에서 디스크에 데이터를 유지해야하고, 다른 노드에 pod이 생성되도 이 데이터를 유지해야한다면 hostpath를 써도 불가능하다. 이럴때는 cloud 환경에서 제공하는 전용 storage를 사용하면 pod의 lifecycle과는 관계없이 재 사용 및 데이터 유지가 가능하다.

```
apiVersion: v1
kind: Pod
metadata:
  name: mongo
spec:
  volumes:
  - name: mongodb
    awsElasticBlockStore:
      volumeId: test		-- EBS volume의 ID
      fsType: ext4		-- file system의 유형
  containers:
  ...
```
