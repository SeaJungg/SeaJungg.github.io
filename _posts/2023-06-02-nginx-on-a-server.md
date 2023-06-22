---
layout: post
title: "AWS EC2 한 대에 nginx 설정해서 여러 서비스(FastAPI, streamlit) 배포하기"
category: done
---

### 현상 파악
- EC2를 만들면 퍼블릭 DNS를 하나 준다.
- 그 하나를 알뜰살뜰하게 사용해서, nginx로 proxy를 구성해서, uri 표기를 통해 프론트서버도 붙고 백서버도 붙고싶다.
- 지금은 한 대지만 추후 여러 대 쓰거나 컨테이너 환경으로 넘어갔을 때에도 작업량이 별로 없는 방안으로 구성하고 싶다.
  - nginx분기 하겠다고 [Fast API root path구성](https://fastapi.tiangolo.com/advanced/behind-a-proxy/) 하는 등의 소스 수정은 하기 싫단 말
  - 아닌가? 오히려 저렇게 해놓는 게 편할까? 나는 언제까지 챗지피티한테 is it weird? Is it common way? 를 물어봐야 하는가?

### nginx 설정
- nginx의 역할은 두 가지
  - 정적 파일을 처리하는 HTTP 서버로서의 역할 : HTML, CSS, Javascript, 이미지와 같은 정보를 웹 브라우저(Chrome, Iexplore, Opera, Firefox 등)에 전송(HTTP 프로토콜을 준수)
  - 응용프로그램 서버에 요청을 보내는 리버스 프록시로서의 역할 : 클라이언트는 가짜서버인 엔진엑스(프록시 서버)에 요청하고, 엔진엑스가 요청을 배분하여 응용프로그램(리버스 서버)에 전달

- 8501에는 프론트 서버인 streamlit 을 배포하고, 8000에는 백엔드 서버인 FastAPI를 배포할 것이다.

```
cd /etc/nginx/sites-available
sudo vi {AWS에서 준 이름}.amazonaws.com
```
- 아래 내용을 입력한 후 :wq
```
server{
       server_name {AWS에서 준 이름}.amazonaws.com;
       location / {
           include proxy_params;
           proxy_pass http://127.0.0.1:8501;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "upgrade";
       }
       location /api {
           include proxy_params;
           proxy_pass http://127.0.0.1:8000;
       }
}
```
- 프론트 서버에는 세 줄이 더 써있는데 WebSocket (ws) 관련 설정도 해준 내용이다.
  - streamlit에서 `ws://{AWS에서 준 이름}.amazonaws.com/_stcore/stream` 에다가도 request를 보내기 때문에.

- 저렇게 하고 나서 심볼릭 링크를 만든다.
```cmd
sudo ln -s {AWS에서 준 이름}.amazonaws.com -> /etc/nginx/sites-available/{AWS에서 준 이름}.amazonaws.com
```

    
- 이제 재시작하면 바로 접속이 된다.
```cmd
sudo service nginx restart
```

- 혹시나 죽어라고 nginx 기본 페이지로만 간다면 기본값을 과감히 삭제하자.
```cmd
sudo rm /etc/nginx/sites-available/default
sudo rm /etc/nginx/sites-enabled/default
```
    
### FastAPI 배포
- gunicorn 배포방법대로 했다.
```cmd
python3 -m gunicorn -k uvicorn.workers.UvicornWorker main:app --daemon --access-logfile ./gunicorn-access.log
```

### streamlit 배포
- tmux를 이용해 배포했다. 솔직히 이게 맞는지 모르겠는데, 아무튼 급하게 대표님이 찾거나 할 때 휘리릭 배포하기 매우 좋아서 이렇게 했다.
- tmux 세션을 하나 만들고, `tmux ls`로 세션에 붙고, `tmux attach -t 세션번호` 로 붙은 다음, `streamlit run app.py`를 하면 바로 실행됨.
- 그 후 컨트롤b를 누르고 d(just d) 를 누르면 살아있다.
- [참고 아티클](https://medium.com/@data.science.enthusiast/how-to-deploy-a-streamlit-machine-learning-app-over-aws-ec2-instance-12b6751268f1)