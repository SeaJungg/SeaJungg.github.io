---
layout: post
title: "python, poetry 환경에서의 package 관리하고 동료 개발자에게 공유하기"
category: done
---

목표
-
- 함께하게 된 동료 개발자가 정상적으로 로컬에서 돌릴 수 있게 하기
- 팀 내 개발환경 공유에 관해 검색해보니 그냥 아키텍트가 딱 정해준단 얘길 발견했다.
- 그럼 나도 그러면 되겠다.
    


쓸모없는 패키지 지우기
-
- 개발하다보면 실험적으로 설치했던 패키지가 pyproject에 흘러가는 경우가 있다.
    

python 및 poetry버전을 리드미에 명시하기
- 
- 플랫폼과 관계 없이 모두가 같은 개발환경을 구축해야 한다.
- python 및 poetry 버전이 달라지면 모든 패키지의 버전이 달라진다.
- python==3.10.*, poetry==1.5.1 로 이번 셋팅을 진행한다.

공유받는 사람이 직접 개발에 참여하는지, 돌리기만 하면 되는지 파악하기
-
- 로컬에서 서버가 도는 것까지만을 바란다면 

pyenv version 으로 원하는 파이썬 버전 맞추기
-
- poetry는 python major version 까지만 정합성을 잡아준다.
- 패키지별로 patch 버전까지 단속하는 경우도 있을 수 있으니 한 번 더 체크해보자.
    

    
poetry 명령어들로 버전 체크하기
-
- poetry show --tree 으로 전체 그림을 볼 수 있다. poetry show --tree > tree.txt 로 저장 후에 검색해가면서 보자.
- poetry show {package_name} 으로 특정 패키지의 이름, 버전, 설명과, 어떤 패키지에 이 패키지가 의존중인지를 볼 수 있다.
    


완전히 새로운 디렉토리에 가상환경 만들고 나서 차근차근 체크해보기
-
```
# 디렉토리 만들고 혹시나 켜져 있을 가상환경 전부 종료
mkdir test_python_ver
cd test_python_ver
deactivate

# major, minor 버전까지는 같고 patch 버전만 다른 경우에도 실행되는지 확인하기 위해 3.10.12 설치
# 작업은 3.10.0에서 했고, 확인하려는 버전은 3.10.12이다.
pyenv install 3.10.12

# 해당 디렉토리에서 사용할 파이썬 버전을 명시한다.
pyenv local 3.10.12

# 파이썬 버전 확인
pyenv version
python --version 
python3 --version

# 해당 파이썬 버전으로 가상환경 만들기
python -m venv testvenv
source testvenv/bin/activate


# 명시할 poetry 버전 설치하기
pip install poetry==1.5.1


# 가상환경에 정확하게 설치되었는지 확인하기
testvenv/bin/python -m poetry --version


# git clone 후 해당 디렉토리, 해당 브랜치에 위치시키기
git clone {repo_url}
cd {repo_dir}
git checkout dev


# poetry 로 패키지 설치
poetry env use 3.10
poetry install



# 필요한 환경변수 주입 후 실행
export SOME_KEY=value
poetry run start
```
    
    

        



