### 디플로이먼트
레플리카세트보다 상위에 해당하는 리소스로 디플로이먼트가 있다. 디플로이먼트는 애플리케이션 배포의 기본 단위가 되는 리소스이다.
레플리카세트는 똑같은 파드의 레플리케이션 개수를 관리 및 제어하고, 디플로이먼트는 레플리카세트를 관리하고 다루기 위한 리소스이다.

### 매니페스트 파일
디플로이먼트의 매니페스트 파일은 레플리카세트의 것과 거의 똑같다. 디플로이먼트가 하는 일은 레플리카세트의 리비전 관리 정도이다.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo
  labels:
    app: echo
spec:
  replicas: 3 # 레플리카 수
  selector:
    matchLables:
      app: echo
  template: # 해당 레플리카세트가 관리할 파드에 대한 정의
    metadata:
      name: simple-echo
    spec:
      containers:
      - name: nginx
        image: gihyodocker/nginx:latest
        env:
        - name: BACKEND_HOST
          value: localhost:8080
        ports:
        - containerPort: 80
      - name: echo
        image: gihyodocker/echo:latest
        ports:
        - containerPort: 8080
```

이 매니페스트 파일을 kubectl apply를 통해 적용하면 디플로이먼트는 물론 레플리카세트와 파드까지 한 번에 생성된다. 레플리카세트의 내용에 변화가 있을 경우 
리비전 번호가 올라간 채로 수정된 레플리카세트가 새로 생성되고 기존 레플리카세트와 교체된다.

### 롤백
디플로이먼트는 롤백 기능 덕분에 최신 디플로이먼트에 문제가 있을 경우 곧바로 이전 버전으로 돌아갈 수 있으며, 애플리케이션의 이전 버전의 동작을
확인하려는 경우에도 활용할 수 있다. 다음 명령어로 롤백을 수행할 수 있다.
```
$ kubectl rollout undo deployment echo
deployment "echo" rolled back
```
