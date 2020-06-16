### 빌드 컨테이너와 실행 컨테이너의 분리
도커 이미지를 빌드하는 과정에서 애플리케이션 빌드와 배포 과정이 모두 같은 컨테이너에서 이루어지는 경우가 많았다. 이 방식은 빌드 과정에만 필요한 라이브러리와
애플리케이션 실행에 불필요한 빌드 부산물을 완전히 제거할 수 없다는 한계가 있었다.(09-02장에서 배운 apk del 등의 방법을 쓴다고 하더라도) 멀티 스테이지 빌드를 통한
빌드 컨테이너와 실행 컨테이너의 분리는 이와 같은 문제점을 해결해 준다.

예시)
```dockerfile
FROM golang:1.9 AS build

WORKDIR /
COPY . /go/src/github.com/gihyodcocker/todoapi
RUN go get github.com/go-sql-driver/mysql
RUN go get gopkg.in/gorp.v1
RUN cd /go/src/github.com/gihyodocker/todoapi && go build -o bin/todoapi cmd/main.go

FROM alpine:3.7

COPY --from=build /go/src/github.com/gihyodocker/todoapi/bin/todoapi /usr/local/bin/

CMD ["todoapi"]
```

멀티 스테이지 빌드의 가장 큰 특징은 하나의 dockefile에 FROM 인스트럭션이 두 번 등장한다는 것이다. 위의 FROM에서는 AS 명령어로 빌드 컨테이너의 이름을 build로 지정해주었으므로,
실행 컨테이너에서 이 이름을 통해 빌드 컨테이너를 참조할 수 있다. 멀티 스테이지 빌드는 빌드 후에 실행 컨테이너만 남기고 빌드 컨테이너는 폐기시키기 때문에 디스크 용량을 효율적으로
관리할 수 있고 실행 컨테이너의 크기를 가볍게 유지할 수 있다.
