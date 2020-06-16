### 배포 자동화와 무중단 배포
쿠버네티스와 같은 컨테이너 오케스트레이션 도구를 이용해 배포를 하면서 중요해진 것은 더욱 더 정교한 운영을 위해 어떻게 배포 작업을 자동화하고 서비스 무중단을
유지해야 할지 고민하는 것이다. 쿠버네티스에서 롤링 업데이트와 블루-그린 배포를 하는 방법에 대해 알아보자.

### 롤링 업데이트
디플로이먼트에서 파드를 교체하는 전략을 .specs.strategy.type 속성에서 정의할 수 있다. 이 속성의 기본값은 RollingUpdate이다. 이 속성을 가진 디플로이먼트는 
자신이 관리하는 파드의 내용에 변화가 있을 경우 알아서 롤링 업데이트 방식으로 파드를 교체한다.

#### 롤링 업데이트 동작 제어하기
디플로이먼트의 매니페스트 파일에서 stragegy.rollingUpdate 아래의 maxUnavailable와 maxSurge 속성을 통해 제어할 수 있다.
```yaml
spec:
  strategy:
    rollingUpdate:
      maxUnavailable: 3
      maxSurge: 4
```
maxUnavailalbe은 롤링 업데이트 중 동시에 삭제할 수 있는 파드의 최대 개수이다. 이를 늘리면 롤링 업데이트에 걸리는 시간이 짧아지겠지만, 업데이트가 일어나는 동안
남아있는 파드에 몰리는 트래픽의 수가 늘어날 것이다.

maxSurge는 롤링 업데이트 중 동시에 생성하는 파드의 개수이다. 기본값은 replicas의 25퍼센트이다. 이 값을 늘리면 필요한 파드를 빠르게 생성하기 때문에 파드 교체시간이 단축되지만,
순간적으로 필요한 시스템 자원이 급증하는 부작용이 있다.

### 실행 중인 컨테이너에 대한 헬스 체크 설정
어플리케이션에 따라 컨테이너가 모두 시작된 후에도 요청을 처리할 수 있는 상태가 될 때까지 좀 더 시간이 걸리는 경우가 있다. 이런 경우에는 파드가 running
상태여도 애플리케이션이 제대로 된 응답을 하지 못할 수 있다. 이러한 문제를 해결하기 위해 쿠버네티스는 두 가지 컨테이너 헬스 체크 기능을 제공한다.

#### livenessProbe
```yaml
spec:
  template:
    spec:
      containers:
      - name: ~~
        livenessProbe:
          exec:
            command:
            - cat
            - /example.txt
          initialDelaySeconds: 3
          periodSeonds: 5
```
livenessProbe는 애플리케이션 헬스 체크 기능으로, 애플리케이션이 의존하는 컨테이너 안의 파일이 존재하는지를 확인하는 용도로 사용한다. 위 예시는 컨테이너 내에
example.txt라는 파일이 없으면 헬스 체크 결과가 unhealthy가 되고, 파드를 재시작한다.

#### readinessProbe

```yaml
spec:
  template:
    spec:
      containers:
      - name: ~~
        readinessProbe:
          httpGet:
            path: /hc
            port: 8080
          initialDelaySeconds: 15
          timeoutSeconds: 3
```
readinessProbe는 컨테이너 외부에서 http 요청 같은 트래픽을 발생시켜 이를 처리할 수 있는 상태인지를 확인하는 기능이다. readinessProbe를 설정하면 정상상태가
아닌 컨테이너가 애플리케이션에 투입되는 것을 막을 수 있으므로 반드시 설정해 두는 것이 좋다.

위 두 헬스 체커가 설정되어 있는 컨테이너를 포함한 파드는 상태가 Running이 되어도 헬스 체커를 통과할 때까지 READY의 값이 올라가지 않는다.

### 블루-그린 배포
블루그린 배포란 기존 서버군과 별도로 새로운 버전이 배포된 서버군을 구성하고, 로드 밸런서 혹은 서비스 디스커버리 수준에서 참조 대상을 교체하는 방식으로 이뤄지는
배포를 말한다. 쿠버네티스에서는 서비스의 셀렉터 레이블을 변경해 참조할 디플로이먼트를 변경할 수 있다. 새로 배포한 버전에 문제가 있어도 셀렉터 레이블의 값을 다시
돌려놓음으로써 손쉽게 롤백이 가능하다. 또한 순간적으로 여러 개의 버전이 공존하게 되는 롤링 업데이트 방식의 문제를 해결할 수 있다.

![bluegreen](https://github.com/BaekGeunYoung/book-docker-and-k8s/blob/master/images/point2blue.png)
