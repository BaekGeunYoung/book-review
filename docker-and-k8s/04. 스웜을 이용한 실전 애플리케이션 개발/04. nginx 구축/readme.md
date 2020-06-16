## nginx 구축
nginx는 웹 UI 서비스 앞단에 1개, api 서버 앞단에 1개로 총 2개를 배치한다. nginx와 같은 웹 서버를 api의 앞단에 배치하는 이유는 접근 로그를 생성하기 편리하다는 점,
캐시 제어, 애플리케이션을 수정하지 않고도 임의로 라우팅 설정이 가능하다는 점 등을 들 수 있다.

### Dockerfile
```
FROM nginx:1.13

RUN apt-get update
RUN apt-get install -y wget
RUN wget https://github.com/progrium/entrykit/releases/download/v0.4.0/entrykit_0.4.0_linux_x86_64.tgz
RUN tar -xvzf entrykit_0.4.0_linux_x86_64.tgz
RUN rm entrykit_0.4.0_linux_x86_64.tgz
RUN mv entrykit /usr/local/bin/
RUN entrykit --symlink

RUN rm /etc/nginx/conf.d/*
COPY etc/nginx/nginx.conf.tmpl /etc/nginx/
COPY etc/nginx/conf.d/ /etc/nginx/conf.d/

ENTRYPOINT [ \
  "render", \
      "/etc/nginx/nginx.conf", \
      "--", \
  "render", \
      "/etc/nginx/conf.d/upstream.conf", \
      "--", \
  "render", \
      "/etc/nginx/conf.d/public.conf", \
      "--" \
]

CMD ["nginx", "-g", "daemon off;"]
```

### stack 정의 파일
04-03에서 작성했던 api 서버의 스택 정의 파일에 다음 내용을 추가한다.
```
	nginx:
		image: registry:5000/ch04/nginx:latest
		deploy:
			replicas: 2
			placement:
				constraints: [node.role != manager]
		depends_on:
			- api
		environment:
			WORKER_PROCESSES: 2
			WORKER_CONNECTIONS: 1024
			KEEPALIVE_TIMEOUT: 65
			GZIP: "on"
			BACKEND_HOST: todo_app_api:8080
			BACKEND_MAX_FAILS: 3
			BACKEND_FAIL_TIMEOUT: 10S
			SERVER_PORT: 80
			SERVER_NAME: todo_app_nginx
			LOG_STDOUT: "true"
		networks:
			- todoapp
```
