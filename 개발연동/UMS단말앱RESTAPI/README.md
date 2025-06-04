# UMS 단말앱 REST API

## 목차

- [단말앱 UMS 속성 연동 팁](#단말앱-UMS-속성-연동-팁)
- [래셔널 UMS 메시지발신 클라이언트 등록](#래셔널-UMS-메시지발신-클라이언트-등록)
- [메시지 발신](#메시지-발신)
- [메시지 전달 콜백 설정](#메시지-전달-콜백-설정)
- [메시지 전달상태 변경 실시간 콜백 호출](#메시지-전달상태-변경-실시간-콜백-호출)


## 단말앱 UMS 속성 연동 팁
- UMS샘플앱 가이드를 통해 속성으로 연동 개발시 아래를 참조한다.
    - [Hello UMS 샘플앱 가이드(Android)](https://github.com/RationalOwl/rationalowl-sample/tree/master/device-app/android/Java/helloUms)을 참조한다.
    - [Hello UMS 샘플앱 가이드(IOS)](https://github.com/RationalOwl/rationalowl-sample/tree/master/device-app/ios-swift/helloUms)을 참조한다.
    - 샘플앱 가이드는 아래 단말앱 UMS REST API 호출을 통해 연동 개발하는 과정을 실행가능한 샘플소스로 보여준다.
    - installApp, unregUser 2개의 REST API는 연동에 필수
    - 나머지 REST API는 편의성을 위해 제공하는 API로 옵션


## UMS단말앱 등록
앱 최초 구동시 혹은 앱 사용자 가입시 호출하여 UMS 앱 등록한다.

 - Method: post
 - url: https://myUmsServer:36001/pushApp/installApp
 - post parameter
 
```java
{
    "cId":303,  
    "aId":"myAccountId",
    "dt":1,
    "dRId":"myDeviceId",
    "pn":"0101111111",
    "auId":"my app user id",
    "n":"홍길동"
}
```

 - post parameter 설명
    - cId : [필수](command id) 303 고정값 입력
    - aId : [필수](account id) 앱 서비스에 부여되는 아이디
    - dt : [필수](device type) 1: Android, 2: IOS
    - dRId : [필수](device registration id) 단말 아이디
    - pn : [옵션](phone number) 
    - auId : [옵션](app user id)
    - n : [옵션](app user name)

- response
 
```java
{
   "cId":303,  
    "aId":"myAccountId",   
    "rc":1,
    "cmt":"성공",
    "usRid":"my app user id"
}
```
 - response parameter 설명
   - cId : [필수](command id) 303 고정값 입력
    - aId : [필수](account id) 앱 서비스에 부여되는 아이디
    - rc : [필수](result code) 1: 성공, 그외 실패
    - cmt : [옵션](comment) api 호출 결과 설명
    - usRid : [옵션](UMS server registration id) UMS 서버 아이디 



## UMS단말앱 탈퇴
UMS 앱 사용을 중지한다.(앱탈퇴)

 - Method: post
 - url: https://myUmsServer:36001/pushApp/unregUser
 - post parameter
 
```java
{
    "cId":305,  
    "aId":"myAccountId",
    "dRId":"myDeviceId"
}
```

 - post parameter 설명
    - cId : [필수](command id) 305 고정값 입력
    - aId : [필수](account id) 앱 서비스에 부여되는 아이디
    - dRId : [필수](device registration id) 단말 아이디

- response
 
```java
{
   "cId":305,  
    "aId":"myAccountId",    
    "dRId":"myDeviceId",
    "rc":1,
    "cmt":"성공"
}
```
 - response parameter 설명
   - cId : [필수](command id) 305 고정값 입력
    - aId : [필수](account id) 앱 서비스에 부여되는 아이디
    - dRId : [필수](device registration id) 단말 아이디
    - rc : [필수](result code) 1: 성공, 그외 실패
    - cmt : [옵션](comment) api 호출 결과 설명



## UMS단말앱 인증번호 요청
앱 등록시 폰번호 입력할 경우 폰번호 검증을 위해 인증번호를 요청한다.
해당 API호출후 폰으로 인증번호가 SMS문자를 통해 날아오게 된다.
앱 가입시 사용자가 폰번호를 잘못 기입하는 확률이 최소 1프로 이상으로 폰번호를 통한 문자, 알림톡 통합 이용시 반드시 본 API를 통해 정확한 폰번호 입력을 권고한다.

 - Method: post
 - url: https://myUmsServer:36001/pushApp/reqAuthNumber
 - post parameter
 
```java
{
    "cId":301,  
    "aId":"myAccountId",
    "cc":"82",
    "pn":"0101111111"
}
```

 - post parameter 설명
    - cId : [필수](command id) 301 고정값 입력
    - aId : [필수](account id) 앱 서비스에 부여되는 아이디
    - cc: [필수](country code) 전화번호 국가코드(한국 82)
    - pn : [필수](phone number) 

- response
 
```java
{
   "cId":301,  
    "aId":"myAccountId",
    "rc":1,
    "cmt":"성공"
}
```
 - response parameter 설명
   - cId : [필수](command id) 301 고정값 입력
    - aId : [필수](account id) 앱 서비스에 부여되는 아이디
    - rc : [필수](result code) 1: 성공, 그외 실패
    - cmt : [옵션](comment) api 호출 결과 설명




## UMS단말앱 인증번호 검증
reqAuthNumber API 호출결과 SMS로 전달받은 인증번호를 검증한다.

 - Method: post
 - url: https://myUmsServer:36001/pushApp/verifyAuthNumber
 - post parameter
 
```java
{
    "cId":302,  
    "aId":"myAccountId",
    "dt":1,
    "pn":"0101111111",
    "an":"1111"
}
```

 - post parameter 설명
    - cId : [필수](command id) 302 고정값 입력
    - aId : [필수](account id) 앱 서비스에 부여되는 아이디
    - dt : [필수](device type) 1: Android, 2: IOS
    - pn : [옵션](phone number) SMS로 전달받은 인증번호
    - an : [옵션](app user id) 

- response
 
```java
{
   "cId":302,  
    "aId":"myAccountId",   
    "rc":1,
    "cmt":"성공"
}
```
 - response parameter 설명
   - cId : [필수](command id) 302 고정값 입력
    - aId : [필수](account id) 앱 서비스에 부여되는 아이디
    - rc : [필수](result code) 1: 성공, 그외 실패
    - cmt : [옵션](comment) api 호출 결과 설명




## 메시지 수신확인 사실 전달
사용자 수신확인시 수신확인했음을 래셔널UMS솔루션에 알려준다.
해당 API호출로 래셔널UMS솔루션은 실시간 모니터링, 발신내역, 통계 등에 수신확인 정보를 실시간 제공한다.

 - Method: post
 - url: https://myUmsServer:36001/pushApp/notiRead
 - post parameter
 
```java
{
    "cId":311,  
    "aId":"myAccountId",
    "dRId":"myDeviceId",
    "mId":"msg id"
}
```

 - post parameter 설명
    - cId : [필수](command id) 311 고정값 입력
    - aId : [필수](account id) 앱 서비스에 부여되는 아이디
    - dRId : [필수](device registration id) 단말 아이디
    - mId : [필수](message id) 메시지 구분자

- response
 
```java
no response
```
 


## 메시지 전달 상태 정보 확인
알림톡/문자 메시지 전달 정보를 요청한다.

 - Method: post
 - url: https://myUmsServer:36001/pushApp/msgInfo
 - post parameter
 
```java
{
    "cId":312,  
    "aId":"myAccountId",
    "dRId":"my app user id",
    "mId":"msg id"
}
```

 - post parameter 설명
    - cId : [필수](command id) 312 고정값 입력
    - aId : [필수](account id) 앱 서비스에 부여되는 아이디
    - dRId : [필수](device registration id) 단말 아이디
    - mId : [필수](message id) 메시지 구분자

- response
 
```java
{
    "cId":312,  
    "aId":"myAccountId",   
    "rc":1,
    "cmt":"성공",
    "ast":111111,
    "ase":1,
    "mst":111111,
    "mt":1,
    "ms":1

}
```
 - response parameter 설명
   - cId : [필수](command id) 312 고정값 입력
    - aId : [필수](account id) 앱 서비스에 부여되는 아이디
    - rc : [필수](result code) 1: 성공, 그외 실패
    - cmt : [옵션] (comment) api 호출 결과 설명

    - ase : [필수](alimtalk state) 알림톡 전달상태 0: 미발신, 1: 발신요청, 2: 전달성공, 3: 전달실패
    - ast : [옵션](alimtalk send time) 알림톡 발신 시간
    - ms : [필수](munja state) 문자 전달상태 0: 미발신, 1: 발신요청, 2: 전달성공, 3: 전달실패
    - mst : [옵션](munja send time) 문자 발신 시간
    - mt : [옵션](munja type) 문자타입 12: sms, 13: lms, 14: mms
    



## 메시지 이미지 데이터 
이미지를 첨부한 알림 발신시 푸시알림에 ii 필드에 이미지 아이디가 세팅되어 오는데 해당 이미지 아이디로 이미지 데이터를 요청한다.

 - Method: post
 - url: https://myUmsServer:36001/pushApp/imgData
 - post parameter
 
```java
{
    "cId":313,  
    "aId":"myAccountId",
    "mId":"msg id",    
    "iId":"image id"
}
```

 - post parameter 설명
    - cId : [필수](command id) 312 고정값 입력
    - aId : [필수](account id) 앱 서비스에 부여되는 아이디    
    - mId : [필수](message id) 메시지 전달 정보를 알고자 하는 메시지 구분자
    - iId : [필수](image id) 메시지 전달 정보를 알고자 하는 메시지 구분자


- response
 
```java
{
   "cId":313,  
    "aId":"myAccountId",   
    "rc":1,
    "cmt":"성공",
    "imgD":"base64 encoded image data"
}
```
 - response parameter 설명
   - cId : [필수](command id) 303 고정값 입력
    - aId : [필수](account id) 앱 서비스에 부여되는 아이디
    - rc : [필수](result code) 1: 성공, 그외 실패
    - cmt : [옵션](comment) api 호출 결과 설명
    - imgD : [필수](image data) base64 인코딩된 이미지 데이터
