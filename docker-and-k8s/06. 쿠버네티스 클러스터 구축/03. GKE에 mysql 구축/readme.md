### 퍼시스턴트볼륨과 퍼시스턴트볼륨클레임
퍼시스턴트 데이터를 다루는 컨테이너를 도커로 실행할 때는 데이터 볼륨을 이용했다.
표준 데이터 볼륨은 결국 호스트 머신에 위치하기 때문에, 이러한 방식은 컨테이너의 호스트에 대한 
의존성을 강화하는 부작용을 낳는다.

쿠버네티스의 경우 호스트에서 분리할 수 있는 외부 스토리지를 볼륨으로 사용하므로써 이러한 문제를 해결한다.
퍼시스턴트볼륨 혹은 퍼시스턴트볼륨클레임은 이 외부 스토리지의 확보를 위한 쿠버네티스 리소스이다.
퍼시스턴트볼륨은 스토리지 그 자체라고 볼 수 있으며, 반면 볼륨클레임은 추상화된 논리 리소스로, 퍼시스턴트볼륨과는 달리 
용량을 필요한 만큼 동적으로 확보할 수 있다.

### 스토리지클래스
스토리지클래스는 퍼시스턴트볼륨으로 확보하는 스토리지의 종류를 정의하는 리소스이다.
퍼시스턴트볼륨을 정의하는 매니페스트 파일에서는 스토리지클래스를 지정하는 storageClassName이라는 필드가 존재한다.
실습에서 사용할 스토리지클래스를 위한 매니페스트 파일을 작성하고, 클러스터에 반영한다.

`storage-class-ssd.yaml`
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
    name: ssd
    annotations:
        storageclass.kubernetes.io/is-default-class: "false"
    labels:
        kubernetes.io/cluster-service: "true"
provisioner: kubernetes.io/gce-pd
parameters:
    type: pd-ssd
```
parameters.type 필드에서 해당 스토리지클래스가 '표준' 타입인지 'SSD' 타입인지 설정할 수 있다.

### 스테이트풀세트(statefulSet)
디플로이먼트는 퍼시스턴트 데이터를 갖지 않는 stateless한 애플리케이션을 배포하는데 적절하다.
반면 stateful한 애플리케이션을 배포하는 데에는 statefulSet라는 리소스를 사용한다.
이 statefulset를 이용해서 배포할 mysql-master 서비스의 매니페스트 파일은 다음과 같다.
`mysql-master.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-master
  labels:
    app: mysql-master
spec:
  ports:
  - port: 3306
    name: mysql
  clusterIP: None
  selector:
    app: mysql-master
    
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-master
  labels:
    app: mysql-master
spec:
  serviceName: "mysql-master"
  selector:
    matchLabels:
      app: mysql-master
  replicas: 1
  template:
    metadata:
      labels:
        app: mysql-master
    spec:
      terminationGracePeriodSeconds: 60
      containers:
      - name: mysql
        image: gihyodocker/tododb:latest
        imagePullPolicy: Always
        args:
        - "--ignore-db-dir=lost+found"
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "gihyo"
        - name: MYSQL_DATABASE
          value: "tododb"
        - name: MYSQL_USER
          value: "gihyo"
        - name: MYSQL_PASSWORD
          value: "gihyo"
        - name: MYSQL_MASTER
          value: "true"
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: ssd
      resources:
        requests:
          storage: 4Gi
```

volumeClaimTemplates 필드를 정의해서 퍼시스턴트볼륨클레임을 파드마다 자동으로 생성하도록 할 수 있다.
이 덕분에 파드가 요구하는 퍼시스턴트볼륨클레임을 매번 만들지 않아도 된다.