---
layout: post
title: AWS 침투테스트 - Kali Linux와 VNC 원격통신
comments: true
---



*Kali Linux 설치와 TightVNC로 원격환경 구축하기*




# 1. 설명


Kali Linux는 Debian 계열의 모의 침투 테스트용 OS이다.

안드로이드, 라즈베리파이, 크롬북 등 다양한 아키텍처에서 사용가능하며. AWS역시 Instance를 제공하고 있다.

AWS Marketplace에서 찾아볼 수 있으며, 가장 최신버전인 2020.04 까지 올라와 있다.


![image 002](https://user-images.githubusercontent.com/52769104/105463376-52af3100-5cd3-11eb-9769-d67b62f91e5b.png)



# 2. AWS Marketplace에서 Kali Linux AMI 설치하고 설정하기


설치 전에 하단으로 내려보면 각 EC2 Instance Type 별 요금정책이 설명되어 있다. **기본적으로 t2.medium 형식을 추천**하고 있으나, 개별적인 설정이 가능하다.


![image 003](https://user-images.githubusercontent.com/52769104/105463391-5773e500-5cd3-11eb-9a69-5902abc7e354.png)



### 칼리 리눅스 사용은 Free 이지만, 인스턴스 설정에 따른 추가 요금이 있다. 하지만 리전 설정으로 요금체계를 다르게 할 수 있다 (Tip)


![image 006](https://user-images.githubusercontent.com/52769104/105463405-5b076c00-5cd3-11eb-8ab6-660e12279d6b.png)



### 이전 장의 설정과 동일하게 원할한 접속과 통신을 위해 다음 두 가지를 유념해야 한다.
1) 인스턴스 모두 같은 pem 키를 사용하여 접속 할 것이다.
2) 인스턴스 모두 같은 VPC에 있어야 한다.


Kali Linux를 Subscribe 한 후에 Continue to Configuration.
이 곳에서 구독할 서비스의 형태를 설정 가능하다.
물론 64-bit의 최신 버전을 선택했다.

![image 007](https://user-images.githubusercontent.com/52769104/105463417-5e9af300-5cd3-11eb-921d-925fac5f1c43.png)

이후 Continue to Launch를 눌러 계속 진행하면 인스턴스 유형을 선택하고, 각종 정책을 생성할 수 있다. 

마음에 드는 것으로 선택 후 정책은 추후에 진행하도록 한다. **VPC만 체크하면 된다.**

![image 008](https://user-images.githubusercontent.com/52769104/105463428-622e7a00-5cd3-11eb-98b8-c1cc9c99c8bf.png)
![image 009](https://user-images.githubusercontent.com/52769104/105463430-622e7a00-5cd3-11eb-8866-b1722b519050.png)



# 3. Putty로 접속과 root 접속 권한 설정

접속 방법은 일반적인 AWS의 원격 Linux 접속 방법과 동일하다.

**AWS Kali Linux 의 사용자 이름은 다음과 같다.**

```
kali   : 일반 계정. kali@퍼블릭ip 로 접속가능
root   : root 계정. root@퍼블릭ip 로 접속가능. 기본적으로 활성화 안되어 있음.
```

![image 012](https://user-images.githubusercontent.com/52769104/105463445-68245b00-5cd3-11eb-9ca9-90c41d3d34ac.png)



만약 root 계정으로 접속하고 싶다면, 우선 kali 계정으로 접속 후 
**/etc/ssh/sshd config** 
를 다음 그림과 같이 설정해준 후 ssh를 Restart 해줘야 한다.


![image 013](https://user-images.githubusercontent.com/52769104/105463458-6c507880-5cd3-11eb-9ba0-7893bbdf6a5b.png)
![image 014](https://user-images.githubusercontent.com/52769104/105463462-6ce90f00-5cd3-11eb-9fd6-12a9e9cfcdb3.png)
![image 015](https://user-images.githubusercontent.com/52769104/105463464-6ce90f00-5cd3-11eb-94f1-d59daba67534.png)






# 4. 보안 강화를 위한 방화벽 설정하기

퍼플릭 아이피에 대해 원격접근을 구성하는 것은 큰 보안 리스크가 될 수 있다. 게다가 root권한까지 열어놓은 상태이니, 어떠한 공격에도 당할 가능성이 있게 된 것이다.

이를 조금이라도 예방하고자 **ufw**와 **fail2ban**을 설치한다.

- **fail2ban** : fail2ban 은 무작위로 로그인을 시도할 경우 해당 IP 를 커널 방화벽에 등록해 차단하는 기능을 한다.

- **ufw** : Uncomplicated Firewall의 약자로, 우분투와 데비안 리눅스에서 사용되는 간단한 방화벽이다. iptables 기반으로 작동하며, 블랙리스팅 용도로 사용할 것이다.


kali 최신버전에는 fail2ban이 원격저장소에 저장되어 있지 않은 것 같다.
그래서 **우선 apt sources.list를 수정해줘야 한다.**
이후 apt-get을 update와 fix 해준다.


![image 019](https://user-images.githubusercontent.com/52769104/105463480-73778680-5cd3-11eb-936c-73524cac03d0.png)
![image 020](https://user-images.githubusercontent.com/52769104/105463482-74101d00-5cd3-11eb-9009-26d956353eed.png)
![image 021](https://user-images.githubusercontent.com/52769104/105463483-74a8b380-5cd3-11eb-9dbd-a7f8a1aad064.png)
![image 022](https://user-images.githubusercontent.com/52769104/105463484-74a8b380-5cd3-11eb-932e-d251e2435e78.png)



수정이 완료 되었으면 **sudo apt install ufw fail2ban** 으로 설치한다.

![image 023](https://user-images.githubusercontent.com/52769104/105463491-783c3a80-5cd3-11eb-9461-8faeefbefc72.png)




ufw는 허용할 포트를 열어줘야 한다. 이후 재시작한다. 아래 그림과 같다.

![image 025](https://user-images.githubusercontent.com/52769104/105463500-7bcfc180-5cd3-11eb-90cb-831dfdf5a741.png)




# 5. tight vnc로 원격환경 구축하기

원격 데스크톱 연결을 구축하는 대표적인 방법인 **VNC (Virtual Network Computing) 와 RDP(Remote Desktop Protocol)** 이다.

VNC는 RFB(Remote Frame Buffer)프로토콜 방식을 이용해 서버에서 보낸 화면 정보를 클라이언트에 설치된 그래픽 라이브러리를 이용해 그리는 방식이다.

VNC는 주로 클라우드 상에서 동작하는 서버 VM을 동작하기 위해서 사용하는 프로토콜이며 QEMU 같은 가상화소프트웨어에서는 VNC를 디스플레이 옵션으로 기본으로 제공하고 있다. VM을 VNC로 연결하려는 경우 Hypervisor에서 VNC 서버만 실행해준다면 서버 VM에는 별도의 소프트웨어 설치가 필요 없으며 원격에서 접속하는 PC도 VNC Client 프로그램만 설치하면 Guest의 화면을 볼 수 있다.

![vnc](https://t1.daumcdn.net/cfile/tistory/99A836375B98F75510)
출처: https://selfish-developer.com/entry/VNC와-RDP [아는 개발자]


단점은 사용자 별로 세션 할당 불가, Server와 Client에 별도 프로그램 필요, 화질 저하 등의 문제가 있다.

그러나 구현이 쉽고 별다른 설정없이도 사용할 수 있다는 점에서 본 프로젝트데 알맞은 환경이라 할 수 있으므로 **가장 범용적으로 쓰이는 tightvnc를 사용한다.**


우선, 화면을 띄우기 위해 gui 데스크탑을 설치해야 하는데, AWS에서 제공하는 kali는 gui 환경이 구축되어 있지 않다.
심플한 mate theme로 구축하도록 한다.

```
apt-get install mate-core mate-desktop-environment-extra* mate-themes
```

꽤나 오랜시간이 걸려 설치가 완료된다.

이후 아래 명령어를 참조해 본격적으로 tightvnc을 설치한다. 

```
$ sudo apt-get update

$ sudo apt-get install xfce4 xfce4-goodies gnome-icon-theme tightvncserver sudo

<!- 사용자 계정 추가 ->
$ sudo adduser vnc
$ sudo gpasswd -a vnc sudo

<!- 서버 실행 ->
$ su - vnc
$ vncserver

```

여기까지 진행했으면 정상적으로 아래 메세지가 출력된다.

New 'X' desktop is sessionnumber:1

### 주의사항은, tightvnc는 기본 포트 설정이 590x이다. 

> New 'X' desktop is sessionnumber:1 이 1로 끝나면 5901,
New 'X' desktop is sessionnumber:2 이 2로 끝나면 5902 이다.
현재 실행중인 세션이 곧 포트넘버가 된다.


이후 당연히 인스턴스의 보안그룹에 해당 포트 번호를 인바운드 규칙으로 추가해줘야 한다.


![image 047](https://user-images.githubusercontent.com/52769104/105463515-838f6600-5cd3-11eb-9f0d-9efd4a9d7ac1.png)



그리고 이어서 tightvnc client를 실행 후, **퍼블릭IP:포트번호**를 입력,  connect를 누른다. 

![image 005](https://user-images.githubusercontent.com/52769104/105463643-b33e6e00-5cd3-11eb-85dc-b53ff2c06f05.png)



아래 화면이 뜬다 !!!


![image 044](https://user-images.githubusercontent.com/52769104/105463524-868a5680-5cd3-11eb-88c7-3949d4b0a905.png)





## 이 다음부터 Kali Linux와 Nessus를 활용해 본격적인 취약점 스캔과 침투테스트를 실행한다.
