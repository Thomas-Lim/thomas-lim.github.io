---
layout: single
title: AWS 침투테스트 - Nessus로 취약점 스캔
categories: InfoSecurity
tag: [blog, infosecurity, nessus, vulnerabilities scan]
toc: true
---



*Nessus Vulnerability Scanner로 Ubuntu와 Windows 취약점 스캔하기*




# 1. 설명


네서스(Nessus)는 상용 취약점 스캐너이다. sectools.org에 따르면 네서스는 세계에서 가장 많이 사용되는 취약점 스캐너이다. 개발사인 Tenable Network Security는 2005년에 세계적으로 75,000 기관에서 사용한다고 추정하였다.

해당 스캐너로 EC2 상의 Windows와 Ubuntu Linux의 취약점을 스캔해본다.


![image 001213123](https://user-images.githubusercontent.com/52769104/107845773-7886ac00-6e21-11eb-8a54-4c43c164ee9d.png)






# 2. TightVNC 환경에서 Nessus 설치하기


우선 해당 Kali는 Web Browser 역할 하는 애플리케이션이 없으므로, **Firefox-esr 버전** 을 설치한다.



![image 030](https://user-images.githubusercontent.com/52769104/107845776-82101400-6e21-11eb-98bb-8cea16584a98.png)


설치 후 [다운로드 사이트](https://www.tenable.com/downloads/nessus)(https://www.tenable.com/downloads/nessus
)를 방문하여 debian 계열의 Kali 이므로 deb 버전을 설치한다. ** 7.2.3 버전 혹은 최신 버전인 8.13.1 버전을 추천한다.**


![image 031](https://user-images.githubusercontent.com/52769104/107845781-89372200-6e21-11eb-8406-351ce74d7afa.png)


이후 압축도 풀어준다.

![image 032](https://user-images.githubusercontent.com/52769104/107845782-89cfb880-6e21-11eb-9243-437384cd1638.png)


커맨드 창으로 돌아와 그림의 명령어와 같이 설치를 진행해준다.

그리고 설치가 완료 되었으면 nessus 데몬을 실행시켜준다.

![image 036](https://user-images.githubusercontent.com/52769104/107845789-90f6c680-6e21-11eb-84a0-09479e5222b5.png)




정상적으로 기동되고 있음을 Status를 통해 확인하고,
웹 브라우저를 통해 접속한다.
**기본적으로 127.0.0.1:8834 이다.**


![image 037](https://user-images.githubusercontent.com/52769104/107845792-948a4d80-6e21-11eb-8144-345e461b45d0.png)



# 3. SSH Tunneling 을 통해 사용자 PC에서 Nessus 접속 사용


PuTTY의 터널링 기능을 이용해 AWS Kali에서 실행되고 있는 Nessus를 사용자 PC의 웹 브라우저에서 사용할 수 있다.

방법은 간단하다.

1. 아래 사진과 같이 PuTTY의 터널링을 설정한다.
![image 040](https://user-images.githubusercontent.com/52769104/107845796-99e79800-6e21-11eb-9b96-4d333eb7ff22.png)


2. AWS Kali에서 Nessus 데몬을 실행한다.
3. 사용자 PC의 웹 브라우저에서 127.0.0.1:8834 으로 접속한다.


![image 038](https://user-images.githubusercontent.com/52769104/107845797-9ce28880-6e21-11eb-815b-b22763b6a6c0.png)




이후 사진에 나온 것 처럼 Basic Network Scan을 선택한다.

간단히 몇 가지 설정을 해줘야 하는데,

Name, Description, Folder 부분은 임의로 설정,

Targets는 EC2 Instance 내부망에서 취약점을 스캔할 것이므로
각각 Ubuntu와 Windows의 사설 IP를 Targets으로 지정한다.

![image 001](https://user-images.githubusercontent.com/52769104/107845799-a10ea600-6e21-11eb-815d-c5e8d876ad4a.png)



그리고 Scan Type은 많은 범위를 스캔할 수 있도록 Complex로 지정한다.


![image 004](https://user-images.githubusercontent.com/52769104/107845801-a4099680-6e21-11eb-8350-00be606d7f76.png)



이후 스캔을 시작해보면 약 10분 정도의 시간이 소요된다. 그리고 완료되면 'History' 탭에 결과가 나타난다.


![image 006](https://user-images.githubusercontent.com/52769104/107845802-a835b400-6e21-11eb-9d46-f1e34474c033.png)





각 시스템에 대한 취약점 내용이 아래 그림과 같이 나타난다.


![image 005](https://user-images.githubusercontent.com/52769104/107845805-abc93b00-6e21-11eb-9abd-a70c111af2cf.png)
![image 007](https://user-images.githubusercontent.com/52769104/107845806-aec42b80-6e21-11eb-8dfb-8ec4981473ec.png)




결과 페이지중 대표적으로 22번 ssh 포트가 열려 있다는 정보를 디테일하게 보여준다.


![image 008](https://user-images.githubusercontent.com/52769104/107845807-b1268580-6e21-11eb-876b-d90e0a68eae8.png)



## 이 결과를 바탕으로 Metasploit의 여러 도구와 조합하여 각종 공격 테스트를 진행할 수 있다.
