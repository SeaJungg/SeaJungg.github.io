---
layout: post
title: "private subnet에 배포된 EKS ingress-controller가 public subnet에 loadbalancer 만들게 하기"
category: done
---

ingress-nginx 를 배포할 때, Service에서 아래의 라인을 삭제한다.
```
service.beta.kubernetes.io/aws-load-balancer-internal: "true"
```
그리고, 현재 클러스터가 위치한 Private subnet과 같은 VPC의 public subnet에 다음의 태그를 추가해야 한다.
```
  key         = "kubernetes.io/cluster/${local.cluster_name}"
  value       = "shared"
```
이미 배포한 적이 있는 subnet이라면 아래의 태그가 있겠지만, 혹시나 없다면 아래의 내용도 추가해준다.
```
  key         = "kubernetes.io/role/elb"
  value       = "1"
```