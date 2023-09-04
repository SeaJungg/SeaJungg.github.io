---
layout: post
title: "AWS EKS에 Redis 환경 구축하기"
category: inProgress
---

문제상황
-
- API서버 내 세션토큰 관리를 위해 Redis가 필요했다
- Redis가 제일 깔끔하긴 한데 사실 너무 귀찮아서 로컬에 저장할까 싶었다
- 마음을 굳게 먹고 Redis를 설치했다.

현황분석
-
- AWS EKS에 Redis를 설치하고, 그에 맞는 host name 등 각 pod간 서로를 부를 수 있도록 세팅해야 한다.
- Helm 을 이용하면 한 줄이면 Redis 환경이 완성되는데 Pending에 걸려서 넘어가지 않는다.

조치내용
- 
- k get pvc 를 해서 뒤져보면 원인을 알 수 있다.
- pv 공간을 잡으려는데 AWS권한이 없어서 안되는 것
```
[ec2-user@ip-172-31-89-237 ~]$ k get pvc
NAME                                     STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
redis-data-my-release-redis-master-0     Pending                                      gp2            25m
redis-data-my-release-redis-replicas-0   Pending                                      gp2            25m
redis-data-redis-service-master-0        Pending                                      gp2            22m
redis-data-redis-service-replicas-0      Pending                                      gp2            22m
[ec2-user@ip-172-31-89-237 ~]$ k describe pvc redis-data-redis-service-master-0
Name:          redis-data-redis-service-master-0
Namespace:     default
StorageClass:  gp2
Status:        Pending
Volume:
Labels:        app.kubernetes.io/component=master
               app.kubernetes.io/instance=redis-service
               app.kubernetes.io/name=redis
Annotations:   volume.beta.kubernetes.io/storage-provisioner: ebs.csi.aws.com
               volume.kubernetes.io/selected-node: ip-192-168-117-83.ec2.internal
               volume.kubernetes.io/storage-provisioner: ebs.csi.aws.com
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:
Access Modes:
VolumeMode:    Filesystem
Used By:       redis-service-master-0
Events:
  Type    Reason                Age                   From                         Message
  ----    ------                ----                  ----                         -------
  Normal  WaitForFirstConsumer  22m                   persistentvolume-controller  waiting for first consumer to be created before binding
  Normal  ExternalProvisioning  2m11s (x82 over 22m)  persistentvolume-controller  waiting for a volume to be created, either by external provisioner "ebs.csi.aws.com" or manually created by system administrator
[ec2-user@ip-172-31-89-237 ~]$ aws eks describe-addon-versions --addon-name aws-ebs-csi-driver
```
- https://docs.aws.amazon.com/eks/latest/userguide/managing-ebs-csi.html
- https://docs.aws.amazon.com/eks/latest/userguide/csi-iam-role.html