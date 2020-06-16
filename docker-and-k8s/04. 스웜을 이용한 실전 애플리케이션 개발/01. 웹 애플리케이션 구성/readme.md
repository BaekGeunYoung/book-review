지금까지 배운 것을 토대로 웹 애플리케이션 제작 실습을 해본다.
### 애플리케이션 요구 조건
- 주제 : TODO 앱
- TODO를 등록, 수정, 삭제할 수 있다.
- 등록된 TODO의 목록을 출력할 수 있다.
- 브라우저에서 사용할 수 있는 웹 애플리케이션이다.
- JSON API 엔드포인트를 제공한다.

### 아키텍처

![architecture](https://github.com/BaekGeunYoung/book-docker-and-k8s/blob/master/images/docker_swarm_architecture.jpg)

### 구성
#### MySQL
마스터-슬레이브 구조로 구성. insert, update, delete 쿼리는 master db에서 처리하고, select 쿼리는 slave db가 처리한다. 마스터 컨테이너는 레플리카를 한 개만 둔다.

#### API & Frontend & Nginx
흔히 볼 수 있는 평범한 구성. 모두 레플리카 수를 우선 3개로 한다.

#### overlay 네트워크
3개의 stack이 서로 소통해야 하므로, 모두 하나의 네트워크를 공유하도록 한다.
