지금까지는 단일 호스트에 여러 컨테이너를 배치하는 방법에 대해 알아보았다.
하지만 규모가 큰 애플리케이션을 실용적으로 운영하기 위해서는 여러 호스트에 컨테이너를 알맞게 배치할 필요가 있다.
이러한 작업을 컨테이너 오케스트레이션이라고 하고, 쿠버네티스 이전에 도커 스웜(swarm)이라는 오케스트레이션 툴을 학습해본다.

### 관련 용어

- 컴포즈

주로 단일 호스트에서 여러 컨테이너로 구성된 애플리케이션을 관리함

- 스웜

멀티 호스트에서 클러스터를 구축하고 관리함

- 서비스

스웜에서 클러스터 안의 서비스(컨테이너 하나 이상의 집합)을 관리함

- 스택

스웜에서 여러 개의서비스를 합한 전체 애플리케이션을 관리함

### manager & worker
도커 스웜에서 취급하는 호스트의 종류는 크게 두가지이다.
- manager

스웜 클러스터를 구축하고 관리하는 호스트
- worker

실제 작업을 수행하는 컨테이너, 스웜의 측면에서는 서비스가 돌아가는 호스트

### manager 호스트 만들기
원하는 호스트에 접속해서 docker swarm init 명령어를 입력하면 해당 호스트는 manager로 마킹되고 스웜 모드가 활성화됨. 이 명령어를 입력하면 생성되는
join 토큰으로 여러 대의 worker 호스트를 등록하여 클러스터를 구축할 수 있음.

### service
스웜에서 관리하는 작업의 단위(?). 하나의 서비스는 하나의 이미지를 갖고 있으며, 레플리카(컨테이너 갯수)를 제어함으로써 스웜 클러스터의 노드를 분산 배치한다.
서비스를 생성하는 것은 단일 호스트에서의 container run과 비슷한 느낌. docker service scale 명령으로 레플리카 수를 조절할 수 있다.

### stack
하나 이상의 서비스를 그룹으로 묶은 단위로 애플리케이션 전체 구성을 의미한다.(ex) nginx 프론트엔드 + api 백엔드 서버로 구성된 웹애플리케이션)
스택으로 배포된 서비스 그룹은 overlay 네트워크(서로 다른 호스트에 속해 있는 컨테이너가 서로 소통하기 위한 네트워크)에 속한다. 하나의 스택으로 묶일
서비스들은 같은 overlay 네트워크를 공유해야한다. yml 형식을 가지는 스택 정의 파일을 이용해 스택을 스웜에 배포할 수 있다.
ex)
```
version: "3"
services:
  nginx:
    image: nginx-proxy:latest
    deploy:
      replicas: 3
      placement:
        constraints: [node.role != manager]
    environment:
      BACKENT_HOST: http://localhost:8080
    depends_on:
      - api
    #### networks:
      - ch03
  api:
    image: example/echo:latest
    deploy:
      replicas: 3
      placement:
        constraints: [node.role != manager]
    networks:
      - ch03
      
networks:
  ch03:
    external: true\
```

### 스웜 클러스터 외부 (호스트)에서 클러스터 내 서비스에 접근하기

스웜 클러스터에서의 서비스는 여러 컨테이너가 여러 노드에 분산 배치 되어있기 때문에 클러스터 외부에서 서비스에 접근하기 위해서는 별도의 프록시 서버가 필요하다. HAProxy를 이용해 프록시 서버를 만들어보자. 클러스터 외부에서 내부로 접근할 수 있도록 다리 역할을 하는 것을 ingress라고 부른다.
ingress라는 이름의 스택을 만들되, 클러스터 외부와 연결해주려고 하는 스택과 같은 overlay 네트워크를 공유하도록 스택 정의 파일을 작성한다.
`ingress.yml`
```
version: "3"
services:
  haproxy:
    image: dockercloud/haproxy
    networks:
      - ch03
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager
    ports:
      - 80:80
      - 1936:1936 # for stats page
      
networks:
  ch03:
    external: true
```
같은 overlay 네트워크를 공유하고 있기 때문에 manager 호스트에서 80번 포트로 요청이 들어오면 haproxy는 ch03 네트워크 내 컨테이너 중 80번 포트에 바인딩 된 컨테이너를 외부 요청과 연결해줄 수 있다. 또한 이 이미지로 만든 컨테이너는 ingress 역할 외에도 로드 밸런싱 기능을 제공한다.(짱짱!)

## docker swarm 요약
- 서비스는 레플리카 수를 조절해 컨테이너를 쉽게 복제할 수 있다. 그리고 여러 노드에 레플리카를 배치할 수 있기 때문에 스케일 아웃에 유리하다.
- 서비스로 관리되는 레플리카는 서비스명으로 name resolution 되므로 서비스에 대한 트래픽이 각 레플리카로 분산된다.
- 스웜 클러스터 외부에서 스웜에 배포된 서비스를 이용하려면 서비스에 트래픽을 분산시키기 위한 프록시를 갖춰야 한다.
- 스택은 하나 이상의 서비스를 그룹으로 묶을 수 있으며, 여러 서비스로 구성된 애플리케이션을 배포할 때 유리하다.

## 실제 배포 환경에서의 스웜 클러스터 구성 관리
AWS나 GCP의 경우 오토스케일링 기능을 제공하므로, 도커가 설치된 호스트를 쉽게 늘리고 줄일 수 있다. 하지만 그러한 호스트들이 스웜 클러스터에 추가되지 않는다면 아무런 의미가 없으므로, 서버 인스턴스가 처음 시작될 때 docker swarm join 명령으로 해당 호스트를 클러스터에 추가할 필요가 있다. AWS 혹은 GCP에서는 서버를 처음 시작할 때 실행할 스크립트를 지정할 수 있으므로 이 스크립트에서 join 명령을 내려주면 호스트를 클러스터에 자동으로 추가할 수 있을 것이다.
