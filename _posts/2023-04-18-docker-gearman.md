---
layout: post
title: "무에서 devops 창조하기 (1)현황대로 도커에서 아키텍처 구성하기 (gearman, zookeeper)"
category: inProgress
---

### 현상 파악
- front api 를 테스트해야 한다.
- 그런데 엮인 게 많아서 front api, gearman, zookeeper 까지는 같이 띄워야 서버를 돌릴 수가 있다...

### 조치 내용
- 도커에 gearman 설치
```cmd
sudo docker run -p 4730:4730 -d --name gearman artefactual/gearmand:latest
```
- 잘 됐는지 확인
```cmd
gearadmin -h localhost -p 4730 --show-jobs
```