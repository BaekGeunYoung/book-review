api 서버를 구축하기 위한 매니페스트 파일을 다음과 같이 작성한다.
결합성이 높은 nginx와 API 서버를 하나의 파드로 묶어 배포한다.

`todo-api.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: todoapi
  labels:
    app: todoapi
spec:
  selector:
    app: todoapi
  ports:
    - name: http
      port: 80
      
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todoapi
  labels:
    name: todoapi
spec:
  replicas: 2
  selector:
    matchLabels:
      app: todoapi
  template:
    metadata:
      labels:
        app: todoapi
    spec:
      containers:
      - name: nginx
        image: gihyodocker/nginx:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 80
        env:
        - name: WORKER_PROCESSES
          value: "2"
        - name: WORKER_CONNECTIONS
          value: "1024"
        - name: LOG_STDOUT
          value: "true"
        - name: BACKEND_HOST
          value: "localhost:8080"
      - name: api
        image: gihyodocker/todoapi:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        env:
        - name: TODO_BIND
          value: ":8080"
        - name: TODO_MASTER_URL
          value: "gihyo:gihyo@tcp(mysql-master:3306)/tododb?parseTime=true"
        - name: TODO_SLAVE_URL
          value: "gihyo:gihyo@tcp(mysql-slave:3306)/tododb?parseTime=true"
```

nginx의 경우 api 서버의 호스트를 환경변수로 전달해야 하는데, 같은 파드 안에 api 서버가 존재하므로
localhost로 변수값을 전달해주면 된다.

api의 경우 db 연결을 위한 connection string이 필요하다. 마스터와 슬레이브 각각은 mysql-master, mysql-slave로
네임 레졸루션 되어있기 때문에 이 값을 이용해 connection string을 만들어 환경변수로 전달한다.

```
$ kubectl get pod -l app=todoapi
NAME                      READY   STATUS    RESTARTS   AGE
mysql-slave-0             1/1     Running   0          80m
mysql-slave-1             1/1     Running   0          80m
todoapi-cdffbff68-nfbtx   2/2     Running   0          69m
todoapi-cdffbff68-q6xzk   2/2     Running   0          69m
``` 
api의 경우 statefulset로 배포한 mysql과는 다르게 파드 이름 뒤에 무작위로 suffix가 붙는 것을 확인할 수 있다.