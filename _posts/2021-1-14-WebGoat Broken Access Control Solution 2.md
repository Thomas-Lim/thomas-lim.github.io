---
layout: single
title: WebGoat (A5) Broken Access Control Solution 2
categories: InfoSecurity
tag: [blog, infosecurity, webgoat]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---



*WebGoat의 (A5) Broken Access Control 메뉴의  Missing Function Level Access Control에 대한 설명과 풀이를 다룬다.*





# 1. 설명



이번 장은 액세스 컨트롤 중  Missing Function Level Access Control (기능 수준 액세스 제어 누락) 를 다룬다.

기능수준의 접근통제 누락 = 기능 수준의 액세스 제어 누락은 구현된 기능에 접근통제를 구현하지 않아, 심하게는 Guest 계정을 가진 자도 접근 할 수 있는 것을 얘기한다.



접근통제를 구현하지 않은 채로 특정 기능만 웹에서 보이지 않게 Hide 시키는 방법은 고전적으로는 **웹 관리자 페이지 버튼, 혹은 공유기 설정 페이지에서 쉽게 볼 수 있었다.**



 앞서 다뤘던 Insecure Direct Object References 와의 차이는, 아래와 같이 생각할 수 있다.

-  **Insecure Direct Object References 는 접근통제를 구분하지 않고 동일한 기능을 하게 설정한 반면,** 

- **Missing Function Level Access Control는 아예 구현하지 않았다.**

  



# 2. (A5) Broken Access Control - 문제들 2

## - Missing Function Level Access Control



### 첫 번째 문제, 드러나지 않은 UI의 숨겨진 기능을 찾는 것이다.

어리석게도, 단순히 HTML과 CSS의 속성을 통해 기능버튼을 숨겼을 뿐, URI에 파라미터 형식으로 주소를 입력하면 작동하는 경우도 상당히 많다.

참고사항으로, 이번 문제의 제목이 **Relying on Obscurity (모호성에 의존)** 인데, 기본적으로 요구되는 **Security Principles (보안 원칙) 에 포함되는 개념이기도 하다.**

보안 원칙의 10가지 구성요소는 다음과 같다.



# ★ 보안 원칙 (Security Principles)

> **1) Simplicity(단순성) :** **Keeping it simple** 원칙. **가능한 한 단순하게 = 가능한 한 작게.** 참조모니터의 검증가능성
>
> **2) Fail-Safe(러닝메이트 같은 존재 )** : 장애 발생 시 시스템은 **안전한 방법으로 셧다운**. 보안성 보다는 **기능면에서 손실보는 게 낫다 = Falut Tolarant 하게 설계**
>
> **3) Complete Mediation (완전성)** : 정보 직접 접근보다는  중재를. 승인 FW, 프락시, GW등, **참조모니터 우회 불가능**
>
> **4) Separation of Duty (직무분리) :** **기능 또는 직무는 분리 = 하나은행 = Granularity = 세분화 = 접근통제권한 세부적 = Fineness**
>
> **5) Least Privilege (최소권한=limit)** : Need to Know등. 각 업무와 프로세스, **사용자는 최소한의 권한**만을 부여받아야 함
>
> **6) Ease of use = Psychological Acceptability :** 인터페이스 직관적, **직원 교육**
>
> **7) Work Factor(암호 깨기 위한 시간/노력 투자됨)** : **공격자의 시간, 노력, 돈의 양** > 공격자가 취하려는 자산의 가치 **= 암호시스템 작업 요소에 영향을 주는 것 = 워크펙터 = 키 길이**
>
> **8) Open Design (개방형 - 독립)** : 시스템 보안은 그것의 구성분. **구성에서 독립되야 한다**. = 의존하지 말아야 한다 = **independent**
>
> **9) Obscurity (모호성)** : **모호성으로 보안 설계,** 개방형 설계 = 단순성의 반대개념, NAT / 근무교대시간. 유사사례:**스테가노 그라피.     심층적 방어와  공격자를 지연시키는 데 목적이 있음**
>
> **10) Defencse-in-Depth** : **Multi-Level Security = Layered Security = 보안통제를 Overlab (단점 : 복잡도 증가).  특히, APT,   CLoud,  물리보안, 가상화 환경의 답이 될 수 있다.**



문제 화면은 다음과 같다.



![IMAGE 005](https://user-images.githubusercontent.com/52769104/104580188-42c69a00-56a0-11eb-8d57-658f115b11bf.png)



 **풀이 방법은 다음과 같다.**

```
1. Chrome 개발자 도구를 꺼낸다 (F12)
2. 개발자 도구의 Elements 탭으로 이동한다.
3. Select an element ... 로 직접 페이지 내 구성요소를 선택할 수 있게한다.
4. Messages 라 적혀있는 controls 영역을 선택하고, 근처에서 Clue를 찾는다. 친절히도 hide 라는 키워드가 직접 들어가 있다.
5. 찾은 attribute를 WebGoat에 적고 Submit 한다.
```



![IMAGE 009](https://user-images.githubusercontent.com/52769104/104580233-570a9700-56a0-11eb-809e-dc914070cafa.png)

![IMAGE 020](https://user-images.githubusercontent.com/52769104/104580252-5c67e180-56a0-11eb-81af-cdabe0fb44e3.png)







### 두 번째 문제, 내 계정의 해시화 된 암호를 찾아 입력한다.

앞선 문제에서 찾은 경로를 사용해 유저 정보를 찾아보는 문제이다.



![IMAGE 026](https://user-images.githubusercontent.com/52769104/104580294-6ab5fd80-56a0-11eb-93ca-6bdd47546bb6.png)



config는 입력시 아래 사진에서 확인할 수 있듯 404 Not found 에러가 발생하지만,

![IMAGE 021](https://user-images.githubusercontent.com/52769104/104580331-7a354680-56a0-11eb-89db-19ef9e70b033.png)

![IMAGE 023](https://user-images.githubusercontent.com/52769104/104580378-8d481680-56a0-11eb-897b-50f1b73b27af.png)





users는 입력시 아무런 페이지가 뜨지 않는다. 하지만 상태코드는 200 OK이다.

![IMAGE 022](https://user-images.githubusercontent.com/52769104/104580356-83beae80-56a0-11eb-821d-03b6e438d7d8.png)

![IMAGE 024](https://user-images.githubusercontent.com/52769104/104580397-946f2480-56a0-11eb-9ada-a351b2cc3f98.png)





`WebGoat는 일반적으로 JSON 형식의 구조화된 컨텐츠 타입으로 구성되어 있다.`

`이를 힌트로 Burp Suite를 이용해 Repeater로 문제를 푼다.`



 **풀이 방법은 다음과 같다.**

```
1. Burp Suite를 킨 상태로 ...users 주소를 입력한다.
2. 캡쳐된 패킷을 Sent to Repeater로 보낸다.
3. 컨텐츠 타입을 JSON으로 입력 후 Send 한다.
4. Response 값을 확인 후 해쉬된 패스워드를 WebGoat 창에 입력한다.
```



![IMAGE 025](https://user-images.githubusercontent.com/52769104/104580522-bbc5f180-56a0-11eb-9779-a5966f5bdbb2.png)





여담이지만, JSON 형식으로 입력하지 않으면 WebGoat 개발과 정보글 작성에 참가한 사람들의 이름이 노출된다. Ending roll과 같은 페이지이다.



올바르게 찾아냈으면 아래와 같이 **축하화면** 을 볼 수 있다.

![IMAGE 027](https://user-images.githubusercontent.com/52769104/104580692-eadc6300-56a0-11eb-8b97-f9f25d670624.png)



## 여기까지 잘 진행했으면 (A5) Insecure Direct Object References의 모든 메뉴가 초록색 체크박스로 채워져 있을 것이다.

![IMAGE 028](https://user-images.githubusercontent.com/52769104/104580714-f039ad80-56a0-11eb-82e1-053d17a892f5.png)
