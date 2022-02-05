---
layout: single
title: WebGoat와 WebWolf의 Standalone-setup
categories: infosecurity
tag: [blog, infosecurity, webgoat]
toc: true
---


*웹고트(WebGoat)와 웹울프(WebWolf)는 희생자와 공격자로 구성된 Wargame 이다.*

좀 더 가다듬자면, WebWolf는 공격에 필요한 파일을 업로드하고, 공격에 따른 메일과 파라미터 등의 로그가 수신되는 곳이라 할 수 있다.

몇 년 전 버전까지만 하더라도 WebGoat 단독으로 실행됐는데, 최근 버전에는 WebWolf가 추가되어서 두 가지를 함께 실행해야 완전한 테스트가 가능하다.

다들 한번쯤은 들어봤을 만한 것일텐데, WebGoat는 <a href="https://owasp.org/www-project-top-ten/">OWASP(The Open Web Application Security Project) Top 10</a>에 따른 웹 취약점에 대해 희생되는 가상의 사이트이다.



> **해당 실습은 Java 가상머신을 직접 실행하는 Standalone 방식으로 실행**
> 된다. 이 외에는 Docker를 활용한 방식이 있는데, 여기서는 다루지 않는다.



# 1. 설치

## 필요 한 것들

- [ ] **Java JDK 11**

  -> Oracle 사이트에서 <a href="https://www.oracle.com/kr/java/technologies/javase-jdk11-downloads.html">Download</a>

- [ ] **webgoat-server-8.0.0.M26.jar**

  -> WebGoat Git 사이트에서 <a href="https://github.com/WebGoat/WebGoat/releases/tag/v8.0.0.M26">Download</a>

- [ ] **webwolf-8.0.0.M26.jar**

  -> WebGoat Git 사이트에서 <a href="https://github.com/WebGoat/WebGoat/releases/tag/v8.0.0.M26">Download</a>



![IMAGE 001](https://user-images.githubusercontent.com/52769104/103536528-0a6fd080-4ed6-11eb-8fb6-e852644dc998.png)

## 환경 설정 순서

1. Java JDK 11 설치 후 시스템 변수를 등록한다.
2. webgoat-server-8.0.0.M26.jar와 webwolf-8.0.0.M26.jar는 다운로드 후 적당한 폴더에 넣는다.
3. webgoat-server-8.0.0.M26.jar와 webwolf-8.0.0.M26.jar는 **각각이 실행**한다. **저장된 폴더로 이동하여** ***Shift+마우스 우클릭*** 해 CMD나 Powershell을 실행한다.
4. 이후 아래 그림의 명령어를 실행한다.



![IMAGE 002](https://user-images.githubusercontent.com/52769104/103536532-0b086700-4ed6-11eb-8488-83fc8ae8a465.png)



# 2. 실행

Google Chrome에서 WebGoat와 WebWolf를 위한 창을 각각 열고, 아래 주소를 입력하면 접속 성공 !



- [ ] **WebGoat**

  -> 127.0.0.1:**8080/WebGoat**

- [ ] **WebWolf**

  -> 127.0.0.1:**9090/WebWolf**



![IMAGE 003](https://user-images.githubusercontent.com/52769104/103536534-0ba0fd80-4ed6-11eb-9e8a-a3291a45a289.png)





# 3. 마무리

- 각 가상환경 사이트에 접속이 가능하면, WebGoat에서 사용자 계정을 생성한다.
- **이 계정으로 WebWolf에서도 사용가능하다.**

