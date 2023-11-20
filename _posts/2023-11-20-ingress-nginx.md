---
layout: post
title: "AWS EKS에서 externalDNS와 ingress-nginx 설정 시행착오 정리"
category: inProgress
---
    
해놓고 나니까 어렵지 않은데, 딱 맞는 usecase tutorial이 없어서 삽질이 길었다.


0. 작업환경
- elsctl과 kubectl이 모두모두 정상 동작하는 아늑한 클러스터 및 ssh
    



1. externalDNS 가 뭐하는 놈인지?
- 쿠버네티스 안에서 생겨나는 오브젝트들에 따라 AWS Route53을 수정해준다.
- 그러면 도메인 이름과 쿠배 안 서비스를 매칭할 수 있다.
- 클릭노가다 해방작업의 일환이다. CNAME 추가를 콘솔에서가 아니라 쿠배 안에서 할 수 있게 된다.
    
    


2. externalDNS 설정 with rbac
- export EXTERNALDNS_NS=externaldns 로 독립된 네임스페이스를 지정해줬다.
- AWS에 변화를 만드는 작업이기에 권한을 줘야한다.
- EKS 클러스터가 가진 권한으로 이 작업을 하는 게 아니라, 쿠버네티스 안 ServiceAccount가 가진 권한으로 이 작업을 한다.
- IRSA, RBAC 에 대한 이해를 먼저 하고 진행하길 강력히 권장한다.
- AWS policy를 만들고, 쿠버네티스 ServiceAccount와 연결한다.
```
vi policy.json
```
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "route53:ChangeResourceRecordSets"
      ],
      "Resource": [
        "arn:aws:route53:::hostedzone/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "route53:ListHostedZones",
        "route53:ListResourceRecordSets",
        "route53:ListTagsForResource"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
```
```
aws iam create-policy --policy-name "AllowExternalDNSUpdates" --policy-document file://policy.json
# 하고 나서 해당 policy의 POLICY_ARN 를 가져와서 export를 지정해주거나 아래 커맨드에 그대로 넣어준다.
eksctl create iamserviceaccount   --cluster 클러스터이름   --name "external-dns"   --namespace ${EXTERNALDNS_NS:-"default"}   --attach-policy-arn $POLICY_ARN   --approve
```
    
- 아래 링크에 접속해  externaldns-with-rbac.yaml 의 내용을 얻은 후 파일을 만든다.
- https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/aws.md#manifest-for-clusters-with-rbac-enabled
```
kubectl create --filename externaldns-with-rbac.yaml \
  --namespace ${EXTERNALDNS_NS:-"default"}
```

2. ingress-nginx 설정
```
helm install ingress-nginx ingress-nginx/ingress-nginx
```
```
vi ingress.yaml
```
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: ingress-nginx-front-stg
  namespace: somenamespace
spec:
  ingressClassName: nginx
  rules:
  - host: some_domain.com
    http:
      paths:
      - backend:
          service:
            name: chat-app-service
            port:
              number: 8080
        path: /api(/|$)(.*)
        pathType: Prefix
      - backend:
          service:
            name: front-app-service
            port:
              number: 3000
        path: /
        pathType: Prefix
```