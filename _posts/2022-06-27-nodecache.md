---
layout: post
title: "key가 없을 때 뭘 리턴할까? node-cache, ioredis"
category: done
---

- '손 닿는 곳에 두고 빠르게 꺼내도록' 저장하는 선택지는 node-cache와 redis 정도중에 고를 수 있다.
같은 역할이라 간과했는데, key가 없을 때 둘이 리턴하는 값이 다르다.
- [node cahce와 redis 비교](https://www.c-sharpcorner.com/blogs/difference-between-node-cache-and-redis-cache)

---
## 값 없을 때
## > node-cache : undefined    
## > ioredis : null    
---

### node-cache 왜 쓰나?
- 네트워크 일시 중단 사태 시, 일시적 수습을 위해 사용했음
- 보통 네트워크가 오래 끊길 리가 없으니까 ttl을 짧게 잡았음

### redis 왜 쓰나?
- 캐시처럼 빠르게 가져오는 건 똑같음
- 네트워크 필요(같은 장비에 띄워두지 않아서이긴 하지만)

```js
const nodeCache = require('node-cache');
const cache = new nodeCache({stdTTL:2});

function setConfig(callback) {
    cache.set('config','sea', (err, res) => {
        callback(err, res)
    })
}

function getConfig(callback) {
    cache.get('config', function(err, value) {
        callback(err, value)
    });
  }

setConfig((err, res) => {
console.log(res); //true

getConfig((err, res) => {
    console.log(res); //sea
    })

setTimeout(() => {
getConfig((err, res) => {
    console.log(res); //undefined
    })
}, 2000)
})
```

```js
const Redis  =require('ioredis');
redis = new Redis({host:redis, port:6379})

function store(callback) {
    redis.setex('configsea', 2, 'sea', (err, res) => {
        callback(err, res)
    })
}

function read(callback) {
    redis.get('configsea', (err,res) => {
        callback(err, res)
    })
}

store((err, res) => {
    console.log(err);
    console.log(res); //OK
    
    read((err, res) => {
        console.log(err);
        console.log(res); //sea
    })

    setTimeout(() => {
        read((err, res) => {
            console.log(err);
            console.log(res); //null
        })
    })
})
```