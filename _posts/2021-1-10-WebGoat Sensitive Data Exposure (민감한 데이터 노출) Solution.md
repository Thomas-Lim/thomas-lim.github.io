---
layout: single
title: WebGoat Sensitive Data Exposure (민감한 데이터 노출) Solution
categories: InfoSecurity
tag: [blog, infosecurity, webgoat]
toc: true
author_profile: false
sidebar:
    nav: "docs"
---



*WebGoat의 (A3) Sensitive Data Exposure메뉴의 Insecure Login 에 대한 설명과 풀이를 다룬다.*





# 1. 설명



이번 장은 데이터 전송시에 (패킷 전송시에) 중요 데이터에 대한 암호화를 거치지 않고 전송시, 어떤 문제가 발생하는 지 알아 볼 수 있다.





# 2. (A3) Sensitive Data Exposure - 문제

## - Insecure Login



아주 간단한 문제이다. Burp Suite 등으로 패킷 스니핑을 해서, 정답에 맞는 username과 password를 알아내면 된다.



![IMAGE 001](https://user-images.githubusercontent.com/52769104/104119962-abe3a000-5376-11eb-865f-34048e533c2e.png)





 **풀이 방법은 다음과 같다.**

```
1. Burp Suite를 킨 상태로, Log in 버튼을 클릭한다.
2. 스니핑 된 패킷에서 usernamer과 password를 확인한다.
3. WebGoat로 돌아와 해당 계정으로 로그인한다.
```



이 문제의 **핵심**은, **평문으로 데이터 전송 시 이에 대한 위험성이다.**



![IMAGE 002](https://user-images.githubusercontent.com/52769104/104119964-ad14cd00-5376-11eb-9b2c-594ff9d71453.png)

![IMAGE 003](https://user-images.githubusercontent.com/52769104/104119966-adad6380-5376-11eb-9bb2-a4b8ada9ea44.png)

간단히 해결해 볼 수 있었다.

사실 이 문제는 (A2) Broken Authentication 와 문제 순서가 바뀌어야 한다고 생각된다.

암호화 된 패킷과 내용을 푸느라 정말 많은 경험을 한 상태에서 이번 문제는 너무나 간단한 문제였다.



## 여기까지 잘 진행했으면 (A3) Sensitive Data Exposure 메뉴가 초록색 체크박스로 채워져 있을 것이다.



![IMAGE 004](https://user-images.githubusercontent.com/52769104/104119968-ae45fa00-5376-11eb-9414-f93513eea03d.png)
