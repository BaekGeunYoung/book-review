### 레플리카세트
파드를 정의한 매니페스트 파일로는 파드를 하나밖에 생성할 수 없다. 그러나 어느 정도 규모가 되는 애플리케이션을 구축하려면 같은 파드를 여러 개 실행해
가용성을 확보해야 하는 경우가 생긴다. 이런 경우에 사용하는 것이 레플리카세트이다. 레플리카세트는 똑같은 정의를 갖는 파드를 여러개 생성하고 고나리하기 위한 리소스다.
파드의 정의 자체가 레플리카 세트를 정의한 매니페스트 파일에 포함되므로 파드를 위한 매니페스트 파일을 따로 둘 필요도 없다.

예시)
```
apiVersion: apps/v1
kind: ReplicaSet
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

레플리카세트를 통해 파드의 개수를 늘리고 줄일 수 있음. 삭제된 파드는 복원할 수 없기 때문에 stateless한 파드를 사용하기에 유리함. stateful한 파드를 관리하기 위한 리소스로
스테잍풀세트가 있음.
