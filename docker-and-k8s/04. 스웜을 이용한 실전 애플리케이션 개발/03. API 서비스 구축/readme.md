## API 서버 서비스 구축
go를 이용해 간단한 CRUD operation을 수행할 수 있는 api 서버를 만들고, 이를 도커 스웜으로 배포한다.

### Dockerfile
```
FROM golang:1.13.6

WORKDIR /
ENV GOPATH /go

COPY . /go/src/github.com/gihyodocker/todoapi
RUN go get github.com/go-sql-driver/mysql
RUN go get gopkg.in/gorp.v1
RUN cd /go/src/github.com/gihyodocker/todoapi && go build -o bin/todoapi cmd/main.go
RUN cd /go/src/github.com/gihyodocker/todoapi && cp bin/todoapi /usr/local/bin/

CMD ["todoapi"]

```

### Stack 정의 파일
`todo-app.yml`
```
version: "3"

services:
    api:
        image: registry:5000/ch04/todoapi:latest
        deploy:
            replicas: 2
        environment:
            TODO_BIND: ":8080"
            TODO_MASTER_URL: "gihyo:gihyo@tcp(todo_mysql_master:3306)/tododb?parseTime=true"
            TODO_SLAVE_URL: "gihyo:gihyo@tcp(todo_mysql_slave:3306)/tododb?parseTime=true"
        networks:
            - todoapp

networks:
    todoapp:
        external: true

```

### stack 배포
(생략)
