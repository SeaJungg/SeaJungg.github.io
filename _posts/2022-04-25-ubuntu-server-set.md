---
layout: post
title: "누구나 접근 가능한 서버환경을 아무것도 없는 상태에서부터 구성하기"
category: inProgress
---

윈도우에 우분투를 깔았다.
기존 앱 노드 버전이 신규 프로젝트와 맞지 않았다.


apt install tree


https://dydals5678.tistory.com/100

를 하려다가

https://integer-ji.tistory.com/370

한 뒤에 sudo apt install npm

하고 나서 npm install pm2

하고 나서 로컬에 있는 .js를 띄우고 싶어서 찾아보니

C드라이브의 위치는 /mnt/c

했는데 node 버전이 10...

해서 업데이트
https://phoenixnap.com/kb/update-node-js-version

nvm ls-remote
nvm install 16.15.0

했는데 WSL 안돼있어서 설치..
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

해도 안돼서..
https://rottk.tistory.com/entry/WSL-%EA%B0%80%EC%83%81-%EC%BB%B4%ED%93%A8%ED%84%B0-%ED%94%8C%EB%9E%AB%ED%8F%BC-Windows-%EA%B8%B0%EB%8A%A5%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8F%84%EB%A1%9D-%EC%84%A4%EC%A0%95%ED%95%98%EA%B3%A0-BIOS%EC%97%90%EC%84%9C-%EA%B0%80%EC%83%81%ED%99%94%EA%B0%80-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8F%84%EB%A1%9D-%EC%84%A4%EC%A0%95%EB%90%98%EC%96%B4-%EC%9E%88%EB%8A%94%EC%A7%80-%ED%99%95%EC%9D%B8-%ED%95%98%EC%84%B8%EC%9A%94



하고나니 이제 로컬호스트에 떤지면 우분투가 안받음.

WSL 설정에 의해 나눠 쓰게 되었다 ....(?)


