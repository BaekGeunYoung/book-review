젠킨스의 마스터 컨테이너와 슬레이브 컨테이너를 예시로 설명.

### docker-compose.yml
```
version: "3"
services:
  master:
    container_name: master
    image: jenkinsci/jenkins:2.142-slim
    ports:
      - 8080:8080
    volumes:
      - ./jenkins_home:/var/jenkins_home
    links:
      - slave01
      
  slave01:
    container_name: slave01
    image: jenkinsci/ssh-slave
    environment:
      - JENKINS_SLAVE_SSH_PUBKEY= ...
```
#### volumes
호스트와 컨테이너가 파일을 공유할 수 있도록 해주는 설정

#### links
master 컨테이너에 links 옵션을 줌으로써 이 컨테이너는 slave01이라는 이름으로 컨테이너를 찾아갈 수 있음.
원래라면 ssh로 접속하는 slave 컨테이너이므로 IP주소를 찾아 설정하는 등의 작업이 필요하지만, 이 옵션을 통해서 보다 쉽게 컨테이너 간 연결을 설정할 수 있음.

### but...
위 예시는 사실 마스터 컨테이너를 먼저 만들고 실행시킨 뒤 젠킨스 홈페이지에서 슬레이브 관련 설정을 해준 뒤 슬레이브 컨테이너를 실행해야 한다.
도커를 사용하는 우리의 목표는 수작업을 최대한 줄이는 것이므로 설정만으로 도커 구성 관리를 할 수 있도록 애플리케이션과 도커를 만드는 방법을 알아보아야 한다.
