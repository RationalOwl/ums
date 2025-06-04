# UMS 메시지 발신 REST API

## 목차

- [래셔널아울 연동 등록](#래셔널아울-연동-등록)
- [래셔널 UMS 메시지발신 클라이언트 등록](#래셔널-UMS-메시지발신-클라이언트-등록)
- [메시지 발신](#메시지-발신)
- [메시지 전달 콜백 설정](#메시지-전달-콜백-설정)
- [메시지 전달상태 변경 실시간 콜백 호출](#메시지-전달상태-변경-실시간-콜백-호출)


## 래셔널아울 연동 등록

- 래셔널 UMS는 모 솔루션인 래셔널아울 API 솔루션을 통해 개발되었다.
- 래셔널 UMS 메시지 발신 연동을 위해서 먼저 래셔널아울 솔루션 앱서버로 등록한다.

### REST API

 - Method: post
 - url: https://래셔널아울서버/server/register/
 - post parameter
 
```java
{
    "serviceId":"service id here",  
    "registerName":"displayed app server name",
    "privateNetwork":1
}
```

 - post parameter 설명
    - serviceId : (service id) 래셔널아울 솔루션에 등록한 서비스 아이디
    - registerName : (register name) 관리자 웹에 표시될 이름
    - privateNetwork : (private network) 래셔널아울 서버와 같은 사설망에 존재시 1로 세팅

- response
 
```java
{
  "serverRegId":"asdfkjkjlasfdasfd"
}
```
 - response parameter 설명
    - serverRegId : (server registration id) 래셔널아울 앱서버 등록 아이디
        - 래셔널아울에서 앱서버에 발급한 아이디
        - 발급받은 아이디를 저장/관리해야 한다.


## 래셔널 UMS 메시지발신 클라이언트 등록

- 래셔널아울 UMS 기본 제공 웹 관리자 화면에서 메시지 발신이 가능하지만 개발 연동을 통한 발신도 지원한다.
- 개발 연동을 통해 메시지 발신하기 위해서는 메시지발신 클라이언트 등록 API를 호출하면 된다.


### REST API

 - Method: post
 - url: https://myUmsServer/msgClient/register
 - post parameter
 
```java
{
    "cId":1,
    "r":1,
    "t":12,
    "sId":"myRationalOwlServiceId",
    "rId":"myRationalOwlRegId"        
    "aId":"myAccountId",
    "ip":"myIp",
    "mcn":"myMsgClientName"
}
```
 - post parameter 설명
    - cId : (command id) 1로 고정
    - r : (request) 1로 고정
    - t : (type) 12로 고정
    - sId : (service id) 래셔널아울 서비스 아이디
    - rId : (Registration id) 래셔널아울 앱서버 등록 아이디
    - aId : (account id) 래셔널UMS 계정 아이디
    - ip : (ip) 메시지 발신 클라이언트 등록 요청하는 서버의 ip
    - mcn : (message client name) 메시지 발신 클라이언트 구분할 수 있는 이름


- response
 
```java
{
    "rc":1,
    "srId":"myMessageClientId",
    "cmt":"정상처리"
}
```
 - response parameter 설명
    - rc : (result code) 결과값
        - 1: 성공, -102: 기등록된 상태에서 api 호출시
        - 등록 성공시 웹 관리자 화면에서 모니터링 지원 시작
    - srId : (msg client id) 등록 요청한 클라이언트에게 부여된 아이디
        - rc 가 1이거나 -102일 경우 세팅됨
    - cmt : (comment) api 호출결과 부연 설명

## 메시지 발신(전화번호 기반)

- 메시지 발신 클라이언트로 등록되면 REST API호출을 통해 메시지 발신이 가능하다.
- 카톡과 같이 앱 가입시 사용자 폰번호 입력을 받아 앱 사용자의 전화번호를 알 경우 이용
- 푸시알림, 문자, 알림톡의 통합메시지 발신에 적합한 방식


### REST API

 - Method: post
 - url: https://myUmsServer/msgClient/msgClientSendMsgByPhoneNum
 - post parameter
 
```java
{
    "rId":"myRationalOwlRegId"        
    "aId":"myAccountId",
    "ip":"myIp",
    "msg": {
        "title":"공지",
        "body":"공지사항입니다~"
    },
    "dest": {
        "t": 2,
        "tg" : [
            {"pn":"0101111111"},
            {"pn":"0102222222", "un":"홍길동"},
            {"pn":"0103333333"}
        ]
    }

}
```
 - post parameter 설명   
    - rId : (Registration id) 래셔널아울 앱서버 등록 아이디
    - aId : (account id) 래셔널UMS 계정 아이디
    - ip : (ip) 메시지 발신 클라이언트 등록 요청하는 서버의 ip
    - msg : message        
        - body : 메시지 내용
        - title [option]: 메시지 타이틀 
        - tId [option]: (template id) 템플릿 아이디
        - tArgv [option]: (template argument) 템플릿 매개변수
    - dest : (destination) 메시지 전달할 대상 사용자
        - t : (type)
            - 1 : 푸시앱 설치한 대상 전체 사용자
            - 2 : 멀티캐스트
        - tg [option]: (target) 대상 사용자 목록, t 가 2인 경우 세팅됨
            - pn : (phone number) 
            - un [option]: (user name)
    - img [option]: image 이미지 첨부시 세팅
        - imgN : (image name) : 이미지 이름(ex: myImg.jpg)
        - imgD : (image data) : base64 인코딩한 이미지 데이터
        - imgS : (image size) : 인코딩 되기전 원본 이미지 크기(바이트 단위)
    - rv [option]: reservation 예약발신시 설정, 미설정시 즉시발신 세팅과 동일
        - t : (type) 1(즉시발신), 2(예약발신)
        - st [option]: (send time) 발신 시간 
            - t: 2인 경우 세팅
            - yyyy/m/d h:m:s 포맷 (ex: 2023/12/25 13:10:0)
    - a [option]: alternative message 대체메시지, 푸시 미전달시 문자/알림톡 대체 발신시 설정
        - s : (set) 1(미설정), 2(설정)
        - sp : (sending phonenumber) 발신번호
        - t : (type) 11(알림톡), 12(문자 , sms/lms/mms)
        - atId [option]: (alimtalk template id) 알림톡 템플릿 아이디, t: 11 일 경우 세팅
        - kcId [option]: (kakao channel id) 카카오 채널 아이디, t: 11 일 경우 세팅
        - atm [option]: (alimtalk to munja) 1(미설정), 2(설정) 알림톡 미전달시 문자 대체 발신 설정, t: 11 일 경우 세팅

    

- response
 
```java
{
    "mId":"myMessageId"
}
```
 - response parameter 설명
    - mId : (message id) 발신요청한 메시지의 아이디
        - 추후 메시지 콜백을 통한 메시지 전달 현황을 해당 mId를 통해 알려줌
        - 메시지 전달 현황 처리를 위해서는 mId 관리가 필수

- 간단한 알림 발신 예
 
```java
{
    "rId":"myRationalOwlRegId"        
    "aId":"myAccountId",
    "ip":"myIp",
    "msg": {        
        "body":"공지사항입니다~"
    },
    "dest": {
        "t": 2,
        "tg" : [
            {"pn":"0101111111"},
            {"pn":"0102222222", "un":"홍길동"},
            {"pn":"0103333333"}
        ]
    }
}
```


- 이미지 첨부 메시지 발신 예
 
```java
{
    "rId":"myRationalOwlRegId"        
    "aId":"myAccountId",
    "ip":"myIp",
    "msg": {
        "title":"공지", 
        "body":"공지사항입니다~"
    },
    "dest": {
        "t": 2,
        "tg" : [
            {"pn":"0101111111"},
            {"pn":"0102222222", "un":"홍길동"},
            {"pn":"0103333333"}
        ]
    }
    "img": {
        "imgN": "myImg.jpg",
        "imgD" : "base64EncodedImageData............",
        "imgS":23443
    }
}
```


- 예약 발신 예
 
```java
{
    "rId":"myRationalOwlRegId"        
    "aId":"myAccountId",
    "ip":"myIp",
    "msg": {        
        "body":"공지사항입니다~"
    },
    "dest": {
        "t": 2,
        "tg" : [
            {"pn":"0101111111"},
            {"pn":"0102222222", "un":"홍길동"},
            {"pn":"0103333333"}
        ]
    },
    "rv": {
        "t":2,
        "st":"2023/12/25 13:30:0"
    }
}
```




- 푸시알림 미전달 사용자에게 문자(sms/lms/mms) 대체 발신 예
 ```java
{
    "rId":"myRationalOwlRegId"        
    "aId":"myAccountId",
    "ip":"myIp",
    "msg": {        
        "body":"공지사항입니다~"
    },
    "dest": {
        "t": 2,
        "tg" : [
            {"pn":"0101111111"},
            {"pn":"0102222222", "un":"홍길동"},
            {"pn":"0103333333"}
        ]
    },
    "a": {
        "s":2,
        "t":12,
        "sp":"01031233123"
    }
}
```


## 메시지 발신(단말등록아이디 기반)

- 보안 이슈로 전화번호 직접 사용이 불가한 경우 단말앱 설치시 발급받은 단말등록아이디 대상 메시지 발신
- 메시지 수신 대상을 전화번호 대신 단말등록 아이디로 지정하고 그외는 두 방식 모두 동일하다.
    - 전화번호 기반 대상 지정 예) "dest": { "tg" : [{"pn":"0101111111"} ]}
    - 전화번호 기반 대상 지정 예) "dest": { "tg" : [{"dId":"myDeviceId1"} ]}


### REST API

 - Method: post
 - url: https://myUmsServer/msgClient/msgClientSendMsgByDeviceId
 - post parameter
 
```java
{
    "rId":"myRationalOwlRegId"        
    "aId":"myAccountId",
    "ip":"myIp",
    "msg": {
        "title":"공지",
        "body":"공지사항입니다~"
    },
    "dest": {
        "t": 2,
        "tg" : [
            {"dId":"myDeviceId1"},
            {"dId":"myDeviceId2", "un":"홍길동"},
            {"dId":"myDeviceId3"}
        ]
    }

}
```
 - post parameter 설명
    - 아래 대상 사용자 지정 필드에서 pn(phone number) 대신 dId(device registration id)로 세팅하는 부분을 제외하고 나머지 필드는 동일
    - dest : (destination) 메시지 전달할 대상 사용자
        - t : (type)
            - 1 : 푸시앱 설치한 대상 전체 사용자
            - 2 : 멀티캐스트
        - tg [option]: (target) 대상 사용자 목록, t 가 2인 경우 세팅됨
            - dId : (device registration id) 
            - un [option]: (user name)

    

- response
 
```java
{
    "mId":"myMessageId"
}
```
 - response parameter 설명
    - mId : (message id) 발신요청한 메시지의 아이디
        - 추후 메시지 콜백을 통한 메시지 전달 현황을 해당 mId를 통해 알려줌
        - 메시지 전달 현황 처리를 위해서는 mId 관리가 필수




## 메시지 전달 콜백 설정

- 메시지 전달 콜백을 설정하면 메시지 전달 상태가 변경시 콜백이 자동 호출된다.
- 앱서버에서 메시지 전달상태 트래킹을 실시간 처리하려면 본 기능을 이용하면 된다.


### REST API

 - Method: post
 - url: https://myUmsServer/msgClient/setCallbackUrl
 - post parameter
 
```java
{
    "sId":"myRationalOwlServiceId",
    "rId":"myRationalOwlRegId"        
    "aId":"myAccountId",
    "ip":"myIp",
    "url":"myCallbackUrl"
}
```
 - post parameter 설명    
    - sId : (service id) 래셔널아울 서비스 아이디
    - rId : (Registration id) 래셔널아울 앱서버 등록 아이디
    - aId : (account id) 래셔널UMS 계정 아이디
    - ip : (ip) 메시지 발신 클라이언트 등록 요청하는 서버의 ip
    - url : 메시지 전달 콜백 url


- response
 
```java
{
    "rc":1
}
```
 - response parameter 설명
    - rc : (result code) 결과값
        - 1: 성공
        - 성공 이외 음수의 에러 코드 반환


## 메시지 전달상태 변경 실시간 콜백 호출

- 메시지 전달 콜백을 설정하면 자신이 발신한 메시지 전달 상태가 변경시 콜백이 실시간 호출된다.
- http post 메소드로 콜백이 호출된다.
- http post로 메시지 전달 상태가 변경된 메시지 목록이 json 배열 형태로 전달된다.
- 전화번호 기반 메시지 발신(sendMsgByPhoneNum API)과 단말등록아이디 기반 메시지 발신(sendMsgByDeviceId)에 따라 "pn" 필드 혹은 "dId" 필드가 세팅된다.


 
```java
전화번호 기반 발신시 콜백: post parameter
[
  
  {
     "mId":"myMessageId",   
     "ds": [       
       {
         "pn":"0102222222",
         "p": {
            "s":1,"st":1222222,"nt":2222222,"rt":3333333
          }
          "a": {
            "s":2, "st":1111111
           },
          "m": {
            "t":12, "s":2, "st":1111111
          }
        } ....
      ]
  }, ....
]
```


```java
단말등록아이디 기반 발신시 콜백: post parameter
[
  
  {
     "mId":"myMessageId",   
     "ds": [       
       {
         "dId":"myDeviceId1",
         "p": {
            "s":1,"st":1222222,"nt":2222222,"rt":3333333
          }
          "a": {
            "s":2, "st":1111111
           },
          "m": {
            "t":12, "s":2, "st":1111111
          }
        } ....
      ]
  }, ....
]
```

 - post parameter 설명    
    - mId : (message id) 메시지 전달 상태가 변경된 메시지 아이디
    - ds : (destination) 메시지 전달 상태가 변경된 대상 사용자 목록
        - pn : (phone number) 전화번호 기반 메시지 발신시 세팅됨.
        - dId : (device registration id) 단말등록아이디 기반 메시지 발신시 세팅됨.
        - p : (push state) 해당 사용자에 대한 푸시알림 전달 상태
            - s : (state) 푸시알림 전달 상태로 아래 상태 중 하나
                - 1 (예약), 2(미전달), 3(알림전달), 4(사용자 수신확인), 5(대체문자 발신), 6(미존재 단말앱 푸시미발신)
            - st : (sending time) 푸시알림 발신시간 밀리초 단위(1970/1/1 이후)
            - nt [optional]: (notification delivery time) 알림전달시간, 푸시알림이 해당 폰에 전달된 시간, 밀리초 단위
            - rt [optional]: (read time) 사용자 수신확인시간, 사용자가 푸시알림 수신확인한 시간, 밀리초 단위
        - a [optional]: (alimtalk state) 알림톡 전달 상태
            - s : (state) 알림톡 전달 상태로 아래 상태 중 하나
                - 1 (발신요청), 2(전달성공), 3(전달실패)
            - st : (sending time) 알림톡 발신시간 밀리초 단위(1970/1/1 이후)
        - m [optional]: (munja state) 문자 전달 상태
            - t : (type) 문자 종류 12(sms), 13(lms), 14(mms)
            - s : (state) 문자 전달 상태로 아래 상태 중 하나
                - 1 (발신요청), 2(전달성공), 3(전달실패)
            - st : (sending time) 문자 발신시간 밀리초 단위(1970/1/1 이후)