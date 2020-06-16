### 로깅 운영
도커와 같은 컨테이너 환경에서 로그 파일을 다루는 정석적인 방법은 컨테이너 내에서 애플리케이션이 발생시키는 표준출력들을 호스트에서 파일에다 수집하는 것이다.
보통 도커에서는 fluentd를 로그를 수집하는 로깅 드라이버로 많이 사용하고, 이는 elasticsearch와 함께 사용해 로그 수집 및 검색 기능을 구축할 수 있다.

### fluentd와 elasticsearch를 이용한 로그 수집 및 검색 기능 구축
fluentd로 컨테이너에서 내뱉는 로그를 수집하고, elasticsearch에다 저장한 다음, kibana를 통해 시각화한다.

#### Elasticsearch와 Kibana 구축
`docker-compose.yaml`
```yaml
version: "3"
services:
    elasticsearch:
        image: elasticsearch:5.6-alpine
        ports:
            - "9200:9200"
        volumes:
        - "./jvm.options:/usr/share/elasticsearch/config/jvm.options"

    kibana:
        image: kibana:5.6
        ports:
        - "5601:5601"
        environment:
            ELASTICSEARCH_URL: "http://elasticsearch:9200"
        depends_on:
        - elasticsearch
```
elasticsearch 및 kibana는 docker hub의 표준 라이브러리에 있는 이미지를 사용하면 된다. 도커 컴포즈의 네임 레졸루션을 이용해 키바나에서 elasticsearch의 url을 추적할 수 있다.
이렇게 실행하면 해당 호스트의 5601번 포트에서 키바나 사이트가 뜨는 것을 확인할 수 있다.

![kibaba](https://github.com/BaekGeunYoung/book-docker-and-k8s/blob/master/images/kibana.PNG)

#### fluentd 구축
fluentd의 설정 파일을 fluent.conf라는 이름으로 작성하고, 이를 COPY 인스트럭션으로 컨테이너 안에다 복사해 준다.
`fluent.conf`
```html
<source>
    @type forward
    port 24224
    bind 0.0.0.0
</source>

<match *.**>
    @type copy
    <store>
        type elasticsearch
        host elasticsearch
        port 9200
        logstash_format true
        logstash_prefix docker
        logstash_dateformat %Y%m%d
        include_tag_key true
        type_name app
        tag_key @log_name
        flush_interval 5s
    </store>
    <store>
        type file
        path /fluentd/log/docker_app
    </store>
</match>
```

`Dockerfile`
```dockerfile
FROM fluent/fluentd:v0.12-debian

RUN gem install fluent-plugin-elasticsearch --no-rdoc --no-ri --version 1.9.2
COPY fluent.conf /fluentd/etc/fluent.conf
```

#### fluentd 로깅 드라이버로 컨테이너 로그 전송하기
fluentd 이미지도 빌드를 완료했으니, 이 이미지와 실행할 애플리케이션에 대한 내용을 docker-compose에 추가해준다.
`docker-compose.yaml`
```yaml
version: "3"
services:
    elasticsearch:
        image: elasticsearch:5.6-alpine
        ports:
            - "9200:9200"
        volumes:
        - "./jvm.options:/usr/share/elasticsearch/config/jvm.options"

    kibana:
        image: kibana:5.6
        ports:
        - "5601:5601"
        environment:
            ELASTICSEARCH_URL: "http://elasticsearch:9200"
        depends_on:
        - elasticsearch

    fluentd:
        image: ch08/fluentd-elasticsearch:latest
        ports:
            - "24224:24224"
            - "24220:24220"
        depends_on:
        - elasticsearch

    echo:
        image: gihyodocker/echo:latest
        ports:
        - "9000:8080" # 현재 8080번 포트가 사용중이어서 다른 포트에 바인딩함
        logging:
            driver: "fluentd"
            options:
                fluentd-address: "localhost:24224"
                tag: "docker.{{.Name}}"
        depends_on:
        - fluentd
```

도커 컴포즈를 통한 컨테이너 실행을 완료한 후, 몇 번 트래픽을 보내보고 키바나에 접속해보면 다음과 같이 로그를 확인할 수 있다.

![logs](https://github.com/BaekGeunYoung/book-docker-and-k8s/blob/master/images/logs.PNG)

### 구글 스택드라이버를 통한 로깅 관리
스택드라이버는 GCP나 AWS에서 로깅 및 모니터링에 사용되는 매니지드 서비스이다. 이 스택드라이버를 통해 EKS나 GKE 등의 매니지드 환경에서는 개발자가 로깅에 대해 직접 관리할 필요가 없고, 애플리케이션 컨테이너에서 로그를 json 포맷으로 출력하기만 하면 스택드라이버로 로그를 열람할 수 있다.
06장에서 GKE로 만든 todolist에 대한 스택드라이버 페이지는 다음과 같다.

![logviewer](https://github.com/BaekGeunYoung/book-docker-and-k8s/blob/master/images/logviewer.PNG)

스택드라이버에서는 자동완성을 통한 편리한 검색 기능을 제공하며, 이 역시 내부적으로는 fluentd로 구성되어있다.
