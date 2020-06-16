### 파드(pod)
파드는 컨테이너가 모인 집합체의 단위로, 적어도 하나 이상의 컨테이너로 이루어짐.
쿠버네티스에서는 결합이 강한 컨테이너(예: nginx 컨테이너와 web UI 컨테이너)를 파드로 묶어 일괄 배포한다. 컨테이너가 하나인 경우에도 파드로 배포한다.
같은 파드를 여러 노드에 배치할 수도 있고 한 노드에 여러 개 배치할 수도 있다. 단, 한 파드를 여러 노드에 걸쳐 배포할 수는 없다.

### 매니페스트 파일
쿠버네티스에서 여러 가지 리소스를 정의하는 파일을 매니페스트 파일이라고 한다.
아래는 nginx 컨테이너와 echo 컨테이너로 구성되는 파드를 정의한 매니페스트 파일이다.
```
apiVersion: v1
kind: Pod
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
kind에서 정의하려는 쿠버네티스 리소스의 유형을 지정할 수 있음. 메타데이터는 말 그대로 리소스에 부여되는 메타데이터를 뜻함.
나머지는 문법은 기존 docker-compose의 yml 파일과 약간 차이가 날 수 있지만 의미는 전체적으로 상통함.

### 파드로 실행시킨 컨테이너 내부 접근하기

docker container exec 명령과 비슷하게 kubectl을 이용해 컨테이너 내부에 접근할 수 있다. 파드 안의 컨테이너가 여러 개인 경우에는 -c 옵션으로
컨테이너 이름을 지정해주면 된다.
```
$ kubectl exec -it simple-echo sh -c nginx
#
```

kubectl log 명령으로 파드 안에 있는 컨테이너의 표준 출력을 화면에 띄울 수 있다.
```
$ kubectl logs -f simple-echo -c echo
2020/01/18 15:30:30 start server
```

### 파드와 파드 안에 든 컨테이너의 주소
파드에는 각각 고유의 가상 IP 주소가 할당된다. 더불어 파드 안의 모든 컨테이너들은 이 파드에 할당된 가상 IP를 공유한다. 그래서 같은 파드 내의 컨테이너들은
단순히 localhost로 통신할 수 있고, 다른 파드의 컨테이너끼리도 파드의 가상 IP만 알면 통신이 가능하다. **파드는 사실 컨테이너를 담은 가상 머신이나 마찬가지다.**
