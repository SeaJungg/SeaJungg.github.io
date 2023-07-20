---
layout: post
title: "산업공학 졸업하고 SKT쇼핑몰 API 만들다가 갑자기 LLM에 입문하게 된 과정"
category: done
---

우리 팀장님 피셜.. 회사 만족도의 40%는 연봉이 결정하고, 30%는 업무가, 나머지 30%는 동료가 결정한다 하셨다.

4:3:3 중 그 어느 것도 탁월하게 만족되지는 않았었는데
갑자기 3할이 LLM으로 채워졌다.

갑작스럽게 세상에 쏟아져나온 GPT 덕분이었다. (공부하고 나니 너무나 차근차근 확장되어 온 분야임) 

# 발단
"우리도 오픈AI로 뭐 하나 해보자" 라는 대표님의 한 마디가 구전되고 구전되어 말단 사원인 내게 전달되었고, 그래서 난 어느날 갑자기 OpenAI API로 서비스를 구현해서 오픈했다.

제공 데이터 : 메타 광고 집행 가이드    
구현 방법 : 메타 Q&A 및 도큐먼트 크롤링 > kobert 임베딩 > pinecone에 업로드 > Semantic search 진행 후 아주 기초적인 Prompt Engineering과 함께 3.5에 제공     
참고 아티클 1: https://www.mlq.ai/gpt-4-pinecone-website-ai-assistant/    
참고 아티클 2: https://www.youtube.com/watch?v=2FeymQoKvrk

![광고팀 동기들과 깔깔거리며 오픈함](..\assets\image\IMG_9807.JPG)

    
<br><br><br><br>    
# 전개

#### 회사가 쪼개졌다. 쪼개진 회사로 옮겨왔다.    
#### 여러 키워드들에 익숙해졌다. 키워드는 다음과 같다. A100은 아니더라도 koalpaca 8bit는 돌릴 수 있는 로컬서버를 적극 활용했다.
  - LLM
    - Vector Embedding
    - Chat Creation
  - Vector DB
  - Cosine similarity
  - Prompt Engineering    
#### 우후죽순 Chat GPT 시장에서 "노하우 있는 애들" 이 되려면 다음의 연구를 해야 한다.
  - LLM 선택
  - LLM에 대한 Fine Tuning(Prompt tuning)
  - Vector DB에 대한 Semantic Search, Hybrid Search의 최적 파라미터    

#### 최근 한 달간 다음과 같은 찍먹을 진행했다.
![찍먹.draw.io](..\assets\image\nyam.png)
    
    
<br><br><br><br>    
# 위기
    
    

#### #1
하나를 공부하고 나면 얻는 건, "1만큼의 지식"이 아니라 "내가 모르는 게 10가지라는 사실"이었다.    
이 현상은 아마 내가 100가지를 공부한다 해도 달라지지 않을 것이다.    
때아닌 직업가치관검사와 무료사주팔자 성격풀이(?)를 진행했다..    
![job1](..\assets\image\job1.png)    
![job2](..\assets\image\job2.png)    
개발자가 추천직업에 있다니... 개발이랑 안 맞는 것 같아요 를 유행어로 밀고있는데.
이런 걸 하고 있는 걸 보니 바쁘다 바쁘다 하면서 한가한 게 틀림없다.
    
        
           
#### #2
대표님만큼이나 이 사업에 진심인 변호사분들을 만나서 얘기도 나누고 [협약식](https://www.draju.com/ko/sub/firm_news.html?type=view&bsNo=2687)을 체결했다.
![대륙아주 협약식](..\assets\image.png)
나같은 변방의 개발자가 이 사이에 있다니...


    

#### #3
이 회사에는 어떤 류의 개발자가 필요할지 고민하고 있다.    
직접 구현하고 싶은 거도 많고 필요한 사내 툴들도 많다.    
일단 내가 만들고 싶은 백엔드 프러덕 두개를 기획하고 화면흐름도를 짜봤다.    
기획은 휘리릭 했는데 막상 개발하려니까 개발할 게 왜이렇게 많은지.. 아무튼 보고드렸고, 칭찬받았다.
    
```
#1 LLMOps
- 개요 : 점점 늘어나는 구성값 히스토리 관리하는 개발자 어드민
- 목적 : “이 설정값 이렇게 바꾸자” 같은 말로만 전달받고 서버 소스 직접수정하는 행위를 그만하고, 체계적으로 저장해가면서 실험하고 배포까지 연결하고 싶음
```
[LLMOps 서비스설계](..\assets\uploaded_file\first admin.pptx)
```
#2 query engine
- 개요 : 데이터 인덱스 만들고, 넣고, 찾는 기본기능만 하는거
- 목적 : 더이상 기본적인 쿼리코드 그만 짜고 싶음 
- 어떻게 짜면 좋을까?
    - 일단 로컬서버에서만 돌릴 목적으로 짜자
```
    
그리고 서비스 오픈과 동시에 있어야만 하는 백엔드 프러덕도 정의해봤다.
```
#1 시스템 모니터링 대시보드
- Cpu점유율이나.. 접속량 같은거
- token I/O도 잘 세서 시각화..
- 이거 aws agent로 구현하는게 나을수도 있음
- 돈 좀 들더라도 clowdwatch dash보드만드는게.. 
```
이건 금방 될 것 같아서 설계없이 당장 만들었다.    
![grafana](..\assets\image\2023-07-20grafana.png)
grafana와 prometheus를 기존 fast api로 만들어둔 서버에 잘 붙였다.      
실시간 토큰수와 fastapi와 관련한 지표들을 시각화했다.     
만들어놓고 나니 openai 를 위한 prometheus API든 패키지든 하나 나올 떄가 됐는데.. 없으면 내가 만들어야겠다.    
