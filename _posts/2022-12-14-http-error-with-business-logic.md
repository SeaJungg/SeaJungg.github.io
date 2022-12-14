---
layout: post
title: "http 에러가 아닌데 httpErrorCode를 리턴해도 되나?"
category: inProgress
---

### 현상 파악
- 우리 서비스의 로그 서버를 새로 만드는데 고민이 생겼다.
- 사내뿐 아니라 사외에서도 request를 보낼 가능성이 있어서, 국제규격 비슷한게 있다면 지키고 싶었다.
- 파라미터가 누락되었을 때 에러를 일관성 있고 관리가 편하게 리턴하고 싶었다.
- 에러처리용 미들웨어를 만들어보았다.
- https://simonplend.com/how-to-create-an-error-handler-for-your-express-api/

### 문제 정의
- '파라미터 누락' 은 비즈니스 관점에서의 오류인데 이걸 http error code를 활용해서 리턴해도 되는가 ?????
- 근데 이미 많은 서비스에서 그러고 있음을 이미 알고 있다. (카카오, ncp)
- https://stackoverflow.com/questions/3050518/what-http-status-response-code-should-i-use-if-the-request-is-missing-a-required