## 웹 서비스 구축
### 정적 파일을 다루는 방법
정적 파일은 node.js를 통해 제공할 필요가 없기 때문에 nginx에서 바로 제공하도록 설정할 필요가 있다. nginx의 conf 파일에 다음과 같은 내용을 추가한다.
```
server {
  location /_nuxt/ {
    alias /var/www/_nuxt/$1;
    {{ if var "LOG_STDOUT" }}
    access_log /dev/stdout json;
    error_log /dev/stderr;
    {{ else }}
    access_log /var/log/nginx/assets_access.log json;
    error_log /var/log/nginx/assets_error.log;
    {{ end }}
  }
}
```

이렇게 하면 /_nuxt/ 경로로 들어오는 요청은 api 서버를 통하지 않고 nginx 단에서 바로 처리한다. 그리고 api 서버 단에서 생성되는 정적 에셋 파일들을 nginx와 공유할 수 있도록 하기위해
volume 설정을 스택 정의 파일에 넣어야 한다.
`todo-frontend.yml`
```
version: "3"
services:
	nginx:
		image: registry:5000/ch04/nginx-nuxt:latest
		deploy:
			replicas: 2
		placement:
			constraints: [node.role != manager]
		depends_on:
			- web
		environment:
			SERVICE_PORTS: 80
			WORKER_PROCESSES: 2
			WORKER_CONNECTIONS: 1024
			KEEPALIVE_TIMEOUT: 65
			GZIP: "on"
			BACKEND_HOST: todo_frontend_web:3000 (이 nginx 서비스의 백엔드는 web 서비스임)
			BACKENT_MAX_FAILS: 3
			BACKENT_FAIL_TIMEOUT: 10s
			SERVER_PORT: 80
			SERVER_NAME: localhost
			LOG_STDOUT: "true"
		networks:
			- todoapp
		volumes:
			- assets:/var/www/_nuxt
			
	web:
		image: registry:5000/ch04/todoweb:latest
		deploy:
			replicas: 2
			placement:
				constraints: [node.role != manager]
		environment:
			TODO_API_URL: http://todo_app_nginx
		networks:
			- todoapp
		volumes:
			- assets:/todoweb/.nuxt/dist

networks:
	todoapp:
		external: true
		
volumes:
	assets:
		driver: local
```

db, api, web, nginx 까지 해서 스웜 클러스터 구축이 끝났기 때문에, 이 애플리케시연을 스웜 외부로 노출시키기 위한 인그레스가 필요하다. 이 또한 스택을 ㅗ만들어 클러스터에 배포한다.
`todo-ingress.yml`
```
version: "3"
services:
	haproxy:
		image: dockercloud/haproxy
		networks:
			- todoapp
		volumes:
			- /var/run/docker.sock:/var/run/docker.sock
		deploy:
			mode: global
			placement:
				constraints:
					- node.role == manager
		ports:
			- 80:80
			- 1936:1936

networks:
	todoapp:
		external: true
```
