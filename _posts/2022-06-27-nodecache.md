---
layout: post
title: "key가 없을 때 뭘 리턴할까? node-cache, ioredis"
category: done
---


> node-cache : undefined
> ioredis : null



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