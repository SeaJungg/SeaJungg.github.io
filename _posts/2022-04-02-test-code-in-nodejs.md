---
layout: post
title: "Node JS 환경에서의 API 테스트 코드 작성 방법"
category: done
---

### 현상 파악

> api 테스트의 시간적 비용이 너무 크다.
- api의 기본적인 I/O 맞추는 게 의외로 너무너무 오래 걸렸다.고작 오타 하나, NULL처리 미숙으로 인해, 열다섯 개쯤 되는 필수값을 프론트 화면에서 일일이 넣어서 한 번 더 post request를 날려야만 테스트가 된다.<br>(등록-수정-put요청-앗!-등록...)
- DB데이터 재정리 시간이 너무 길다. DB를 정리하느니 그냥 처음부터 하는 게 빠를 정도(너무나도 귀찮아진다.) DB에 의존하는 CRUD api인데 자동 롤백 기능을 만들지 않았었다.

### 개선 목표
- 프론트 화면 없이도, Test-case가 알아보기 쉽게 정리될 수 있는 테스트 환경 만들기.
- 기본적인 예외field 때문에 소요되는 프론트엔드 팀과의 커뮤니케이션 시간 단축하기.(NULL, 공백, default값과 같이.. 돌발적이지만 식상한 것들.)
- api를 3개 이상 만들어야 하는 프로젝트에서, 각 api간 독립성이 보장된 테스트 환경 만들기.

### 해결 방안
- Mocha test : Node.js 의 함수 테스트 프레임워크. 인턴 후배님이 사용하셔서 알게 됨. 
- super test : Mocha 위에서 돌아가는, API 단위의 테스트. 익스프레스 서버를 구동시켜서 실제 요청 후 결과를 검증한다. 역시 내가 뭔가 필요하다 싶으면 이미 세상에 있다. 무작정 request 보내고 response를 log에서 찾던 나의 시간들 안녕..

### 적용 내용
- mocha step 사용했다. `--require mocha-steps` 하나의 API안에서 aync.waterfall로 동작하는 걸 하나하나 테스트하기 위해서.
```js
describe("database with swagger", function () {

    step ("#1-1 전자계약서 발행용 토큰 획득", (done) => {
        esignon.getToken(function(err, result) {
            if(err) {return done(err)};
            result.should.have.property('body').that.includes.all.keys(["access_token"]);
            esignonToken = result.body.access_token;
            done();
        })
    });

    step ("#1-2 전자계약서 생성", (done) => {
            console.log(esignonToken);
            console.log(contractId);
            esignon.newContract(esignonToken, contractId, config.partnerInput, (createContractError, createContractRes) => {
                if (createContractError) {
                    callback(createContractError)
                } else {
                    console.log(createContractRes)
                    let body = createContractRes.body
                    
                    if(err) {return done(err)};
                    assert.equal(result.header.result_msg, "Work Flow가 시작됩니다.");
                    result.should.have.property('body').that.includes.all.keys([
                        'memb_email',
                        'workflow_id',
                        'workflow_name',
                        'play_url',
                        'next_play_user',
                        'next_play_user_name',
                        'token'
                    ]);
                    contractInfo = {
                        contractId : contractId,
                        partnerId : partnerId,
                        workflowId : result.body.workflow_id,
                        name : result.body.workflow_name,
                        url : result.body.play_url,
                        next_play_user : result.body.next_play_user,
                        next_play_user_name : result.body.next_play_user_name
                    }
                    done();
                }
            })
    });

    ...
});
```

### 결과
- Test-case를 알아보기 쉽게 하기, 성공. Message Queue 기반으로 돌아가는 우리 회사 스택 특성상 input 이 뭐였는지는 구전되는 수준이었다. mocha 파일이 있으니 인수인계 할 때도 편할 듯.
- I/O 맞추기의 편의성 도모, 반쯤만 성공. 처음 설계에서 사소한 변수명들이 자주 바뀌는 환경을 고려하지 못했다. 수정요청이 너무 자주 생기는 우리 회사 문화(..) 특성상 IN/OUT이 자주 바뀌어서, 실제 코드 바꾸고, 프론트 바꾸고, 연결된 애들 바꾸고, 테스트코드 바꾸고, 하다보니 일부 누락되는 곳이 많았다.
- API간 독립성 보장하기, 실패.. 이번 프로젝트는 하나의 API안에서 aync.waterfall로 동작하는 프로젝트였다. 이런 입장에서 보면 내 test code는 '과도한 모듈화'였다. 그래서 API 테스트에는 생각보다 별로 못써먹었다.
- 설계단에서 '이 부분은 모듈화해서 진행하자!'고 확실히 결정난 부분에서는 엄청 잘 써먹었다. 메일을 찾고 보내는 모듈 같은..

### 느낀점
- 타사 API Wrapping 많이 하는 플젝에선 좋을 듯?
- 모듈화를 잘 하자. 애초에 코드를 단순하게 짜면 테스트도 쉽다.
- input 기준에서의 테스트 케이스를 먼저 잘 정의하자. 이번 프로젝트에서 병목은, 생각지도 못한 에러들이 밤새 나왔다는 점이다. 그런데 그게 input의 변화나 node_module 버전 문제가 대부분이었다.
- '의도한' 에러를 잘 발생시키는 서버를 만드는 게 API테스트코드를 짜는 것보다 생산성 향상에 도움이 될 것 같다..

