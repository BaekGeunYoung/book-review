모든 도커 이미지에는 기반 이미지가 존재하는데, 이 기반 이미지를 가능한 가볍게 하면 우리가 만들 이미지도 그만큼 가벼워질 것이다.

### 가벼운 기반 이미지

#### scratch

scratch는 빈 도커 이미지이다. 모든 도커 이미지를 따라가 보면 결국 이 scratch 이미지에 이르게 된다. 이 이미지는 모든 도커 이미지의 조상 격이라고 할 수 있다.
빈 이미지이므로 크기가 거의 0에 가깝지만, 셸이 없으므로 디버깅하기가 불편하고 패키지 관리자가 없기 때문에 의존성 설치를 위해 많은 수고를 감내해야 한다.

#### BusyBox

busybox는 임베디드 시스템에서 많이 사용하는 리눅스 배포판으로, 크기가 매우 작다는 것이 특징이다. busybox는 리눅스 기반의 유틸리티를 갖추고 있음에도 이미지의 크기가 1MB남짓밖에 되지 않는다.
scratch와는 다르게 셸을 포함하고 있기 때문에 컨테이너 안에서 디버깅 작업이 가능하다. 따라서 데이터 볼륨 컨테이너 등으로 활용될 여지가 있다. 하지만 scratch와 마찬가지로
패키지 관리자가 없으므로 그에 따른 불편함이 있을 수 있다.

#### alpine linux

알파인 리눅스는 bubybox를 기반으로 만든 리눅스 배포판으로, 많은 도커 이미지의 기반 이미지로 쓰이는 대표적인 리눅스 이미지이다. 알파인 리눅스 이미지의 장점은
위 두 이미지와는 다르게 apk라는 패키지 관리자를 가지고 있다는 것이고, 그럼에도 불구하고 이미지의 크기가 4MB로 다른 리눅스 이미지에 비하면 압도적으로 가벼운 크기이다.
패키지 관리자를 갖고 있으므로 API 서버나 웹 애플리케이션, Nginx 등의 미들웨어까지 다양한 상황에서 활용할 수 있다.

### apk 사용법

#### apk update
로컬에 캐싱된 apk 패키지 인덱스를 업데이트하는 명령이다. 패키지 검색 및 설치는 이 로컬에 캐싱된 인덱스에 든 정보를 이용한다.

#### apk search
현재 사용할 수 있는 패키지를 검색하는 명령이다.

#### apk add
패키지를 설치하는 명령이다. 설치하려는 패키지 이름과 함께 사용하면 된다.
** --no-cache 옵션을 사용하면 /var/cache/apk 디렉터리에 저장된 apk 인덱스 대신 새로 받아온 인덱스 정보를 이용해 패키지를 설치한다. 또한 /var/cache/apk 디렉토리에
캐시를 저장하지 않으므로, 가벼운 도커 이미지를 만들기 위한 dockerfile에서 패키지를 설치할 때는 --no-cache 옵션을 자주 사용한다. **

#### apk del
설치된 패키지를 제거하는 명령이다. apk add --virtual 명령과 함께 사용하면 불필요한 패키지를 한 번에 지울 수 있다.

#### 알파인 리눅스 기반 도커 이미지 만들기
이 책에서 다룬 todoapi의 dockerfile을 알파인 리눅스 기반으로 작성하면 다음과 같다.
```dockerfile
FROM alpine:3.7

WORKDIR /
ENV GOPATH /go

# 1. 빌드 시에만 필요한 라이브러리 및 도구 설치 (--virtual 옵션 붙여서)
RUN apk add --no-cache --virtual=build-deps go git gcc g++

# 2. 실행 시에 필요한 라이브러리 및 도구 설치
RUN apk add --no-cache ca-certificates

# 3. todoapi를 빌드해 실행 파일을 만듬
COPY . /go/src/github.com/gihyodcocker/todoapi
RUN go get github.com/go-sql-driver/mysql
RUN go get gopkg.in/gorp.v1
RUN cd /go/src/github.com/gihyodocker/todoapi && go build -o bin/todoapi cmd/main.go
RUN cd /go/src/github.com/gihyodocker/todoapi && cp bin/todoapi /usr/local/bin/

# 4. 빌드 시에만 필요한 라이브러리 및 도구 제거
RUN api del --no-cache build-deps

CMD ["todoapi"]
```
