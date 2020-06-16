## MySQL 서비스 구축
mysql의 설정파일인 mysqld.conf를 수정해 마스터-슬레이브 db를 구축한다. 스택 내의 각 서비스들이 중복이 없도록 이 파일안에 server-id를 설정해야 한다.
`add-server-id.sh`를 통해 이 작업을 수행한다.
```
#!/bin/bash -e
OCTETS=(`hostname -i | tr -s '.' ' '`)

MYSQL_SERVER_ID=`expr ${OCTETS[2]} \* 256 + ${OCTETS[3]}`
echo "server-id=$MYSQL_SERVER_ID" >> /etc/mysql/mysql.conf.d/mysqld.conf
```

그 다음 master인지 slave인지의 여부를 환경변수로 받아 경우에 맞게 db 설정을 해주고 서버를 시작하는 prepare.sh파일을 작성한다. (내용 생략)

### Dockerfile
```
FROM mysql:5.7

# (1) 패키지 업데이트 및 wget 설치
RUN apt-get update
RUN apt-get install -y wget

# (2) entrykit 설치
RUN wget https://github.com/progrium/entrykit/releases/download/v0.4.0/entrykit_0.4.0_linux_x86_64.tgz
RUN tar -xvzf entrykit_0.4.0_linux_x86_64.tgz
RUN rm entrykit_0.4.0_linux_x86_64.tgz
RUN mv entrykit /usr/local/bin/
RUN entrykit --symlink

# (3) 스크립트 및 각종 설정 파일 복사
COPY add-server-id.sh /usr/local/bin/
COPY etc/mysql/mysql.conf.d/mysqld.cnf /etc/mysql/mysql.conf.d/
COPY etc/mysql/conf.d/mysql.cnf /etc/mysql/conf.d/
COPY prepare.sh /docker-entrypoint-initdb.d
COPY init-data.sh /usr/local/bin/
COPY sql /sql

# (4) 스크립트, mysqld 실행
ENTRYPOINT [ \
  "prehook", \
    "add-server-id.sh", \
    "--", \
  "docker-entrypoint.sh" \
]

CMD ["mysqld"]

```
** entrykit을 통해 mysql 서비스 시작 이전에 해야할 일(add-server-id.sh)을 실행시킬 수 있다.

### stack 정의 파일
`todo-mysql.yml`
```
version: "3"

services:
    master:
        image: registry:5000/ch04/tododb:latest
        deploy:
            replicas: 1
            placement:
                constraints: [node.role != manager]
        environment:
            MYSQL_ROOT_PASSWORD: gihyo
            MYSQL_DATABASE: tododb
            MYSQL_USER: gihyo
            MYSQL_PASSWORD: gihyo
            MYSQL_MASTER: "true"
        networks:
            - todoapp

    slave:
        image: registry:5000/ch04/tododb:latest
        deploy:
            replicas: 2
            placement:
                constraints: [node.role != manager]
        depends_on:
            - master
        environment:
            MYSQL_MASTER_HOST: master
            MYSQL_ROOT_PASSWORD: gihyo
            MYSQL_DATABASE: tododb
            MYSQL_USER: gihyo
            MYSQL_PASSWORD: gihyo
            MYSQL_REPL_USER: repl
            MYSQL_REPL_PASSWORD: gihyo
        networks:
            - todoapp

networks:
    todoapp:
        external: true

```
04-01에서 정의했듯이 마스터는 레플리카 수를 1개만, 슬레이브는 2개로 설정하여 스택정의 파일을 작성한 후, 아래 명령어로 스택을 스웜 클러스터에 배포한다.

```
docker container exec -it manager \
docker stack deploy -c /stack/todo-mysql.yml todo_mysql
```

아래와 같이 배포 현황을 확인할 수 있다.
```
docker container exec -it manager \
docker service ls

ID                  NAME                MODE                REPLICAS            IMAGE                               PORTS
nd3qtgaf6yr1        todo_mysql_master   replicated          1/1                 registry:5000/ch04/tododb:latest
d3vkcoroi2nl        todo_mysql_slave    replicated          2/2                 registry:5000/ch04/tododb:latest

```
