### 지금까지 살펴본 리소스
- 파드
- 레플리카세트
- 디플로이먼트
- 서비스
- 인그레스
- 스테이트풀세트
- 스토리지클래스
- 컨시스턴트볼륨
- 컨시스턴트볼륨클레임

지금까지 살펴본 리소스는 데몬으로 동장하는 서버 애플리케이션을 구축할 때 주로 사용되는 리소스들이다. 쿠버네티스는 데몬으로 동작하는 서버 애플리케이션 외에도
배치 서버 등 다양한 형태의 애플리케이션을 구축할 수 있다. 그러한 경우에 사용할 수 있는 리소스들을 살펴보자.

### 잡
하나 이상의 파드를 생성해 지정된 수의 파드가 정상 종료될 때까지 이를 관리하는 리소스. 잡이 생성한 파드는 정상 종료된 후에도 삭제되지 않고 그대로 남아 있기 때문에
작업이 종료된 후에 파드의 로그나 실행 결과를 분석할 수 있다. 그러므로 데몬형 애플리케이션보다는 배치 작업 위즈의 애플리케이션에 적합하다.

#### 예시 매니페스트 파일
`simple-job.yaml`
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pingpong
  labels:
    app: pingpong
spec:
  parallelism: 3
  template:
    metadata:
      labels:
        app: pingpong
    spec:
      containers:
      (생략(파드 정의와 같음))
```
spec.parallelism 속성을 정의해 여러 개의 파드가 병렬 실행되도록 할 수 있다.

### 크론잡
잡 리소스는 파드가 단 한 번만 실행되는 데 반해, 크론잡 리소스는 스케줄을 지정해 정기적으로 파드를 실행할 수 있다.
이 리소스 덕분에 Cron을 이용해 이벤트를 일으키는 애플리케이션을 따로 만들지 않아도 된다. 컨테이너 친화적인 특성을 유지하면서 스케줄에 따른 작업을
수행할 수 있다는 점이 가장 큰 장점이다. 또한 크론잡 리소스는 일반적인 Cron 작업과 달리 그 작업의 내용이 코드로 관리된다는 장점도 있다.

#### 예시 매니페스트 파일
`simple-conjob.yaml`
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: pingpong
spec:
  schedul: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: pingpong
        spec:
          containers:
      (생략(파드 정의와 같음))
```
spec.schedule 속성에 cron과 같은 포맷으로 파드를 실행할 스케줄을 정의할 수 있다.

### 시크릿
시크릿은 인증서나 비밀키, 패스워드 같은 기밀정보를 Base64로 인코딩해서 저장할 수 있도록 하는 리소스이다.

#### 예시 매니페스트 파일
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: nginx-secret
type: Opaque
data:
  .htpasswd: [base64 인코딩된 문자열]
```

이렇게 클러스터 내에 secret을 하나 생성하고, 이를 볼륨으로 마운트함으로써 다른 리소스에서 참조할 수 있다.
```yaml
...
volumeMounts:
  - mountPath: /etc/nginx/secret
    name: nginx-secret
env:
  - name: BASIC_AUTH_FILE
    value: "/etc/nginx/secret/.htpasswd"
...
```
시크릿이 볼륨의 형태로 목표 컨테이너에 마운트될 때는 데이터가 디코딩되어 저장된다.
