### 데이터 볼륨
컨테이너에서 내에서 관리되는 파일들은 컨테이너가 파기될 경우 함께 사라진다. 새로운 버전의 컨테이너가 이전 버전의 컨테이너의 상태를 그대로 이어받아
작동하도록 하기란 쉽지 않다. 그래서 도커는 컨테이너 내의 데이터를 호스트 운영 체제의 퍼시스턴트 데이터로 관리할 수 있는 데이터 볼륨 기능을 제공한다.

docker container run 명령 시 -v 옵션으로 볼륨을 설정하거나 docker-compose.yml 파일에서 volumes 필드를 설정하면 호스트와 컨테이너 간의 파일 공유가 가능하도록 설정할 수 있다.
하지만 이 방법은 컨테이너가 호스트의 특정 경로에 의존되도록 만들기 때문에 이식성 측면에서 개선이 필요한 기능이라는 것을 알아두어야 한다.

### 데이터 볼륨 컨테이너
말했듯이 위의 방식은 컨테이너가 호스트에 의존성을 갖기 때문에 이식성 측면에서 좋지 않다. 이에 대한 대안으로 데이터 볼륨만을 위한 컨테이너를 따로 두면 호스트 머신의 영향력을 최소화할 수 있다.
퍼시스턴트 데이터의 관리가 필요한 컨테이너는 데이터 볼륨 컨테이너의 존재로 인해 호스트에 대해서는 아무것도 몰라도 되고 조금더 loosely coupled된 구조로 컨테이너를 관리할 수 있다.

### 예시 : 볼륨 컨테이너에 MySQL 데이터 저장하기
#### 볼륨 컨테이너 이미지 생성
`dockerfile`
```
FROM busybox
VOLUME /var/lib/mysql
CMD ["bin/true"]
```
** 볼륨 컨테이너는 데이터 저장만을 위한 컨테이너이므로 busybox와 같은 가벼운 베이스 이미지를 주로 사용한다.

#### mysql 컨테이너 실행
```
$ docker container run -d --rm --name mysql
  -e "MYSQL_ALLOW_EMPTY_PASSWORD=yes"
  -e "MYSQL_DATABASE=volume_test"
  -e "MYSQL_USER=example"
  -e "MYSQL_PASSWORD=example"
  --volumes-from mysql-data
  mysql:5.7
```
volumes-from 옵션으로 볼륨 컨테이너를 mysql 컨테이너에 마운트한다. 이렇게 하면 mysql 컨테이너 내의 /var/lib/mysql에 저장되어야할 데이터가
볼륨 컨테이너에 저장된다. 이 후 mysql 컨테이너를 파기하고 새로운 mysql 컨테이너를 실행시켜도 데이터는 영구적으로 보존되어있다.
