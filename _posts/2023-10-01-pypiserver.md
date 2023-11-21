---
layout: post
title: "pip private repository 만들기"
category: done
---

준비물
-
- docker 가 설치된 AWS EC2
   
설치
-
- 아래 링크대로 진행했다.
- https://hub.docker.com/r/pypiserver/pypiserver 
   

이후 해야하는 작업
-
- 계정 만들기
- 해당 포트에 잘 접속되는지 확인하기
- 클라이언트 측에서 어떻게 업로드하면 되는지 가이드하기 

클라이언트 측 사용방법
-
```
vi ~/.pypirc
```
```
[distutils]
index-servers =
    local
[local]
repository: http://{instance_public_ip}:{port_for_running_container}
username: {user_name}
password: {user_password}
```
```
최상위 디렉토리에 setup.py 생성
```
```
from setuptools import setup, find_packages

with open("README.md", "r") as fh:
    long_description = fh.read()

setup(
    name="package_name",
    version="0.0.1",
    author="정세아",
    author_email="{some_email}",
    description="Package for something",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/{some_repo}",
    packages=['some_package'],
    classifiers=[
        "Programming Language :: Python :: 3"
    ],
    install_requires=[
       'aiohttp',
    ],
    python_requires='>=3.9',
)
```
    


관리방법
-
- 업로드한 패키지들은 docker local 에 저장됨
- docker ps로 컨테이너 ID 를 확인하고 exec실행 
```
docker exec -it {container_id} /bin/bash
```
- /data/packages 에 들어가면 확인 가능