---
layout: post
title: WebGoat-General 설명과 풀이
---


*WebGoat의 General 메뉴에 대한 설명과 풀이를 다룬다.*
*이번 메뉴에선 HTTP의 기본, HTTP Proxy, Chrome 개발자 도구와 Confidentiality/Integrity/Availability 3형식의 문제가 제공되고 있다.*



# 1. 설명



이번 메뉴에선 HTTP의 기본, HTTP Proxy, Chrome 개발자 도구와 Confidentiality/Integrity/Availability 3형식의 문제가 제공되고 있다.

우선 간단히 HTTP를 설명한다. 



## - HTTP 동작 원리

- **작업의 최소단위는 Transaction**
- 1회의 Transaction은 Client의 Request 부터 Server의 Response 까지.

- Client의 Request / Server의 Response의 이하 세 부분으로 구성

1. **Request / Response (Start 또는 Request(Client 입장) 또는 Status(Server 입장) line)**
2. **Header**
3. **Body**

* 기본적으로 GET Method 는 Request Line에 데이터가 포함되고, POST Method는 Body 부분에 데이터가 포함.

![img](https://www.programmersought.com/images/677/93df016d213e2419d5738192613b428d.png)

![img](https://www.programmersought.com/images/785/701272696c7b9f40c8d78771d3153ff9.png)

*이미지 출처 : https://www.programmersought.com/article/93773742856/





> 그리고 General의 문제들 풀이를 위해 Burp Suite가 사용된다. 

Bup Suite 는 Proxy 서버를 열어 Client와 Server 사이 통신 시 이 통신을 도청해 패킷을 조작하는 중간자 공격이 가능한 툴이다. 

```http
Proxy는 중간자 (Man-in-the-middle) 입장에서 Message를 중계한다.

Client로부터 요청을 받아 대신 전달하며 그 내용을 기록해둔다. 그렇기 때문에 차단된 콘텐츠나 사이트에 Proxy를 사용해 접근이 가능한 것이다. 하지만 기록 때문에 반드시 사용에 주의를 요한다.

뿐만 아니라 Proxy는 Request와 Response의 내용을 변경해 송신 할 수도 있다.
```



다운로드는 https://portswigger.net/burp  이곳에서 Community (무료버전)을 받으면 된다.

그 후에 우선 버프슈트 내의 프록시를 설명해준다.

간단하게 **Options** 에서 프록시와 포트번호 설정이 가능하다.

![IMAGE 001](https://user-images.githubusercontent.com/52769104/103628848-e6b69400-4f82-11eb-8511-b3ab819b12b7.png)



# 2. General - 문제들

## - HTTP Basics

해당 문제는 통신 형식이 POST 형식인지, GET 형식인지를 맞추며 패킷안에 있는 magic number 를 맞추는 문제이다.

Wireshark 등의 패킷캡쳐 툴로 해당문제를 풀이해도 되지만 다음 Proxy 문제와의 연관성으로 인해 Burp Shite로 해당 문제를 해결한다.

![IMAGE 003](https://user-images.githubusercontent.com/52769104/103628871-ee763880-4f82-11eb-9e07-1cca65ab85b6.png)



 **풀이 방법은 다음과 같다.**

```
1. Burp Suite가 켜진 상태로 첫번째 Textbox에 임의의 글자를 입력한다. POST, GET, adjifjaoif 등등
2. Burp Shite의 Proxy -> HTTP history를 확인한다.
3. 해당 부분에서 URL 부분이 /WebGoat/HttpBasics/attact2 인 패킷의 내용에서 정답을 확인한다.
4. WebGoat로 돌아와 Textbox를 채워 정답을 제출한다. (Go! 버튼 클릭)
```



해당 패킷에는 다른 패킷들과 다르게 **Params 부분에 체크표시가 되어 있다.** 

**이것은 서버에 전송하는 별도의 데이터가 있다는 의미이다.** 
**Form에 값을 입력하여 메시지를 전송할 때도 물론 발생한다.**

![IMAGE 002](https://user-images.githubusercontent.com/52769104/103628897-f7ffa080-4f82-11eb-8e00-6f6854926703.png)





## - HTTP Proxies

이번 문제의 설명은 길다. 

![IMAGE 007](https://user-images.githubusercontent.com/52769104/103628915-fdf58180-4f82-11eb-8306-dbf6cf198455.png)

![IMAGE 014](https://user-images.githubusercontent.com/52769104/103628938-051c8f80-4f83-11eb-9752-f73fdfd9f47e.png)



해당 문제에서 패킷 변조공격을 직접적으로 사용해 풀이해야 하므로, 이에 대한 설명과 이를위해 [OWASP ZAP](https://owasp.org/www-project-zap/) (Zed Attack Proxy)라는 툴을 사용하기 위한 설명들인데, Burp Suite로 대체가 가능하다.

또한 이번 단계를 위해 아주 조금의 준비 단계가 필요하다.

- [ ] **Windows 자체나 IE에서 Proxy 사용에 체크**
- [ ] **Burp Suite의 Proxy -> Intercept 에서 Intercept is on 으로 설정**
- [ ] **Burp Suite의 Proxy -> Intercept 에서 Open Browser 사용 (Chromium이 열림. 여기서 WebGoat 접속)**
- [ ] **Burp Suite의 Proxy -> Options 에서 Intercept Client Requests를 설정(옵션, 하단 사진 참조)**

마지막 옵션 사항은 컨텐츠에 대한 파라미터 값이 반드시 포함된 (And) 것만 가로채기(Intercept)한다는 의미이다. 이 옵션을 설정하지 않으면 여러가지 패킷이 버프슈트에 한번에 들어오는 경험을 하게 된다.



![IMAGE 013](https://user-images.githubusercontent.com/52769104/103628968-0f3e8e00-4f83-11eb-87b0-7be0e8729b98.png)



 **풀이 방법은 다음과 같다.**

```
1. Burp Suite의 위 설정이 완료된 상태에서 해당 페이지의 Textbox 옆 Submit을 클릭
2. Burp Suite의 Proxy -> Intercept 에서 도청된 패킷을 확인
3. 해당 패킷을 문제 요구사항에 맞게 입력
4. 이후 Burp Suite의 동일 화면에서 Forward 버튼으로 변조된 패킷을 웹브라우저에 전송
```

### 패킷 변조 전 사진

![IMAGE 004](https://user-images.githubusercontent.com/52769104/103629001-19f92300-4f83-11eb-8bc1-1fe9688c73d3.png)

### 패킷 변조 후 사진

(오타가 있다. ~~changeMe=doesnt+matter+really~~ 가 아니라
                       **changeMe=Requests+are+tampered+easily** 이다. 요청들이 쉽게 탬퍼(변조)됐다.)

![IMAGE 005](https://user-images.githubusercontent.com/52769104/103629003-1b2a5000-4f83-11eb-9402-c8f855502750.png)

### 요구사항에 맞게 변조 후 Forward 완료 화면 

### ![IMAGE 006](https://user-images.githubusercontent.com/52769104/103629007-1bc2e680-4f83-11eb-980b-2752b15bdb28.png)





## - Google Chrome Developer Tools (크롬 개발자 도구)

이제는 대중화 된 Chrome Web Browser의 개발자도구를 사용해 보는 문제들이다.

너무 간단한 문제들이라 길게 다루지 않는다.

**단축키는** **F12** 이다.



**풀이 방법은 다음과 같다.**

```
1. Chrome에서 F12 후 Console 탭에 문제 내용 입력 후 나오는 임의의 숫자를 Textbox에 입력 후 Submit
2. Chrome에서 F12 후 Network 탭 클릭 후 WebGoat에서 요청 버튼 후 패킷에서 networkNum: 찾아 Textbox에 입력 후 Submit
```

![IMAGE 009](https://user-images.githubusercontent.com/52769104/103629059-2e3d2000-4f83-11eb-8f9e-d46eec8a661b.png)

![IMAGE 010](https://user-images.githubusercontent.com/52769104/103629064-2ed5b680-4f83-11eb-9a89-99958ea88307.png)





## - CIA Triad (기밀성, 무결성, 가용성 3형식)

**CIA Triad (기밀성, 무결성, 가용성)는 정보 보안의 가장 중요한 구성요소이자 기본 표준이다.**  
그리고 모든 보안 시스템에서 보장되어야 한다.
이 세 가지 요소 중 하나라도 위반 할 수 있는 경우 심각한 결과를 초래할 수 있다.

이에 대한 개념을 간단히 설명하고, 문제를 풀어보는 페이지이다.

![IMAGE 012](https://user-images.githubusercontent.com/52769104/103629085-385f1e80-4f83-11eb-89b3-7cff553d99af.png)

 

### 여기까지 잘 진행했으면 General 메뉴가 초록색 체크박스로 채워져 있을 것이다.
