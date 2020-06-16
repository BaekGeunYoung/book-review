### 헬름(helm)
쿠버네티스를 운영하다보면 같은 애플리케이션을 여러 클러스터에 배포해야 하는 경우가 생긴다.
(개발용/서비스용을 나누거나 부하 테스트용 클러스터를 따로 두거나)
 
여러 클러스터를 배포할 각각의 환경에 맞는 설정값만 적절히 설정해둔 다음 이에 따라 배포하는 메커니즘이 필요했다.
이것이 바로 헬름(helm)이다.

`헬름은 쿠버네티스 '차트'를 관리하기 위한 도구이다. 차트는 사전 구성된 쿠버네티스 리소스의 패키지다.`

헬름, 차트, 매니페스트, 쿠버네티스 간 관계는 대략 아래와 같다.

- 헬름은 차트를 관리
- 차트를 통해 매니페스트 파일을 생성
- 매니페스트 파일을 기반으로 쿠버네티스 클러스터 구축

### 틸러(tiller)
**책에서는 틸러라는 개념을 소개하고 있지만, 헬름 3.0 버전부터는 틸러를 없애는 등 책의 내용과 달라진 점이 꽤 많아보였다.
그래서 책과 인터넷 자료를 같이 참고해가며 학습했다.**

### 커스텀 차트 생성하기
우선 차트를 만드는 명령어를 입력한다.
```
$ helm create echo
Creating echo
``` 
그러면 명령어를 입력한 디렉토리에 echo라는 디렉토리가 생성되고, 그 구조는 아래와 같다.
```
.
|-- Chart.yaml
|-- charts
|-- templates
|   |-- NOTES.txt
|   |-- _helpers.tpl
|   |-- deployment.yaml
|   |-- ingress.yaml
|   |-- service.yaml
|   |-- serviceaccount.yaml
|   `-- tests
|       `-- test-connection.yaml
`-- values.yaml
```
우리가 해야할 일을 정리해보면 다음과 같다.
1. templates 폴더안의 템플릿 파일들 작성하기
2. 기본 values.yaml 파일 작성하기
3. 레포지토리 생성 및 차트 등록하기
4. 차트를 다운받아 커스텀 values 파일을 적용해 클러스터 구축하기

### 템플릿 파일 작성하기
템플릿 파일은 우리가 만들고 싶어하는 매니페스트 파일의 모양새를 고려하여 작성하면 된다.
변수화할만한 부분들을 적절히 골라 values.yaml 파일에서 정의해주어야 한다.

### values.yaml 파일 작성하기
환경에 관계없이 공통으로 적용될 변수들은 values.yaml 파일에 작성한다.
#### 예시
```yaml
replicaCount: 1
nginx:
  image:
    repository: gihyodocker/nginx
    tag: latest
    pullPolicy: Always
  healthCheck: /
  backendHost: localhost:8080

echo:
  image:
    repository: gihyodocker/echo
    tag: latest
    pullPolicy: Always
  httpPort: 8080

```

### 레포지토리 생성 및 차트 등록하기
다른 사람들이 내가 만든 차트를 공유할 수 있도록 하려면 차트를 관리하는 원격 레포지토리가 필요하다.
깃헙, AWS S3, google cloud storage 등 다양한 선택지가 있다. 깃헙을 통해 실습을 진행했다.

![repo](https://github.com/BaekGeunYoung/book-docker-and-k8s/blob/master/images/helmrepo.PNG)

레포지토리에는 레포지토리 내에 존재하는 차트들의 정보를 담고 있는 index.yaml 파일과 각 차트들의 아카이브 파일이 담겨있어야 한다.
그리고 깃헙 페이지로 이 내용을 공개하도록 설정하면 로컬에서 이 레포지토리를 추가할 수 있고, 그 안의 차트들 또한 열람할 수 있다.

```
$ helm repo list
NAME            URL
stable          https://kubernetes-charts.storage.googleapis.com
my-stable       https://baekgeunyoung.github.io/helm-chart-practice/stable
```

```
$ helm search repo my-stable
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
my-stable/echo          0.1.0           1.16.0          A Helm chart for Kubernetes
my-stable/example       0.1.0           1.16.0          A Helm chart for Kubernetes
```
