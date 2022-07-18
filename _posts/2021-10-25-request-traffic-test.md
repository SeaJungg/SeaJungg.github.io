---
layout: post
title: "분산 환경에서 express server 트래픽 테스트"
category: done
---

메시지 서버에 요청을 보내는 메시지 모듈(express server)이 있다.
기존에는 SKT서버에만 요청했었는데, TOAST쪽으로도 요청을 보내게 되면서 새로운 router를 만들었다.
만들면서 전체 서버 리팩토링과 트래픽 테스트를 진행했다.
일평균 200만 건의 request를 처리 및 생성하므로 정말 `잘` 해야 했음.