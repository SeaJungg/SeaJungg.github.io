---
layout: post
title: "이미 구축한 ingress-nginx 환경에 https 인증서 붙이기"
category: done
---

들어가며
-
- https://kubernetes.github.io/ingress-nginx/deploy/#aws 를 참고했다.
- deploy.yaml파일을 통째로 적용하면 바로 된다는데, 나는 이미 생성되어 있는 오브젝트들이 많으니 차이점만 짚어 빠르게 추가한다.

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
수정 내용 (2) configmap
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
- 감격스럽다.