---
layout: post
title: "산업공학 졸업하고 쇼핑몰 API 만들다가 갑자기 LLM에 입문하게 된 과정"
category: inProgress
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
#### 여러 키워드들에 익숙해졌다. 키워드는 다음과 같다.
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
하나를 공부하고 나면 얻는 건, "1만큼의 지식"이 아니라 "내가 모르는 게 10가지라는 사실"이었다.    
이 현상은 아마 내가 100가지를 공부한다 해도 달라지지 않을 것이다.    
때아닌 직업가치관검사와 무료사주팔자 성격풀이(?)를 진행했다..    
![job1](..\assets\image\job1.png)    
![job2](..\assets\image\job2.png)    
애국심 쌈싸먹은 디지털노마드 지망생으로서 적절한 가치관인 것 같다. 그리고 개발자가 추천직업에 있다.
    
        
           
이런 걸 하고 있는 걸 보니 바쁘다 바쁘다 하면서 한가한 게 틀림없다.