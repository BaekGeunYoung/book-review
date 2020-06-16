`todo-web.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: todoweb
  labels:
    app: todoweb
spec:
  selector:
    app: todoweb
  ports:
    - name: http
      port: 80
  type: NodePort
  
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todoweb
  labels:
    name: todoweb
spec:
  replicas: 2
  selector:
    matchLabels:
      app: todoweb
  template:
    metadata:
      labels:
        app: todoweb
    spec:
      volumes: # 1
      - name: assets
        emptyDir: {}
      containers:
      - name: nginx
        image: gihyodocker/nginx-nuxt:latest
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
          value: "localhost:3000"
        volumeMounts: # 2
        - mountPath: /var/www/_nuxt
          name: assets
      - name: web
        image: gihyodocker/todoweb:latest
        imagePullPolicy: Always
        lifecycle: # 3
          postStart:
            exec:
              command:
              - cp
              - -R
              - /todoweb/.nuxt/dist
              - /
        ports:
        - containerPort: 3000
        env:
        - name: TODO_API_URL
          value: http://todoapi
        volumeMounts: # 4
        - mountPath: /dist
          name: assets
```

#### 주의해서 볼 부분
**#1** : assets라는 이름의 볼륨 생성. emptyDir로 설정해 파드 단위로 할당되는 가상 볼륨 생성.

**#2** & **#4** : 1에서 만든 볼륨을 각각의 컨테이너의 어느 위치에 마운트시킬지 설정

**#3** : 초기에 이 볼륨은 빈 디렉터리이므로, 컨테이너 시작 직후에 값을 넣어줄 필요가 있음. lifecycle 이벤트를
이용해 이 작업을 실행