---
layout: post
title: "root로 뭘 하려고 하지 말자"
category: inProgress
---
- docker로 mariadb를 띄우고 잘 사용중이었다.
- 계정은 따로 안만들고 root를 사용했다.
- 비밀번호를 설정한 적이 없는데 어느날 갑자기 Access denied가 수습이 안된다.
- 해당 volume으로 새 컨테이너를 띄워도 수습이 안된다.
- root 쓰지 말자.