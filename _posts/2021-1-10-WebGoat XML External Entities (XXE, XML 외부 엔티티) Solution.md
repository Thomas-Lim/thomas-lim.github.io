---
layout: single
title: WebGoat (A4) XML External Entities (XXE, XML 외부 엔티티)  Solution
categories: InfoSecurity
tag: [blog, infosecurity, webgoat, xml]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---



*WebGoat의 (A4) XML External Entities (XXE, XML 외부 엔티티 공격) 메뉴의  XXE에 대한 설명과 풀이를 다룬다.*





# 1. 설명



이번 장은 XML (eXtensible Markup Language) 를 사용한 공격에 대해 다룬다.
일반적으로, **'XML 외부 엔티티 공격'** 이라 불린다.

XML은 HTML을 개선/확장한 언어이다.

XML의 HTML과 가장 크게 다른점은, DTD (Document Type Definition) 이라는 것을 활용해 함수를 직접 만들어 사용할 수 있다는 것이다. HTML의 함수라 하면 <a>, <src> 등등이 있다.

엔티티(Entity)는 XML 문서 내의 데이터 항목을 참조하기 위해 DTD에서 선언하며, 문서 내에서 참조를 통해 사용된다. 또한 XML 문서가 다른 문자열로 변환되는 과정에서 Parsing(해석) 될 때 동작한다.

일반적으로 3가지 종류의 엔티티가 존재한다.

- Internal Entities
- External Entities
- Parameter Entities



참조하는 방법은 일반 엔티티 참조는 &~; 이며, DTD의 파라미터 엔티티의 참조는 %~;로 구성된다.

```xml-dtd
<?xml version="1.0" standalone="yes" ?>
<!DOCTYPE author [
  <!ELEMENT author (#PCDATA)>
  <!ENTITY js "Jo Smith">
]>
<author>&js;</author> 

(일반 엔티티 참조이므로, &js; 는 Jo Smith 호출됨.)
```



자세한 내용은 https://ko.wikipedia.org/wiki/XML 방문을 추천한다.



![IMAGE 001](https://user-images.githubusercontent.com/52769104/104122766-55805c80-538a-11eb-83eb-8ee347e74e6c.png)



# 2. (A4) XML External Entities (XXE) - 문제들

## - XXE



XML External Entity attack (Injection Attack)은 입력되는 XML 구문을 Parsing하는 웹사이트등을 대상으로 이뤄지는 공격이다. 취약한 XML Parser가 External Entities를 처리할 때 발생한다. 

아래 3가지 종류의 공격으로 구분된다.

- **Classic:** in this case an external entity is included in a local DTD
                (외부 엔티티가 로컬 DTD에 포함)
- **Blind:** no output and or errors are shown in the response
                (Blind SQL Injection 공격과 비슷함)
- **Error:** try to get the content of a resource in the error message
                (오류 메시지에서 리소스의 내용을 알 수 있음)



또한, DoS 공격 또한 가능한데, XXE를 이용한 유명한 공격은 **["Billion laughs"](https://en.wikipedia.org/wiki/Billion_laughs_attack)** 이다.

```xml-dtd
<?xml version="1.0"?>
<!DOCTYPE lolz [
 <!ENTITY lol "lol">
 <!ELEMENT lolz (#PCDATA)>
 <!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
 <!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
 <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
 <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
 <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
 <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
 <!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
 <!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
 <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
]>
<lolz>&lol9;</lolz>
```

**이 공격은 &lol;이 lol로 전환되며 10^9 = 10억번 수행되며 시스템에 엄청난 부하를 준다.**







### 첫 번째 문제, XXE Injection을 사용해 File System의 Root directory를 나열하는 것이다.



나열된 이미지에 댓글을 달아 XXE Injection을 시도한다.

XML DTD 일반 외부엔티티 형식을 이용할 것이다. 

주의할 점은, 해당 풀이 코드를 사용하면, 공격자 본인이 사용하는 File System의 디렉토리가 나열된다.



![IMAGE 003](https://user-images.githubusercontent.com/52769104/104122771-5f09c480-538a-11eb-9d44-e33c69866989.png)





 **풀이 방법은 다음과 같다.**

```
1. Burp Suite를 킨 상태로, WebGoat 문제에서 Add a comment에 무언가를 입력한다.
2. 스니핑 된 패킷에서 XML의 외부엔티티 상태를 확인한다.
3. 이 패킷을 Send to Repeater로 보낸 후 코드를 작성해 Send 한다.
4. WebGoat로 돌아와 댓글창에 입력된 나의 경로를 확인
```



**경로 작성 시에 /(Slash)는 반드시 3개로 구성해야 한다.**



![IMAGE 004](https://user-images.githubusercontent.com/52769104/104122777-6a5cf000-538a-11eb-8699-4768cf9efca4.png)

![IMAGE 005](https://user-images.githubusercontent.com/52769104/104122778-6b8e1d00-538a-11eb-97a4-7a77d2a3261f.png)

![IMAGE 006](https://user-images.githubusercontent.com/52769104/104122780-6b8e1d00-538a-11eb-84e3-789ef6caac20.png)

![IMAGE 007](https://user-images.githubusercontent.com/52769104/104122781-6c26b380-538a-11eb-92ff-d3108445a8a2.png)





### 두 번째 문제, REST framework를 사용한 XXE Injection을 사용해 File System의 Root directory를 나열하는 것이다.

첫 번째 문제와 유사하다.

하지만 차이점은, 서버용 웹 어플레케이션 제작 시 사용가능한 Library를 사용해 공격을 진행한다는 것이다.

이 형식은 JSON 형식이 문제에서 사용된다.

이 문제가 말하고자 하는 것은, 서버 개발자가 의도한 대로 메시지를 주고받지 않아도, 서버단에서 적당히 해석되어 Client에게 보여준다는 것이다. 잘 생각해보면 엄청난 보안취약점이다.



![IMAGE 008](https://user-images.githubusercontent.com/52769104/104122784-76e14880-538a-11eb-8563-b1845fe84689.png)





 **풀이 방법은 다음과 같다.**

```
1. Burp Suite를 킨 상태로, WebGoat 문제에서 Add a comment에 무언가를 입력한다.
2. 스니핑 된 패킷에서 JSON 형식임을 확인한다.
3. 이 패킷을 Send to Repeater로 보낸 후 코드를 작성해 Send 한다.
4. Content-Type의 형식을 XML로 수정한다.
5. WebGoat로 돌아와 댓글창에 입력된 나의 경로를 확인
```



**Content-Type 수정이 이 문제의 포인트이다.**



![IMAGE 010](https://user-images.githubusercontent.com/52769104/104122790-82cd0a80-538a-11eb-95e7-e7dd206f8269.png)

![IMAGE 011](https://user-images.githubusercontent.com/52769104/104122792-83fe3780-538a-11eb-8e6b-11cda1781dc9.png)

![IMAGE 012](https://user-images.githubusercontent.com/52769104/104122793-8496ce00-538a-11eb-9033-e00341bc47d5.png)

![IMAGE 014](https://user-images.githubusercontent.com/52769104/104122794-852f6480-538a-11eb-9fac-41c566d7b356.png)







### 세 번째 문제, 시스템의 상태를 체크하는 dtd파일을 이용, Blind XXE 를 실행하고, 결과를 댓글로 남겨라



이 문제를 이해하기 위해선 반드시 앞 페이지 (6번) Blind XXE 을 읽고 넘어가야 한다. 

![IMAGE 013](https://user-images.githubusercontent.com/52769104/104122802-92e4ea00-538a-11eb-80b9-950337f7078e.png)



여기에 쓰이는 파일 형식으로 해당 문제풀이를 위해 사용하기 때문이다.



> **그리고 이 문제 풀이를 위해선 WebWolf의 Logging기능이 필요하다.**



![IMAGE 015](https://user-images.githubusercontent.com/52769104/104122808-97a99e00-538a-11eb-9789-ef9e45d19ce6.png)





 **풀이 방법은 다음과 같다.**

```
1. 임의의 경로에 dtd 파일을 생성한다.
2. 생성된 파일을 WebWolf -> files를 통해 업로드한다.
3. WebGoat 문제에서 Add a comment에 무언가를 입력한다.
4. Burp Suite에 스니핑 된 패킷을 확인한다. 
5. 이 패킷을 Send to Repeater로 보낸 후 코드를 작성해 Send 한다.
6. 변조된 패킷을 Send 한다.
7. WebWolf -> Incoming requests를 확인한다.
8. 댓글창에 해당 내용을 적고, 시스템 정보가 노출됨을 확인한다.
```



주의 사항은, 문제의 'Location' 부분에 있는 경로로 가보면 txt 파일이 하나 있을테고,

이 txt 파일에 내용을 댓글로 달아도 문제 해결이 가능하다. 하지만 중요 사항은, DTD를 만들어 Upload 하는 것이다.





![IMAGE 016](https://user-images.githubusercontent.com/52769104/104122814-9f694280-538a-11eb-8efa-8a48b4e1ab0c.png)





> 파일명.dtd 파일을 업로드 해서 WebWolf에서 Log를 확인하고, 문제를 풀어야 한다.



![IMAGE 019](https://user-images.githubusercontent.com/52769104/104122823-aabc6e00-538a-11eb-9412-dbbbd72dc878.png)

![IMAGE 021](https://user-images.githubusercontent.com/52769104/104122819-a55f2380-538a-11eb-995f-5a6e7263b5a8.png)

![IMAGE 018](https://user-images.githubusercontent.com/52769104/104122825-b019b880-538a-11eb-82a4-b5ff6d85e9aa.png)



![IMAGE 017](https://user-images.githubusercontent.com/52769104/104122830-b445d600-538a-11eb-81f9-2d38b80bb475.png)



![IMAGE 020](https://user-images.githubusercontent.com/52769104/104122841-bf006b00-538a-11eb-8a84-7233b442e308.png)



추가적으로 XXE Mitigation에 대한 방안은 다음과 같다.

- DTD를 사용하지 않는다.
- 만약 사용이 꼭 필요하다면, External Entities만 사용토록 해야 한다.
- 검증 시 Client가 Header를 조작하는 것을 검출하기 위해 406(Not Acceptable) 오류를 띄우도록 해라.



더 자세한 XXE 예방책은 아래 사이트를 방문하면 좋다.

https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html



**모든 Injection 공격에 대한 최선의 대응책은 'Input Validation' 이다 !**



## 여기까지 잘 진행했으면 (A4) XML External Entities (XXE) 메뉴가 초록색 체크박스로 채워져 있을 것이다.



![IMAGE 022](https://user-images.githubusercontent.com/52769104/104122845-c3c51f00-538a-11eb-9129-d129321c3f29.png)
