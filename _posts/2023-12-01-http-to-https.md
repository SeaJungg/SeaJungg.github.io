---
layout: post
title: "이미 구축한 ingress-nginx 환경에 https 인증서 붙이기"
category: done
---

요약
-
- External DNS가 설치되어 있어야 한다.
- AWS Certificate Manager console에서 인증서를 만든다.
- https://kubernetes.github.io/ingress-nginx/deploy/#aws 를 참고했다.
- deploy.yaml파일을 다운받은 후 열어서 `VPC CIDR`과 `SSL/TLS 인증서의 ARN` 만 수정하고 k apply -f 로 적용한다.
- 이후 원하는 Ingress를 등록하면 알아서 https 설정과 http > https 리다이렉트를 수행한다.


하고자 하는 작업
-
- 인그레스와 인그레스 컨트롤러를 사용하고 있으면, SSL/TLS 설정을 인그레스 컨트롤러에서 해주면 된다.
- 현재 EKS 는 ingress-nginx(ingress와 ingress controller, configmap 등 많은 오브젝트들로 이루어져 있음) External DNS 로 구성되어 있으니 여기에 맞게 적용한다.

인증서 만들기
-
- Route 53에 등록하여 쓰고 있는 DNS에 SSL/TLS 인증서를 발급한다.
- AWS Certificate Manager console에서 '인증서 요청'을 클릭한다.
- 서브도메인들을 모두 서빙해야 하므로 *.{도메인 이름}.com 으로 등록하고,검증 방법을 'DNS 인증' 으로 선택한 후 요청을 마친다.
- 인증서 요청을 마친 페이지에서 cname 등록을 통한 검증 절차를 거치면 곧바로 발급된다.


Ingress 추가하기
- 원하는 ingress를 추가하면 External DNS가 cname 을 추가해주고, 도메인이 같은 애면 ingress-nginx가 모니터링하며 인증서도 붙여주고 리다이렉트도 해준다.
- 하기와 같은 형태로 파일을 만들었다.
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: alb
    nginx.ingress.kubernetes.io/cors-allow-methods: PUT, POST, GET, OPTIONS
    nginx.ingress.kubernetes.io/cors-allow-origin: http://*.whatever.com
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
  name: ingress-nginx
  namespace: whatever
spec:
  ingressClassName: nginx
  rules:
  - host: whatever-dev.whatever.com
    http:
      paths:
      - backend:
          service:
            name: whatever-whatever-app-service
            port:
              number: 8080
        path: /
        pathType: Prefix
  - host: whatever-dev2.whatever.com
    http:
      paths:
      - backend:
          service:
            name: whatever-whatever-app-service
            port:
              number: 8080
        path: /
        pathType: Prefix
```

이후의 내용들은 모두 삽질의 기록이라 볼 필요가 없는데, 미래의 내가 심심해서 볼 수도 있으니까 남겨둔다.
-
- https로 접속이 되길래 다 된 줄 알았는데 http 가 접속이 안되어 설정을 이리저리 만지다보니 tls termination 관련해서 too many redirect 지옥에서 뻉뺑 돌았다.
- 추상화의 추상화라 편하게 쓸 수 있는 쿠버네티스지만 쉽게만 되지는 않는다.
- https://fastapi.tiangolo.com/deployment/https/ 여기 설명이 친절하고 좋았다.
- 눈감고 코끼리 만들려던 내 잘못이다. 네트웍 기본기를 다지는 계기가 되었다.
```
nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
nginx.ingress.kubernetes.io/ssl-passthrough: "true"
```
- 위의 두 설정이 충돌한 것도 원인이었고, deployment에서 "나 여기로 보낼께" 하고 정해놓은 포트 이름을 정작 컨테이너에서는 정의해두지 않아서 no response가 뜬 것도 원인이었다.


더보기 .......
<br><br><br><br><br><br><br><br>


수정 내용 (1) ingress-nginx-controller
- `인증서의 ARN` 텍스트와, EKS 클러스터가 사용 중인 `VPC CIDR` 이 필요하다.
- `k edit service ingress-nginx-controller` 로 ingress controller를 수정한다.
  - annotation 에 아래의 여섯 줄을 한 줄도 빠짐없이 추가한다.
```
annotation
...
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: http
    service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "60"
    service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:XXXXXXXX # 인증서의 ARN
    service.beta.kubernetes.io/aws-load-balancer-ssl-ports: https
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
```
  - ports에 아래의 값들을 한 줄도 빠짐없이 추가한다.
```
spec:
...
  ports:
  - appProtocol: http
    name: http
    nodePort: 30519 # 이 값은 수정하지 않는다.
    port: 80
    protocol: TCP
    targetPort: tohttps
  - appProtocol: https
    name: https
    nodePort: 30907 # 이 값은 수정하지 않는다.
    port: 443
    protocol: TCP
    targetPort: http  # 이거 빠지면 400 Bad Request "HTTP request was sent to HTTPS port" 발생함
```

수정 내용 (2) deployment
-
- 포트를 더 열어줘야 한다.
```
# previous
...
        name: controller
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
        - containerPort: 443
          name: https
          protocol: TCP
        - containerPort: 8443
          name: webhook
          protocol: TCP
...
# to be
...
        name: controller
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
        - containerPort: 80
          name: https
          protocol: TCP
        - containerPort: 2443
          name: tohttps
          protocol: TCP
        - containerPort: 8443
          name: webhook
          protocol: TCP
...
```

수정 내용 (3) configmap
-
- `k edit configmap ingress-nginx-controller` 로 configmap을 수정한다.
```
  allow-snippet-annotations: "true"
  http-snippet: |
    server {
      listen 2443;
      return 308 https://$host$request_uri;
    }
  proxy-real-ip-cidr: 10.XXXXX #VPC의 CIDR
  use-forwarded-headers: "true"
```
결과
-
- EC2 > 로드밸런서 > ingress-nginx 가 들고 있는 DNS 이름 클릭 > 리스너에 들어가면, TLS:443 포트에 보안 정책과 기본 SSL/TLS 인증서가 붙어있다.
- https에서 too many redirect 에러가 발생한다.

```
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/aws/nlb-with-tls-termination/deploy.yaml
```
위의 명령어를 진행한 후 cidr과 acm arn을 실제 값으로 치환한다. 그런 다음 적용을 해보면, 부분적으로 적용한다고 해놓고 미흡했던 것들이 추가로 적용된다.
```
Warning: resource namespaces/ingress-nginx is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
namespace/ingress-nginx configured
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
Warning: resource clusterroles/ingress-nginx is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
clusterrole.rbac.authorization.k8s.io/ingress-nginx configured
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
Warning: resource clusterrolebindings/ingress-nginx is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx configured
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
Warning: resource ingressclasses/nginx is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
ingressclass.networking.k8s.io/nginx configured
Warning: resource validatingwebhookconfigurations/ingress-nginx-admission is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission configured
```

결론
-
- 재야의 고수들이 만들어논 헬름 차트를 손으로 옮길 생각을 하지 말자.