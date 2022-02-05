---
layout: single
title: WebGoat (A5) Broken Access Control Solution 1
categories: InfoSecurity
tag: [blog, infosecurity, webgoat]
toc: true
---



*WebGoat의 (A5) Broken Access Control 메뉴의  Insecure Direct Object References에 대한 설명과 풀이를 다룬다.*





# 1. 설명



이번 장은 액세스 컨트롤 중  Insecure Direct Object References (안전하지 않은 직접 개체 참조) 를 다룬다.



직접 개체 참조란 쉽게 얘기해서, Get 방식을 예로 들 수 있다.

```
https://some.company.tld/dor?id=12345
https://some.company.tld/images?img=12345
```

**dor?id=12345,  images?img=12345** 이러한 부분과 같이 클라이언트에서 입력을 사용해 Data 및 Object에 직접 엑세스 할 수 있는 것이다.



HTTP Get Method만 예로 들었지만, **이외에도 흔히 볼 수 있는 Post, Put, Delete 방식**도 Payload 부분이 **잠재적으로 취약**하다는 것은 다르지 않다.



직접 개체 참조는 뿐만 아니라, 우리 주변에서 쉽게 볼 수 있는 URL로도 접속 확인이 가능하다.

```
https://some.company.tld/app/user/23398
https://some.company.tld/app/user/23399
...
https://some.company.tld/app/user/24500
```



**이런식으로 23398, 23399, ... 24500 부분만 바꿔서 접속이 가능하다.**

실제로 기밀사항이 많은 공모전 제출 파일에도 이렇게 타인의 출품 작품과 정보를 엿볼 수 있는 문제가 있다.



이렇게 **안전하지 않은** 직접 개체 참조 (**Insecure** Direct Object References) 는 데이터를 임의로 조작할 수도 있다.



문제 풀이에 앞서, 아래 소개하는 페이지들은 안전하지 않은 직접 개체 참조를 다루는 자료들이다.

https://owasp.org/www-project-top-ten/2017/A5_2017-Broken_Access_Control.html

https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html

http://cwe.mitre.org/data/definitions/639.html



# 2. (A5) Broken Access Control - 문제들 1

## - Insecure Direct Object References



### 첫 번째 문제, 단순 로그인 이다.

이후 문제풀이를 위한 준비.

문제 본문에 설명된 아이디와 패스워드를 입력해 Submit 한다.



![IMAGE 001](https://user-images.githubusercontent.com/52769104/104419799-53054900-55bc-11eb-87a0-c20dd044cf53.png)



참고로

> Authentication (인증) 은 주체를 시스템에 증명하는 과정이고,
>
> Authorization (인가) 는 시스템이 주체를 승인하고 권한을 부여하는 과정이다.



![IMAGE 002](https://user-images.githubusercontent.com/52769104/104419796-51d41c00-55bc-11eb-904c-46de61f20a8f.png)









### 두 번째 문제, Response Payload의 값을 입력하는 것이다.

Request에는 담기지 않은 내용의 Response Payload를 정답란에 2가지 입력하는 것이다.



![IMAGE 003](https://user-images.githubusercontent.com/52769104/104419814-5b5d8400-55bc-11eb-9f32-47e089cf4b50.png)



하지만, 본 문제 풀이 시스템에서는 'View Profile' 버튼 클릭시에도 Not found 페이지 404 error가 뜨고 있다.

![IMAGE 004](https://user-images.githubusercontent.com/52769104/104419830-60223800-55bc-11eb-9d7a-501f8859fd49.png)



이에 상관없이 풀이방법을 말하자면......



 **풀이 방법은 다음과 같다.**

```
1. Chrome 개발자 도구를 꺼낸다 (F12)
2. 개발자 도구의 Network 탭으로 이동한다.
3. 문제에서 'View Profile'을 누른다.
4. Response 값을 확인 후, Request 에는 없는 속성 두 가지를 입력한다.
   (role, userid)
```



![IMAGE 006](https://user-images.githubusercontent.com/52769104/104419838-66b0af80-55bc-11eb-9a89-832fa5667ea6.png)







### 세 번째 문제, 직접개체참조방식으로 Profile에 접근하라



두 번째 문제에서 확인한 Payload 값으로 진행하면 쉽게 풀수 있는 문제이다.

Tom의 프로필을 조회하면 된다.

**이 문제에 나오는 'RESTful 패턴' 이란, *REST(REpresentational State Transfer) 설계방식(아키텍처) 기반의 소프트웨어 구현시에 활용되는 패턴 (가이드사항) 이다.***



![IMAGE 007](https://user-images.githubusercontent.com/52769104/104419857-6dd7bd80-55bc-11eb-8d7a-74bf691a08c8.png)



 **풀이 방법은 다음과 같다.**

```
1. 두 번째 문제의 Response Payload에서 Profile의 접속 경로/userid 를 입력한다.
```





![IMAGE 008](https://user-images.githubusercontent.com/52769104/104419861-6e705400-55bc-11eb-9b9d-60f32ec7f8f6.png)







### 네 번째 문제, 직접개체참조방식으로 다른 사람의 Profile에 접근하고, 수정해라



![IMAGE 010](https://user-images.githubusercontent.com/52769104/104419889-77612580-55bc-11eb-92b6-ff86da6b83b7.png)



세 번째 문제에서 확인한 형식을 GET 형식으로 Browser의 주소창에 입력해 접근한다. 아주 약간의 Brute Force 형식이다.



![IMAGE 011](https://user-images.githubusercontent.com/52769104/104419911-7e883380-55bc-11eb-8263-cc4acb846367.png)



![IMAGE 012](https://user-images.githubusercontent.com/52769104/104419917-7f20ca00-55bc-11eb-917c-86bfce25c09b.png)



![IMAGE 013](https://user-images.githubusercontent.com/52769104/104419918-7fb96080-55bc-11eb-8bf9-cfda226fe008.png)



실행 시 다음과 같이 '2342388'에서 Buffalo Bill 의 Profile을 확인할 수 있다.



그리고 수정하는 방법은 언제나 그렇듯 Burp Suite의 Intercept나 Repeater를 사용한다.



 **풀이 방법은 다음과 같다.**

```
1. 발견한 Buffalo Bill의 Profile 정보를 Send to Repeater 한다.
2. Method를 'PUT'으로, Content-Type을 JSON으로 추가, Payload 부분에 Buffalo Bill의 변경시킬 신상을 적는다. ('='가 ':' 로 변경됨에 주의)
3. Send를 눌러 Response 값을 확인한다.
```



![IMAGE 014](https://user-images.githubusercontent.com/52769104/104419973-91026d00-55bc-11eb-8dea-f32ad94b2757.png)

추가적으로, 수정 과정에서 Request에 Content-Type를 적지 않으면,  [`415 Unsupported Media Type`] **요청한 미디어 포맷은 서버에서 지원하지 않습니다, 서버는 해당 요청을 거절할 것입니다.**  라는 상태메시지가 뜨며 변경이 되지 않는다.

![IMAGE 015](https://user-images.githubusercontent.com/52769104/104419995-98c21180-55bc-11eb-9ec8-db83b84fa909.png)



보다 자세한 HTTP 상태코드는 다음 사이트 방문을 추천한다.

https://developer.mozilla.org/ko/docs/Web/HTTP/Status



![IMAGE 016](https://user-images.githubusercontent.com/52769104/104420000-99f33e80-55bc-11eb-9bf0-a77079d2522a.png)

![IMAGE 017](https://user-images.githubusercontent.com/52769104/104420003-99f33e80-55bc-11eb-9ff4-e20eef873381.png)



추가적으로 XXE Mitigation에 대한 방안은 다음과 같다.

- DTD를 사용하지 않는다.
- 만약 사용이 꼭 필요하다면, External Entities만 사용토록 해야 한다.
- 검증 시 Client가 Header를 조작하는 것을 검출하기 위해 406(Not Acceptable) 오류를 띄우도록 해라.



더 자세한 XXE 예방책은 아래 사이트를 방문하면 좋다.

https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html



**좋은 Secure Object References 방법**은

- 사용자와 관리자 별로 권한 구분 하여 HTTP Method 사용.

![IMAGE 018](https://user-images.githubusercontent.com/52769104/104420005-9a8bd500-55bc-11eb-85d1-d483fb870a87.png)

- Access Control 시 Logging으로 감사 구현
- 해시값 등을 사용한 Indirect References 사용
- JSON Web Token (JWT) 등의 API를 사용

등을 권장하고 있다.





## 여기까지 잘 진행했으면 (A5) Insecure Direct Object References의  Insecure Direct Object References 메뉴가 초록색 체크박스로 채워져 있을 것이다.



![IMAGE 019](https://user-images.githubusercontent.com/52769104/104420006-9a8bd500-55bc-11eb-9669-9eb5ba1b810e.png)
