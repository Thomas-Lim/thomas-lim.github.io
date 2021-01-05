---
layout: post
title: WebGoat-Introduction
---

아래 사진의 좌측은 웹고트(WebGoat), 우측은 웹울프(WebWolf)의 초기화면이다.



![IMAGE 004](https://user-images.githubusercontent.com/52769104/103536418-d1376080-4ed5-11eb-8db3-2d99df95ff83.png)

여기서 해야 할 것은 WebGoat의 Introduction 부분을 읽거나, 해결해야 한다.

물론 바로 공격 문제를 시작해도 된다.



WebGoat의 설명을 보면, 다른 동물을 공격하지 말라고 한다. 다른 동물 = 실제 웹사이트 라고 생각하면 되는데, WebGoat의 취지를 재미있게 표현한 부분이다.



# 1. 설명



여기서 풀어야 할 부분은 Introduction의 WebWolf 부분이다.

> 해당 포스트에선 이미 문제풀이를 완료했기 때문에 WebWolf에도 초록색 체크박스가 존재하지만, 초기에는 WebGoat에만 초록색 체크박스가 존재한다.



![IMAGE 005](https://user-images.githubusercontent.com/52769104/103536422-d4cae780-4ed5-11eb-94fa-db44f7b7de83.png)

1. 1번 페이지는 WebWolf의 소개이다
2. 2번 페이지는 WebWolf 창에서 파일 Uploading 에 관한 내용이다. WebGoat내에서 XSS 등의 공격을 테스트할 때 사용한다.
3. 3번 페이지는 WebWolf 창에서 확인할 수 있는 mailbox 설명이다. **이 페이지 부터 풀어야 할 문제가 있다.**
4. 4번 페이지는 WebWolf 창에서 Log로 확인할 수 있는 Landing Page에 대한 설명이다. **이 페이지 역시 풀어야 할 문제가 있다.**



# 2. Introduction - 문제

## - 3번 페이지 Your own mailbox

mailbox에 관한 설명과 함께 하단에 아주 간단한 풀이문제가 있다. (빨간 네모칸 참조)

![IMAGE 006](https://user-images.githubusercontent.com/52769104/103536423-d4cae780-4ed5-11eb-9b77-6790dd370ce4.png)



개괄적인 풀이 방법은 다음과 같다.

```
1. WebGoat 에서 자신의계정@임의의도메인을 사용해 (예: root@webgoat.org) 메일을 보낸다.
2. WebWolf 에서 메일 수신을 확인하고, 메일에 담겨있는 Unique Code를 확인한다.
3. WebGoat로 돌아와 해당  Unique Code를 입력한다.
```

![IMAGE 007](https://user-images.githubusercontent.com/52769104/103536424-d5637e00-4ed5-11eb-9adf-bb4f1b81e5d1.png)

## - 4번 페이지 Landing page

XSS 공격을 위한 Cookie 등을 수집하기 위해 랜딩 페이지로 WebWolf를 사용가능하며, 역시 연결되어 있다.



![IMAGE 008](https://user-images.githubusercontent.com/52769104/103536425-d5637e00-4ed5-11eb-8f5d-f41a5d85c72c.png)

개괄적인 풀이 방법은 다음과 같다.

```
1. 사진의 빨간네모칸의 Password Reset 메뉴를 클릭 후 임의의 문자로 리셋한다. 
2. 그 후 아무 페이지도 뜨지 않는다.
3. WebWolf 에서 Incoming requests를 확인하고, 요청값의 Unique Code를 확인한다.
4. WebGoat로 돌아와 해당  Unique Code를 입력한다.
```





![IMAGE 009](https://user-images.githubusercontent.com/52769104/103536426-d5fc1480-4ed5-11eb-94f6-db91e6e8b55c.png)



### 여기까지 잘 진행했으면 Introduction 메뉴가 초록색 체크박스로 채워져 있을 것이다.
