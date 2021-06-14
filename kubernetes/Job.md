# Job
지속적으로 Task를 실행하는 Object가 아닌, 단일로 Task를 실행하고 완료하는 Resource도 존재한다.
이 Job은 컨테이너 내부의 실행 중인 프로세스가 성공적으로 완료되면 컨테이너를 다시 시작하지 않는 Pod을 생성하게 해준다.

즉 batch job 같은 기능에 어울리는 Resource이다.

Job은 실행중에 Node의 문제가 발생하면, ReplicaSet과 동일하게 다른 Node로 Pod을 배치하고, 컨테이너를 다시 실행할지 말지를 결정한다.

job은 kubernetes API 그룹중 batch/v1에 속한다.
```
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  template:
    metadata:
      labels:
        app: batch-job
    spec:
      restartPolicy: ONFailure	-- Job의 재시작 정책이다. Always는 지원하지 않으며, OnFailure나 Never로 지정해야한다.
      containers:
      - name: main
        image: luksa/batch-job
```

Job은 process를 정상적으로 종료했을 경우 Status가 Complete로 기록된다.
Job의 모든 상태를 보고 싶으면 `-a` Option을 넣어 조회하면 된다.

Pod이 완전하게 삭제되지 않고 남아있는 이유는 바로 해당 Pod의 Log를 검사할 수 있게 함이다.

Job은 하나의 Task가 아닌 두개 이상의 pod을 생성해 병렬, 순차적으로 process를 실행하도록 구성할 수 있다. 이 설정은 completions, parallelism 속성을 이용해 지정한다.

completions option은 순차적으로 몇개의 pod을 실행할지에 대한 설정값이며, parallelism은 몇개의 job을 병렬로 실행할지를 나타낸다.

이 Job이 종료가 되지 않는 상태라면 무제한으로 Job이 끝나지 않고 유지되게 된다.
이럴땐 activeDeadlineSeconds 속성을 이용해 실행시간을 제한할 수 있다.

# CronJob
Job을 일정 시간마다 실행하고 싶을때는 CronJob을 이용해 실행한다.
Job과는 개념이 비슷하며, Job + Cron의 개념이라고 생각하면 쉽다.

```
apiVerison: batch/v1beta1
kind: CronJob
metadata:
  name: batch-job-every-fifteen-minutes
spce:
  schedule: "0,15,30,45 * * * *" 	-- cron
  jobTemplate:
   spec:
     template:
       metadata:
         labels:
           app: periodic-batch-job
       spec:
         restartPolicy: OnFailure
         containers:
         - name: main
           image: luksa/batch-job
```
