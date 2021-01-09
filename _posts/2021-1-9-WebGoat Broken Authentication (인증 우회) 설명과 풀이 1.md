---
layout: post
title: WebGoat Broken Authentication (인증 우회) 설명과 풀이 1
comments: true
---



*WebGoat의 (A2) Broken Authentication메뉴 중 Authentication Bypasses(인증 우회) 와 JWT (JSON Web Token)tokens 에 대한 설명과 풀이를 다룬다.*





# 1. 설명





앞서 다룬 SQL Injection 도 인증 우회 공격의 한 종류로 사용되지만, 이번에는 직접적으로 token이나 Password등 신원인증에 직접적으로 필요한 것들을 우회하고, 크랙하는 방식의 인증 파괴기법을 실습한다.

참고 사항으로 인증방식에 따른 분류는 아래와 같다.

|                                                 |                      |
| ----------------------------------------------- | -------------------- |
| Type 1 인증 (Something you know - **지식기반)** | **패스워드 PIN**     |
| Type 2 인증 (Something you have - **소유기반)** | **스마트카드, 토큰** |
| Type 3 인증 (Something you are - **존재기반)**  | **홍채, 지문**       |



# 2. (A2) Broken Authentication - 문제들 ①

## - Authentication Bypasses



인증 우회는 여러 방식으로 발생하지만 일반적으로 설정이나 논리 결함을 이용하는 경우가 많다. 또한 데이터를 도청 혹은 변조하여 성공시키는 경우도 많다. 

대표적으로 Web Browser에서 웹페이지 / DOM에 있는 숨겨진 입력을 찾아내거나 파라미터 값을 이용한 Forced Browsing 등으로 사용이 가능하다.

> 이것은 놀랍게도 **Two Factor Authentication 에서도 유효**하다 !
> SMS로 2차인증이 아닌, PW 재설정을 응용하여 사전에 정해놓은 질문에 답변해 인증을 수행하는 형태에서 가능



### 첫 번째 문제,

**Burp Suite 를 이용해 문제를 푼다.**

해당 문제는 단순하게 질문의 파라미터값을 변경해서 해결이 가능하다.

단, 실제 서비스되고 있는 사이트에서는 해당 방식이 유효하지 않을 것이다.



![IMAGE 001](https://user-images.githubusercontent.com/52769104/104091360-446b1900-52c0-11eb-8b2c-0c4256735b19.png)



 **풀이 방법은 다음과 같다.**

```
1. WebGoat의 문제에서 임의의 값을 Form에 입력 후 Submit 한다.
2. 버프슈트에서 해당 패킷을 Intercept 한다.
3. Body 값의 setQuestion0 과 setQuestion1의 숫자를 다음 숫자로 바꾼다.
4. Forward 하여 패킷을 진행시킨다.
```



![IMAGE 003](https://user-images.githubusercontent.com/52769104/104091371-56e55280-52c0-11eb-85b4-4ec444302970.png)

![IMAGE 004](https://user-images.githubusercontent.com/52769104/104091384-65336e80-52c0-11eb-9dcb-cba7842786b7.png)



![IMAGE 002](https://user-images.githubusercontent.com/52769104/104091374-5b117000-52c0-11eb-8c22-3c178af68e9c.png)





## - JWT (JSON Web Token) tokens



JWT는 공개 표준이다. (RFC 7519)

HMAC(Hash-based + Message Authentication Code) 을 이용한 디지털 서명이 포함되어 있어 검증된 정보이며, 신뢰성을 부여한다. 

기본적으로 JWT 인증은 매 작업시마다 클라이언트<->서버 간 발생한다.

**인증의 구조는 간단히 말해선 아래와 같다.**

1.  **Client가 POST에 로그인값을 Server에 전송하면, JWT (암호 키) 를 만들어 Client에 보낸다.**
2.  **Client JWT Authorization(승인(혹은 인가)) 헤더를 Server에 전송하면, 서명을 검증 후 응답값을 Client에 보낸다.**

![IMAGE 005](https://user-images.githubusercontent.com/52769104/104091497-1508dc00-52c1-11eb-9ea9-a310806ebd2c.png)



JWT의 토큰은 Base64 알고리즘으로 Encoding 된다. 

아래 그림과 같은 3가지 파트로 구성되어 있다.

![IMAGE 006](https://user-images.githubusercontent.com/52769104/104091500-1b975380-52c1-11eb-87be-ad576f0bc6bf.png)

이런 JWT 토큰이 Decoding 된 그림은 다음과 같다.

![IMAGE 007](https://user-images.githubusercontent.com/52769104/104091502-1cc88080-52c1-11eb-827c-f61bcf7cef17.png)



마지막으로, JWT 토큰은 사용자 신원과 관련 정보 전달용으로 **'Claims'이라 불리는 Payload를 포함하게 된다.** 그러므로 안전한 채널의 사용은 필수이다.



### 첫 번째 문제, 

### JWT의 서명과 인증을 위조해, admin 으로써 vote를 reset!



Guest 계정을 제외한 Tom, Jerry, Sylvester 는 투표 권한만 있다.

이 투표 결과를 초기화 시켜야 하지만, 이것은 오로지 Admin만 가능하며, 해당 메뉴에는 없다.

JWT Tokens을 조작해 admin으로 가장하여 결과를 Reset하는 것이 목표이다. 



![IMAGE 008](https://user-images.githubusercontent.com/52769104/104091508-294cd900-52c1-11eb-9396-b1c2aa538c67.png)



이 작업을 성공적으로 완수하기 위해선 아래의 준비물이 필요하다. 

이후 작업에서는 아래의 준비물이 계속 쓰일 것이다.



### 준비물 : 

### 1. Burp Suite

### 2. https://jwt.io/ (JWT Tokens <-> JSON 평문 전환 용)





 **풀이 방법은 다음과 같다.**

```html
1. 해당 문제에서 임의의 계정 선택
2. 'reset'버튼(휴지통 모양) 클릭 후 버프슈트에서 해당 패킷 확인
3. 해당 패킷을 Send to Repeater 하여 Repeater로 전송
4. access_token 부분을 JWT.io 페이지의 하단 'Encoded' 부분에 삽입
5. Decoded 부분에서 HEADER 부분의 알고리즘을 "none" 으로, PAYLOAD 부분의 admin을 "true"로 변환
6. Encoded 에서 시그니처 부분을 제외하고 복사 후, 다시 버프슈트의 해당 Request 패킷의 access_token= 부분에 붙여넣기 (기존 것을 수정)
7. 이렇게 변조된 패킷을 Send !
```





JWT Token은 **. 단위로 구분되어 있다.**

문제 해결을 위해 알고리즘을 none으로 지정했으니, 당연히 HMACSHA256 부분은 지정하지 않고 패킷을 변조하는 것이 맞다.



![IMAGE 012](https://user-images.githubusercontent.com/52769104/104091513-323daa80-52c1-11eb-968a-c075191601ba.png)





![IMAGE 014](https://user-images.githubusercontent.com/52769104/104091516-34076e00-52c1-11eb-86f9-1e9b09d37355.png)



![IMAGE 013](https://user-images.githubusercontent.com/52769104/104091514-336ed780-52c1-11eb-9b7e-bf3face6f494.png)



![IMAGE 015](https://user-images.githubusercontent.com/52769104/104091522-3e296c80-52c1-11eb-994d-674f642f5922.png)



여기까지 잘 진행했으면, **아래 사진과 같이 Response 값의 lessonCompleted 값이 'True'** 로 반환 되는 것을 확인해 볼 수 있다 !



![IMAGE 009](https://user-images.githubusercontent.com/52769104/104091532-4b465b80-52c1-11eb-9126-3bf1e5420bfa.png)







### 두 번째 문제, 

### 확보된 토큰에서 비밀 키를 찾고 새로운 토큰 만들어라! 



username이 WebGoat인 새로운 토큰을 만들어야 하는데, 기존의 토큰을 베이스로 새 토큰을 생성해야 한다.

문제에서 제공되는 토큰은 아래와 같다.

eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJXZWJHb2F0IFRva2VuIEJ1aWxkZXIiLCJhdWQiOiJ3ZWJnb2F0Lm9yZyIsImlhdCI6MTYxMDExMDg3OSwiZXhwIjoxNjEwMTEwOTM5LCJzdWIiOiJ0b21Ad2ViZ29hdC5vcmciLCJ1c2VybmFtZSI6IlRvbSIsIkVtYWlsIjoidG9tQHdlYmdvYXQub3JnIiwiUm9sZSI6WyJNYW5hZ2VyIiwiUHJvamVjdCBBZG1pbmlzdHJhdG9yIl19.japvxmvFXoODMMRaiNMWWO_RA18oed_iK9AybvCdtpE



> 이 문제는 시간이 상당히 많이 소요된다. **왜냐하면 JWT Tokens에서 VERIFY SIGNATURE를 찾아내기 위해 '사전공격'을 진행해야 하기 때문이다.**
>
> 나의 경우는 5시간 정도가 소요됐다. 여러번의 시행착오를 극복해 나갔다.
>
> John the Ripper도 사용해 보았으나, Hashcat이 더욱 빨랐다.



![IMAGE 016](https://user-images.githubusercontent.com/52769104/104091539-54cfc380-52c1-11eb-860d-1de152c3e5d8.png)



이 역시 추가적인 준비물이 필요하다.



### 준비물 : 

### 1. Hashcat (https://hashcat.net/)

### 2. OWASP Seclists (https://owasp.org/www-project-seclists/migrated_content)



 **풀이 방법은 다음과 같다.**

```html
1. WebGoat 문제 화면에 나온 JWT Token 값을 복사
 2.1. JWT.io 사이트의 Encoded 부분에 붙여넣고 대기
 2.2. JWT Token 값으로 별도에 txt 파일을 만들어 놓음
 2.2. 해당 JWT Token 값을 cmd를 열고 Hashcat과 Seclists의 적당한 파일을 사용해 사전공격 시작
3. 발견된 평문값을 JWT.io의 Decoded 부분의 VERIFY SIGNATURE -> your-256-bit-secret 부분에 입력,
4. exp(토큰 만료시간) 부분도 변환
5. 이렇게 조작된 토큰 값을 WebGoat 정답 입력폼에 입력.
```



cmd 명령어는 다음과 같다.

hashcat 1.txt -m 16500 -a 3 -w 3 ./seclists/Discovery/Web-Content/raft-large-words.txt

- 1.txt : JWT Token이 저장된 파일 
- -m 16500 : Hash mode 중 JWT Token
- -a 3 : Attack mode 중 Brute-force 모드
- -w 3 : 사용될 시스템에 부여할 Workload



명령어에 대한 더욱 자세한 설명은 아래 페이지를 방문하길 추천한다.
https://hashcat.net/wiki/doku.php?id=hashcat



참고로, 발견되는 비밀키는 사용자 환경마다 다르다.

![IMAGE 029](https://user-images.githubusercontent.com/52769104/104091544-5c8f6800-52c1-11eb-9924-b43edf407140.png)



아래는 성공한 모습이다.

![IMAGE 030](https://user-images.githubusercontent.com/52769104/104091547-62854900-52c1-11eb-91e9-761a3ff5b553.png)





### 세 번째 문제, Refreshing a Token 문제이다.



일반적으로 JWT Token은 액세스 토큰과 리프레쉬 토큰의 두 가지 유형이 있다.

액세스 토큰은 서버에 대한 API 호출을 만드는 데 사용된다. 액세스 토큰의 수명이 제한되어 있으므로 리프레쉬 토큰이 제공된다. 액세스 토큰이 더 이상 유효하지 않으면 리프레쉬 토큰을 제공하여 새 액세스 토큰을 얻기 위해 서버에 요청할 수 있다. **리프레쉬 토큰은 만료 될 수 있지만 수명이 훨씬 더 길다.** 

바로 이 부분에서 일반적으로 사용되는 Cookie 보다 더욱 취약하다고 알려져 있으며, 재생공격이 가능해진다.

이를 실습하는 부분이며, 실습에 앞서서 반드시 다음 페이지를 읽고 진행하는 것을 추천한다.

[JWT Refresh Token Manipulation](https://emtunc.org/blog/11/2017/jwt-refresh-token-manipulation/) (https://emtunc.org/blog/11/2017/jwt-refresh-token-manipulation/)



![IMAGE 032](https://user-images.githubusercontent.com/52769104/104091554-6ca74780-52c1-11eb-8e31-dca6ee3dead7.png)



 **풀이 방법은 다음과 같다.** 



> 이 문제는 WebGoat의 소스코드를 확인하는 Cheating 방법으로 Refresh Token을 알아내는 방법이 있는데, 잘 적용이 되지 않아 간단한 풀이만 싣게 되었다.

```html
1. WebGoat문제에서 'Checkout' 버튼을 클릭
2. 버프슈트로 전송된 패킷을 Send to Repeater 로 보냄.
3. Repeater에서 Request 부분의 Authorization: Bearer 이후 부분을 복사
4. JWT.io 의 Encoded 부분에 붙여넣고 Decoded 부분의 HEADER와 PAYLOAD 부분을 조작
5. 이렇게 변조된 HEADER와 PAYLOAD 부분을 다시 Authorization: Bearer 이후에 붙여넣고 Send !
```



Intercept에서 진행해도 결과는 동일하다. 



![IMAGE 037](https://user-images.githubusercontent.com/52769104/104091559-7630af80-52c1-11eb-951b-bb98719fa2a0.png)



![IMAGE 036](https://user-images.githubusercontent.com/52769104/104091558-74ff8280-52c1-11eb-89b5-82042746af2b.png)





### 네 번째 문제, Tom의 Twitter 계정 삭제하기



지금까지 학습한 모든 문제를 총 망라하는 문제이다 ! 



![IMAGE 021](https://user-images.githubusercontent.com/52769104/104091563-7fba1780-52c1-11eb-9f2b-dd4a351f3bbf.png)



이 문제의 hint를 봐도 JWT Header 값을 잘 살펴보라 명시되어 있다.

그리고 JWT.io에 대입해봐도 역시 kid라는 key identifier가 존재하고 있다.

그렇다면 이 문제에서는 변조해야 할 것들이 많다는 것이다.

또, 원본 소스코드를 보고 판단해야 할 필요가 있다.
https://github.com/WebGoat/WebGoat/blob/develop/webgoat-lessons/jwt/src/main/java/org/owasp/webgoat/jwt/JWTFinalEndpoint.java 이곳에 제공되어 있다.





**풀이 방법은 다음과 같다.**

```html
1. WebGoat에서 Tom의 Delete 버튼 클릭
2. 버프슈트로 넘어간 패킷을 Send to Repeater 
3. POST 이후에 온 파라미터 token 값을 복사 후 JWT.io Encoded 에 붙혀넣기
4. 다음 단계로 Decoded 부분의 HEADER, PAYLOAD, VERIFY SIGNATURE 부분을 변조
 4.1. HEADER -> kid부분에 "something_else' union select 'new_key의 base64값' from INFORMATION_SCHEMA.SYSTEM_USERS; --" 로 수정
 4.2. PAYLOAD -> username을 "Tom"으로 수정
 4.3. VERIFY SIGNATURE 부분에 new_key  입력
5. 이렇게 입력된 값을 버프슈트의 동일 부분에 수정 후 Send.
```



![IMAGE 022](https://user-images.githubusercontent.com/52769104/104091569-86e12580-52c1-11eb-9041-1efcbee3cbd3.png)



![IMAGE 025](https://user-images.githubusercontent.com/52769104/104091576-8c3e7000-52c1-11eb-8396-c093df126451.png)

![IMAGE 026](https://user-images.githubusercontent.com/52769104/104091578-8d6f9d00-52c1-11eb-8f72-80843826f0c9.png)



![IMAGE 028](https://user-images.githubusercontent.com/52769104/104091584-9a8c8c00-52c1-11eb-8b62-85e6316056f8.png)





## 여기까지 잘 진행했으면 (A2) Broken Authentication 메뉴의 두 부분이 초록색 체크박스로 채워져 있을 것이다.



![IMAGE 038](https://user-images.githubusercontent.com/52769104/104091587-9fe9d680-52c1-11eb-9077-861d99c10a1d.png)
