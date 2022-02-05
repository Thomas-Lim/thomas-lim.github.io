---
title: AWS 침투테스트 - Python Script로 취약한 S3 Bucket 탐색과 응용
layout: default
comments: true
---



*Python Script로 Public S3 Bucket을 스캔하고 접속과 파일생성을 시도해본다.*




# 1. 설명


S3 는 Simple Storage Service의 약자로, AWS의 스토리지 기능을 담당할 뿐만 아니라, 정적 웹호스팅 기능 또한 존재한다.

많은 자료 저장소들이 그러하듯, 당연히 정보자산에 대한 보안은 철저해야 한다. 

하지만 간혹 모든 자료를 공개적으로 읽기 및 쓰기가 가능하게 설정해 놓은 계정들이 있다.

이는 대단히 위험한 취약점이며, 휴먼 에러의 한 종류라 볼 수 있다.

역시 이런 취약점을 놓치지 않고 스캔할 수 있는, Python으로 제작된 스크립트가 존재한다.



![image 074](https://user-images.githubusercontent.com/52769104/107848003-b12f8100-6e33-11eb-9f24-cb8fcb16358e.png)



[https://github.com/jordanpotti/AWSBucketDump](https://github.com/jordanpotti/AWSBucketDump) 이곳에서 다운로드 가능하며, **이렇게 취약한 S3 버킷을 Bruteforcing하여 찾아내는** 등의 기능을 하는 코드를 담고있다 !




# 2. S3 버킷 생성 후 취약한 버킷 환경 만들어보기


### S3 버킷은 두 종류의 권한 시스템을 갖고 있다.

- **Access Central Policy** : 웹 UI에서 주로 사용. cmdlet으로 cli로 접근 또한 가능하다.
- **IAM 접근 정책** : 각 권한의 상세 내용을 볼 수 있는 JSON 오브젝트로 구성.

> S3 버킷의 권한 설정은 ACP 활용 시 최대 20KB 까지만 설정이 가능하다. 이후 설정은 IAM로 상세 설정이 가능하다는 정로도 이해해도 실습에는 전혀 문제가 없다.


#### - 취약한 버킷 만들기

아래와 같이 임의의 문자를 입력해 버킷을 생성하고, ACL을 편집한다.

![image 061](https://user-images.githubusercontent.com/52769104/107848011-b987bc00-6e33-11eb-9d41-d117291c15ad.png)


![image 070](https://user-images.githubusercontent.com/52769104/107848017-bf7d9d00-6e33-11eb-954d-0305f60e1060.png)


![image 069](https://user-images.githubusercontent.com/52769104/107848015-bee50680-6e33-11eb-99bf-e5bc65be5966.png)



#### - CMDLET 이용 CLI로 AWS S3 Access

이를 수행하기 위해선 우선, `awscli` 설치가 필요하다.


![image 064](https://user-images.githubusercontent.com/52769104/107848022-c99f9b80-6e33-11eb-9c22-13f6fb291242.png)




그리고, **aws configure**를 해 줘야 하는데 AWS Access Key ID와 PW가 필요하다.
이는 **AWS 사이트 -> 보안자격증명** 에서 발급이 가능하다.


![image 065](https://user-images.githubusercontent.com/52769104/107848027-cefce600-6e33-11eb-9239-b6374670881c.png)
![image 066](https://user-images.githubusercontent.com/52769104/107848028-cf957c80-6e33-11eb-9f12-490baf8810db.png)
![image 067](https://user-images.githubusercontent.com/52769104/107848029-d02e1300-6e33-11eb-852e-282f050aa09d.png)



이후, touch로 txt 파일을 하나 생성하여, 아래 명령어로 업로드 하면 다음 그림과 같은 화면을 확인 가능하다.

```
$ aws s3 cp abc.txt s3://버킷계정/abc.txt(업로드할 임의의 파일명)
```


![image 068](https://user-images.githubusercontent.com/52769104/107848031-d4f2c700-6e33-11eb-9afc-b46b1fdc6037.png)




여기까지 완료 후, 내 버킷 주소를 웹 브라우저 주소 입력창에 입력해보면, 가지고 있는 컨텐츠를 확인할 수 있다.


![image 071](https://user-images.githubusercontent.com/52769104/107848033-dae8a800-6e33-11eb-9960-4ef617aa528d.png)


컨텐츠 또한 내용의 확인이 가능하다.

![image 072](https://user-images.githubusercontent.com/52769104/107848032-da501180-6e33-11eb-93ee-b1b3849a7994.png)





# 3. AWSBucketDump 이용해 취약한 S3 버킷 스캔

아래 명령어를 사용해 git clone 해온다.

```
$ git clone https://github.com/jordanpotti/AWSBucketDump
$ cd AWSBucketDump
```

내부 파일 중 **BucketNames.txt 파일**을 보면 마치 Dictionary Attack 과 같이 취약 할 수 있는 문자열들을 사전에 정의해 놨다.


![image 075](https://user-images.githubusercontent.com/52769104/107848037-e340e300-6e33-11eb-9db5-b341e83876fd.png)
![image 076](https://user-images.githubusercontent.com/52769104/107848038-e3d97980-6e33-11eb-9d31-6f089ce2dfb3.png)



실행은 간단하다. AWSBucketDump.py 파일을 실행시키면 된다.

만약 import 대상이 없다고 하면 pip3를 활용해 라이브러리를 install 한다. 

실행 명령어는 다음과 같다.

```
$ python3 AWSBucketDump.py -D -l BucketNames.txt -g interesting_Keywords.txt 
```
interesting_Keywords.txt는 오브젝트 검색용이다.


![image 087](https://user-images.githubusercontent.com/52769104/107848040-e9cf5a80-6e33-11eb-9d1f-ed6e5ea8b177.png)



실행하면 다음과 같이 화면이 뜨는데,
**1. 접속할 수 없는, 취약하지 않은 곳이면 '~~~ is not accessible.'**
**2. 접속할 수 있는, 취약한 곳이면 '컨텐츠를 나열' 한다.**

![image 077](https://user-images.githubusercontent.com/52769104/107848049-f0f66880-6e33-11eb-90f7-ac105d0a362f.png)
![image 078](https://user-images.githubusercontent.com/52769104/107848050-f2279580-6e33-11eb-9d6d-dd028c6613fa.png)




**스캔 된 곳들 중 접속이 가능한 곳을 실제로 웹 브라우저의 주소 입력창에 입력해 보았고, 
컨텐츠들이 노출되며 심지어 개별 컨텐츠에 접근도 가능한 것을 확인했다.**

![image 079](https://user-images.githubusercontent.com/52769104/107848054-f653b300-6e33-11eb-82bf-41fd0c2b81e4.png)






# 4. AWSBucketDump 이용해 취약한 S3 버킷 스캔

S3 버킷 스캔을 통해 중요 데이터를 획득하는 것으로만 끝나는 것이 아니라, **S3 버킷에 악성코드를 업로드하여 자동실행 시, 해당 버킷을 방문하는 사람들에게 모든 영향을 끼칠 수 있거나, 백도어를 업로드 하여 언제든지 해당 버킷을 탐닉할 수도 있다.**


이에 대한 예제로, 간단히 악성 js 파일(이라 가정) 을 업로드 하여, 버킷 방문자에게 무조건적으로 alert을 띄우는 기능을 구현한다.

우선, 악성 js 파일은 다음과 같이 cli환경에서 업로드 가능하다. 물론 웹 UI에서도 가능하다.

![image 081](https://user-images.githubusercontent.com/52769104/107848060-fe135780-6e33-11eb-98c2-ee5645876ff1.png)





해당 테스트를 위한 최소 구성요소와 index.html 내용은 다음과 같다.

![image 080](https://user-images.githubusercontent.com/52769104/107848064-023f7500-6e34-11eb-9366-c4004df0f10e.png)


![image 086](https://user-images.githubusercontent.com/52769104/107848065-05d2fc00-6e34-11eb-801c-f660ad0c2bd7.png)







자 이제, 버킷을 열었을 시 자동으로 alert이 나오게 되며, 이후 메인 페이지가 보이게 된다.


![image 084](https://user-images.githubusercontent.com/52769104/107848069-0b304680-6e34-11eb-9360-5a066c4fff09.png)
![image 085](https://user-images.githubusercontent.com/52769104/107848070-0bc8dd00-6e34-11eb-8816-ac44b549f97c.png)





## 흥미롭게도 이렇게 취약한 버킷 환경을 구성하거나 응용해 다양한 방법으로 활용 및 연구해 볼 수 있다.
