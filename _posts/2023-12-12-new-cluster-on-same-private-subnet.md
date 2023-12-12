---
layout: post
title: "기존 private subnet에 새 클러스터 추가하기"
category: done
---

- EKS 클러스터는 태그를 기준으로 본인이 쓸 수 있는 서브넷을 검색한다.
- 새 클러스터를 프라이빗 서브넷에 추가했다면, 그 서브넷들에 모두 태깅을 해주어야 한다.
- 지정한 private subnet들에 들어가 `kubernetes.io/cluster/cluster-name` 를 추가해준다.
- 참고 : https://repost.aws/ko/knowledge-center/eks-vpc-subnet-discovery