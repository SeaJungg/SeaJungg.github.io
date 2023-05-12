---
layout: post
title: "clean layer in Node.js express server"
category: inProgress
---

러닝크루 앱을 장고로 만들다가 노드로 넘어오게 되었다.
날쿼리를 날리려다가 크루 내 쿼리단속반에 적발되어 ORM과 clean layer를 알게 됐다.
지금까지 내가 사내에서 습관적으로 만들어 온 코드들이 얼마나 삶의 질을 떨어뜨렸는지 체감하게 됐다.


```cmd
.
├── app
│   ├── app.js
│   ├── controllers
│   │   ├── session.controller.js
│   │   ├── sessionHistory.controller.js
│   │   └── user.controller.js
│   ├── database
│   │   ├── migrations
│   │   │   └── _20230426154442-add-user-name-to-users.js
│   │   ├── models
│   │   │   ├── Session.js
│   │   │   ├── SessionHistory.js
│   │   │   ├── User.js
│   │   │   ├── _index.js
│   │   │   └── index.js
│   │   └── seeders
│   │       ├── 20230425100515-userData.js
│   │       ├── 20230425100743-sessionData.js
│   │       └── 20230425100752-sessionHistoryData.js
│   ├── logger.js
│   ├── middleware
│   │   ├── auth.js
│   │   └── httpRequest.js
│   ├── routes
│   │   ├── index.js
│   │   ├── sessionHistories.js
│   │   ├── sessions.js
│   │   └── users.js
│   ├── services
│   │   ├── sessionHistories.js
│   │   ├── sessions.js
│   │   └── users.js
│   ├── swagger-output.json
│   ├── swagger.js
│   └── test.html
├── config
│   └── config.json
├── package-lock.json
├── package.json
└── request.rest
```