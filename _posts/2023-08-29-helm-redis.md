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

### 환경 구성 퀵스타트
- https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/helm.html 를 참고해 헬름을 설치했다.
```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
helm repo add bitnami https://charts.bitnami.com/bitnami
```

- https://medium.com/@thanawitsupinnapong/setting-up-redis-in-kubernetes-with-helm-and-manual-persistent-volume-f1d52fa1919f 를 참고해 레디스를 설치했다.

```

helm install redis-service \
  --set cluster.slaveCount=3 \
  --set password=password \
  --set securityContext.enabled=true \
  --set securityContext.fsGroup=2000 \
  --set securityContext.runAsUser=1000 \
  --set volumePermissions.enabled=true \
  --set master.persistence.enabled=true \
  --set slave.persistence.enabled=true \
  --set master.persistence.enabled=true \
  --set master.persistence.path=/data \
  --set master.persistence.size=8Gi \
  --set slave.persistence.enabled=true \
  --set slave.persistence.path=/data \
  --set slave.persistence.size=8Gi \
bitnami/redis


NAME: redis-cluster
LAST DEPLOYED: Tue Oct 10 08:31:31 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: redis
CHART VERSION: 18.1.3
APP VERSION: 7.2.1

** Please be patient while the chart is being deployed **

Redis&reg; can be accessed on the following DNS names from within your cluster:

    redis-cluster-master.default.svc.cluster.local for read/write operations (port 6379)
    redis-cluster-replicas.default.svc.cluster.local for read-only operations (port 6379)



To get your password run:

    export REDIS_PASSWORD=$(kubectl get secret --namespace default redis-cluster -o jsonpath="{.data.redis-password}" | base64 -d)

To connect to your Redis&reg; server:

1. Run a Redis&reg; pod that you can use as a client:

   kubectl run --namespace default redis-client --restart='Never'  --env REDIS_PASSWORD=$REDIS_PASSWORD  --image docker.io/bitnami/redis:7.2.1-debian-11-r24 --command -- sleep infinity

   Use the following command to attach to the pod:

   kubectl exec --tty -i redis-client \
   --namespace default -- bash

2. Connect using the Redis&reg; CLI:
   REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h redis-cluster-master
   REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h redis-cluster-replicas

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/redis-cluster-master 6379:6379 &
    REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h 127.0.0.1 -p 6379
```