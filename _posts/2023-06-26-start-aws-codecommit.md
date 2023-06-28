---
layout: post
title: "AWS CodeCommit 시작하기"
category: done
---

### 사용자 만들기
- IAM 사용자를 만든다.
- 적절한 설정값으로 잘 만든 후 계정 관련 정보를 CSV로 저장한다.

### 이대로 끝내면 git 작업 시 403 뜸. IAM 권한 부여하기
- 권한 정책 우측 상단 권한 추가 > 권한 추가 클릭
- 권한 옵션에서 '직접 정책 연결' 클릭 후 검색창에 codecommit 검색
- AWSCodeCommitFullAccess, AWSCodeCommitPowerUser 를 선택함

### CodeCommit 에 들어가 리포지토리 생성하기
- 5명, 50기가까지 무료이니 엄청난 이상한 짓만 안하면 웹서비스 배포 정도 목적으로는 사실상 무료로 이용할 수 있다.
- 다 만든 다음 URL 복제 > HTTPS 복제 클릭

### 로컬 커맨드창으로 돌아온다.
- 로컬 컴퓨터에 git이 설치되어 있다고 가정한다.
- `git clone {복제한 HTTPS URL}`
- 아까 다운받은 csv 파일을 열어 UserName과 Password를 확인한다.
- 입력을 요구할 때 하나하나 넣어준다.
- 정상적으로 clone이 실행완료된다.

### git관련 커맨드들 잘 되는지 수행
- 기존에 어딘가에 푸시한 적이 있는 레포가 있고, 통쨰로 aws에 올리고 싶다고 가정한다.
- awsorigin 이라는 repo를 추가하고 푸시한다
```cmd
git remote add awsorigin {복제한 HTTPS URL}
git push -u awsorigin main
```