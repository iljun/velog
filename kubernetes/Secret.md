# Secret
Application에 전달되어야하는 정보 중 외부에 노출이되면 안되는 정보가 존재한다.
password, 인증서 등등이 대표적인 값이다.
이러한 값은 외부로 노출되지 않아야한다.

secret은 configmap과 매우 유사하지만 
- memory에만 올라가기 때문에 container 종료 후 완벽하게 삭제된다.
- 물리 저장소 즉 etcd에 저장되지 않아 저장소가 노출되어도 노출되지 않는다.

이 secret은 master node에 저장되며, master node는 외부로 노출되지 않도록 해야한다.

secret의 항목은 base64로 인코딩 된 문자열로 표시된다. base64를 사용하는 이유는 간단하게 secret의 값이 언제나 문자열이 아닐수 있기 때문이다. binary 형태의 값도 secret을 이용해 담을 수 있다.

pod에서 secret을 사용하는 방법은 configmap과 대부분 동일하다. 환경변수 또는 volume에 mount하여 사용할 수 있다.

이 secret은 pod에서 필요한 정보 뿐만 아니라 kubernetes 자체에서도 사용한다.
가장 쉬운 예시로는 private repository에서 docker image를 가져올때 당연히 인증이 필요하다.
이 인증값을 secret으로 저장하고, serviceaccount에서 사용하여 private repository에서 image를 가져올 수 있게 한다. private repository에 접근하기 위해 secret을 사용하려면 serviceaccount나 pod, deployment에서 imagePullSecret을 이용하는 방법이 있다.

이 secret에서 중요한점은 master node의 장애가 발생했고, 이후에 복구가 되었다고해도 secert은 복구가 되지 않는다. 
