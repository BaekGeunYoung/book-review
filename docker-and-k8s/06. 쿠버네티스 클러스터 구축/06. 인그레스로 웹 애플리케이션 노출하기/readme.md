### 인그레스(ingress)
인그레스 또한 쿠버네티스 리소스 중 하나이므로 매니페스트 파일로 간단하게 인그레스를 클러스터에 반영할 수 있다.
`ingress.yaml`
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress
spec:
  rules:
  - http:
      paths:
      - path: /*
        backend:
          serviceName: todoweb
          servicePort: 80
```
어떤 서비스의 어떤 포트를 외부에 노출시킬 지에 대한 것을 rules 필드 내에서 정의할 수 있고, 이를 클러스터에 적용한 후 할당받은 글로벌 IP 주소는
todoweb으로 연결된다.
GCP 콘솔을 보면 지금껏 만든 서비스 및 인그레스를 확인할 수 있고, 인그레스에 할당되어 있는 엔드포인트로 접속해보면 정상적으로 투두리스트가 잘 뜨는 것을 확인할 수 있다.

![cluster](https://github.com/BaekGeunYoung/book-docker-and-k8s/blob/master/images/cluster.PNG)

![todoweb](https://github.com/BaekGeunYoung/book-docker-and-k8s/blob/master/images/todoweb.PNG)
