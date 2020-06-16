### docker compose
하나의 시스템은 여러 애플리케이션의 유기적인 연동 및 통신으로 이루어짐.
여러 개의 컨테이너를 쉽게 실행시키고, 컨테이너 간 의존 관계를 적절히 설정할 수 있는 방법으로 docker compose가 있음.

### docker-compose.yml
```
docker container run -d -p 9000:8000 example/echo:latest
```
위 명령을 docker-compose로 실행하려면 다음과 같은 yml 파일을 만들어 실행시키면 된다.
```
version: "3"
services:
  echo:
    image: example/echo:latest
    ports:
      - 9000:8000
```
docker compose up 명령으로 컨테이너를 실행할 수 있고, down 명령으로 정지시킬 수 있다.
image를 지정해주는 대신 로컬에 있는 dockerfile의 경로를 build 필드에 작성해줄 수도 있다.
