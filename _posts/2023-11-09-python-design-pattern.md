---
layout: post
title: "Singleton-Observable pattern combo를 사용해 config값 우아하게 가져오기"
category: inProgress
---

만들고자 했던 것
-
- fastapi 서버가 있다. 그것은 server/main.py 에서 실행되고 있다. 이 코드는 service/ directory에서 다양한 서비스를 가져온다. 이 소스들은 모두 '컨피그 정보'에 접근할 수 있다. '컨피그 정보'은 AWS parameter store에 저장되며 주기적으로 새로 고쳐야 한다. 그리고 '컨피그 정보'을 얻는 작업은 서버의 모든 요청과 별도로 트리거되어야 한다.
    


만든 것
-
- 만들고자 했던대로 동작은 한다.
- main.py에서 가져오는 방식대로, 다른 모든 서비스들에서도 참조할 수 있다.
```
### main.py
...
from modules.settings import settings
...
CHAT_MODEL = settings.CHAT_MODEL
CHAT_REFER_TOP_K = settings.CHAT_REFER_TOP_K
...
assert CHAT_MODEL is not None
assert CHAT_REFER_TOP_K is not None
...
@app.on_event("startup")
@repeat_every(seconds=settings.SETTING_RELOAD_PERIOD_SECONDS, wait_first=True)
    app.logger.info('reload environmetal variables')
    settings.update()
    global CHAT_MODEL
    global CHAT_REFER_TOP_K
...
    CHAT_MODEL = settings.CHAT_MODEL
    CHAT_REFER_TOP_K = settings.CHAT_REFER_TOP_K
...
    assert CHAT_MODEL is not None
    assert CHAT_REFER_TOP_K is not None
```
```
### module/settings.py
# settings.py
import boto3
from botocore.exceptions import ClientError
import time
import os
from loguru import logger


REGION = os.getenv('REGION')
ENV = os.getenv('ENV')
SETTING_RELOAD_PERIOD_SECONDS=int(os.environ.get('SETTING_RELOAD_PERIOD_SECONDS'))

assert REGION is not None
assert ENV is not None
assert SETTING_RELOAD_PERIOD_SECONDS is not None

class Settings:
    def __init__(self):
        setattr(self, 'SETTING_RELOAD_PERIOD_SECONDS', SETTING_RELOAD_PERIOD_SECONDS)
        self.update()

    def get_parameter(self, name, region_name=REGION, with_decryption=False):
        ssm = boto3.client('ssm', region_name)
        parameter = ssm.get_parameter(Name=name, WithDecryption=with_decryption)
        return parameter['Parameter']['Value']

    def update(self):
        try:
            ssm = boto3.client('ssm', region_name=REGION)
            logger.info(f"Connected to AWS Parameter store at {REGION} successfully")
            paginator = ssm.get_paginator('describe_parameters')
            for page in paginator.paginate(
                ParameterFilters=[
                    {
                        'Key': 'Name',
                        'Option': 'BeginsWith',
                        'Values': [
                            '/chat-teamjang/'+ENV+'/',
                        ]
                    },
                ],
            ):
                for parameter in page['Parameters']:
                    name = parameter['Name'].split('/')[-1]
                    value = self.get_parameter(parameter['Name'])
                    logger.debug(f'Got parameter {name}={value}')
                    setattr(self, name, value)

            for page in paginator.paginate(
                ParameterFilters=[
                    {
                        'Key': 'Name',
                        'Option': 'BeginsWith',
                        'Values': [
                            '/prompt/final-answer-with-abstract/',
                        ]
                    },
                ],
            ):
                messages = [0] * len(page['Parameters'])
                for parameter in page['Parameters']:
                    ordering = parameter['Name'].split('/')[-2]
                    role = parameter['Name'].split('/')[-1]
                    value = self.get_parameter(parameter['Name'])
                    messages[int(ordering) - 1] = {
                            "role" : role,
                            "content" : value
                    }
                logger.debug(f'Prompt : {messages}')
                setattr(self, 'prompt', messages)

            # other business logics...
            
            logger.info(f"Got parameters successfully")

        except ClientError as error:
            raise error

settings = Settings()
```
    


이상함을 느낀 계기
-
- 나는 원래 타고나기를 뭔가 이상함이나 찜찜함을 잘 느낀다. 
- [Open AI](https://github.com/openai/chatgpt-retrieval-plugin) 코드를 직접 써보고 필요한 기능을 붙여보고 있었는데, 내가 짠 모듈이 fastapi 서버 시작보다 앞서있었다.
- 자세히 보니 위 레포에서 datastore 를 가져오는 방식이 굉장히 우아해 보였다.
  - 핵심로직 객체와 별개로 factory 파일을 만들고, fastapi startup 이벤트에 해당 factory 함수를 콜하고 있었다.

    

만들어진 것에 대한 GPT 4의 평가
-
- 한줄요약 : 끔찍하다.
```
The current design pattern for this code can be considered as a Monolithic design pattern. In such an architecture, all the codes and components that make up an application/project are interconnected and interdependent. But for the current needs, it seems like the Singleton design pattern in combination with the Observer pattern will be better suited.

In the project, settings.py is crucial as it loads in configurations from Amazon SSM service and updates them on a periodic basis. This pattern/drivers to update the settings are all tied in a unique monolithic entity - Settings class. This pattern has several disadvantages:

1) The Settings instance is reloaded across all the modules, but the pattern does not provide a clear way to notify or propagate these changes where the settings are being used.

2) Since all loading/configurations are housed in the instance of the Settings Class, it is a single point of failure.

3) Tracking issues becomes a nightmare as all functionalities are interdependent and tightly coupled.

4) Fresh configurations made will not be accessible to other instances until the 'update' method is called.
```


To address these issues, let's try to adopt the Singleton-Observable pattern combo:

1) A Singleton class can be used to create a global Settings object that can be accessed from anywhere in the application. The pattern ensures a class has one instance and provides a global point of access to it, promoting consistency of values across different modules.

2) Instead of manually calling settings.update() in every periodic task, implement an Observer pattern, and make the Settings class an Observable. This way, whenever the settings are changed, all observers (classes that utilize these settings) are notified and can react accordingly.

3) The Observable can automatically schedule the refresh of the settings in a separate thread, so the refresh process will not interfere with the request handling.

4) A singleton Settings instance that is both thread-safe and refreshing itself in the background would make the configuration values available across the application.
```

