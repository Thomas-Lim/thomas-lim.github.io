---
layout: single
title: WebGoat Broken Authentication (인증 우회) Solution 2
categories: InfoSecurity
tag: [blog, infosecurity, webgoat]
toc: true
---



*WebGoat의 (A2) Broken Authentication메뉴 중 Password reset 과 Secure Passwords 에 대한 설명과 풀이를 다룬다.*





# 1. 설명



이번 장에서는 비밀번호를 유추해 내는 방법과, 안전한 비밀번호를 만드는 방법에 대해 소개한다. 

비밀번호를 유추해 낼 수 있는 사회공학적 기법은 굉장히 많다.

앞 선 과제에서 봤듯, 2FA를 이용하더라도 질의응답을 통해 유추해 볼 수 있는 방법이 있는 등 처음부터 최대한 자신의 비밀번호를 지킬수 있도록 Server - Client 상호간의 신경을 써야 한다.

**특히 취약한 부분은 '비밀번호 찾기' 시에 많이 나타나는데,** 이와 관련되어 찾고싶은 비밀번호에 대한 응답을 평문으로 제공해주는 (!!!) 사이트들만 모아놓은 사이트를 앞서 소개한다.

https://plaintextoffenders.com/

![IMAGE 020](https://user-images.githubusercontent.com/52769104/104093646-2bb62f80-52cf-11eb-9fad-f2bd02eda59e.png)





# 2. (A2) Broken Authentication - 문제들 ②

## - Password reset



이번 장은 WebWolf의 메일링 기능, 그리고 Incoming Requests LOGS 기능을 사용해서 문제를 풀어나간다.



### 첫 번째 문제,

내 계정에 대한 Password 찾기 문제이다. 



![IMAGE 001](https://user-images.githubusercontent.com/52769104/104093650-31137a00-52cf-11eb-8462-39f60b46e0c2.png)



 **풀이 방법은 다음과 같다.**

```
1. Forgot your password? 를 클릭, 본인의 계정을 입력한다.
2. WebWolf로 이동해 수신된 메일에서 Password를 확인한다.
3. WebGoat로 돌아와 로그인한다.
```





![IMAGE 002](https://user-images.githubusercontent.com/52769104/104093657-3a044b80-52cf-11eb-989b-bcf0e69f0ae2.png)

![IMAGE 003](https://user-images.githubusercontent.com/52769104/104093658-3b357880-52cf-11eb-800a-b6e0f94c14f8.png)

![IMAGE 004](https://user-images.githubusercontent.com/52769104/104093659-3b357880-52cf-11eb-9d9c-3c9fa02818fa.png)

![IMAGE 005](https://user-images.githubusercontent.com/52769104/104093661-3b357880-52cf-11eb-8711-c5bc061ee945.png)





### 두 번째 문제,

계정이 있는지 확인하는 (유추하는), 그리고 비밀번호를 찾아보는 페이지이다.

입력된 아이디 존재 여부에 따라 이후 노출되는 페이지가 달라진다. 

그리고 비밀번호 찾기에서 쉬운 질문은 뻔한 대답을 이끌어 내고 그것이 바로 취약한 부분이 된다.

이렇게 취약한 질문 생성을 예방할 수 있는 페이지는 다음에 있다.
http://goodsecurityquestions.com/



![IMAGE 021](https://user-images.githubusercontent.com/52769104/104093677-556f5680-52cf-11eb-95d8-3ad9c8f813b4.png)





![IMAGE 008](https://user-images.githubusercontent.com/52769104/104093681-5acca100-52cf-11eb-8797-0c94317bea9f.png)



 **풀이 방법은 다음과 같다.**

```
1. webgoat 외에 위에 소개된 tom, admin 등의 계정을 입력한다.
2. 색상에 관한 영문단어를 유추해서 입력.
```



![IMAGE 007](https://user-images.githubusercontent.com/52769104/104093684-6029eb80-52cf-11eb-954b-6ecfd6fcb450.png)





### 세 번째 문제,

취약한 질문 리스트와 왜 취약한지에 대한 이유이다.



![IMAGE 011](https://user-images.githubusercontent.com/52769104/104093688-65873600-52cf-11eb-9205-2436ee2c5eef.png)



단지 두 항목만 확인해보면 되므로 (Check) 살펴보며 풀어보길 바란다.



![IMAGE 010](https://user-images.githubusercontent.com/52769104/104093690-69b35380-52cf-11eb-858d-b8a8f65ddfc9.png)





### 네 번째 문제, 단 한번만 사용되는 비밀번호 재설정에 대한 링크 만들기이다.

내 계정에 대한 것이 아니라, tom의 비밀번호를 재설정 해서 tom으로 로그인하는 문제이다.

까다로운 문제처럼 보이지만, WebGoat의 시스템이 잘 되어 있어서 그리 어렵지 않게 풀 수 있는 문제이다. 

![IMAGE 012](https://user-images.githubusercontent.com/52769104/104093704-8a7ba900-52cf-11eb-958a-0ebc4492f6f4.png)



 **풀이 방법은 다음과 같다.**

```
1. 우선, 본인의 계정으로 패스워트 찾는 메일을 보낸다.
2. WebWolf로 이동해 수신된 메일에서 Link를 확인하고 연다.
3. 이와 함께 버프슈트에서 해당 패킷을 Repeater 한다.
4. Host 부분의 포트를 WebWolf 포트로 바꾸고, email을 tom의 이메일로 바꿔 Send
5. WebWolf의 Incoming requests 로 이동하면 로그가 들어와 있다.
6. uri의 ~reset-password/ 뒤 파라미터 값을 복사
7. 2번의 열려진 Link 뒤의 파라미터 부분을 해당 복사된 파라미터 값으로 수정한다.
8. 임의의 패스워드를 입력한다. (수정하고 싶은 것)
9. WebGoat로 돌아와, tom의 계정과 변경한 Password를 입력하면 성공
```



해당 풀이가 가능한 이유는, 

> Repeater로 변조시켜 Send 한 후, 가상으로 수신된 WebGoat상에 존재하는 Tom이 암호변경 메일을 수신하자마자 Link를 클릭해 읽는다는 기능이 있기 때문이다. 

그래서 Incoming Requests 에 Log가 남는 것이다.



![IMAGE 013](https://user-images.githubusercontent.com/52769104/104093706-90718a00-52cf-11eb-9e36-1833e32a9da9.png)



![IMAGE 022](https://user-images.githubusercontent.com/52769104/104093710-97000180-52cf-11eb-9786-df744698a89a.png)



![IMAGE 023](C:\Users\ultrarisk\Pictures\IMAGE 023.png)







![IMAGE 017](https://user-images.githubusercontent.com/52769104/104093714-9ff0d300-52cf-11eb-8f88-9fc88b5c972b.png)



![IMAGE 019](https://user-images.githubusercontent.com/52769104/104093720-a7b07780-52cf-11eb-918a-ec1140eeb6ea.png)



이후 소개된 **OWASP Cheat Sheet Series** 에서 이번 주제에 다뤄진 

**Forgot Password Cheat Sheet[¶](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html#forgot-password-cheat-sheet)** 에 대한 소개가 있으니 반드시 읽어보시길 바란다.





## - Secure Passwords

 

강력한 암호를 만드는 방법과 저장하는 방법에 관한 실습이다.

[NIST(National Institute of Standards and Technology)](https://www.nist.gov/)암호 표준에서 만든 가장 중요한 권장 사항이 소개되어 있다.

이 암호 사항은 [SP (Special Publications) 800](https://www.nist.gov/itl/publications-0/nist-special-publication-800-series-general-information) 시리즈에 소개되어 있다.



[Pwned Passwords](https://haveibeenpwned.com/Passwords) 에서는 자신이 사용하는 Password가 얼마나 많이 사용되고 있는지,![IMAGE 027](https://user-images.githubusercontent.com/52769104/104093738-d62e5280-52cf-11eb-982d-c7bc9c6a9da3.png)



[DEHASHED](https://www.dehashed.com/) 에서는 자신이 사용하는 ID가 어디에서 얼마나 쓰이고 있는지 확인할 수 있다.![IMAGE 028](https://user-images.githubusercontent.com/52769104/104093746-e47c6e80-52cf-11eb-8dfa-78b4843bf565.png)





### 첫 번째 문제, 

### NIST Password Standard에 맞는 강력한 Password 만들기



![IMAGE 024](https://user-images.githubusercontent.com/52769104/104093724-b4cd6680-52cf-11eb-9f16-5a030994ae4d.png)



**Password rules**에 따르는 강력한 암호를 만들면 된다.

NIST가 표준으로 요구하는 암호 규칙은 아래와 같다.

- **구성 규칙 없음**
- **암호 힌트 없음**
- **보안 질문 없음**
- **불필요한 암호 변경 금지**
- **최소 8 자 크기**
- **모든 유니 코드 문자 지원**
- **강도 측정기**
- **알려진 잘못된 선택과 비교하여 암호 확인**



```
문제에서 답으로 제출한 암호는 Brute-Force로 깨지는 데 3만 1천 7백년이 걸린다고 한다......
```



![IMAGE 025](https://user-images.githubusercontent.com/52769104/104093727-bc8d0b00-52cf-11eb-8ff4-d39d42ac1f3f.png)







다음 페이지는 안전한 패스워드를 저장하는 방법에 대한 방법들이다.



## Storing passwords

After a strong and secure password was created, it also has to be stored in a secure way. The NIST gives recommendations on how applications should handle passwords and how to store them securely.

### How should a password be stored?

- first of all: **use encryption and a protected channel for requesting passwords**
  The verifier shall use approved encryption and an authenticated protected channel when requesting memorized secrets in order to provide resistance to eavesdropping and MitM (Man-in-the-middle) attacks.
- **resistant to offline attacks**
  Passwords should be stored in a form that is resistant to offline attacks.
- **use salts**
  Passwords should be salted before storing them. The salt shall have at least 32 bits in length and should be chosen arbitrarily so as to minimize salt value collisions among stored hashes.
- **use hashing**
  Before storing a password it should be hashed with a one way key derivation function. The function takes the password, the salt and a cost factor as inputs and then generates a password hash.
  Examples of suitable key derivation functions:
  - Password-based Key Derivation Function 2 ([PBKDF2](https://pages.nist.gov/800-63-3/sp800-63b.html#SP800-132)) (as large as possible ⇒ at least 10.000 iterations)
  - [BALLOON](https://pages.nist.gov/800-63-3/sp800-63b.html#SP800-132)
  - The key derivation function shall use an approved one-way function such as:
    - Keyed Hash Message Authentication Code ([HMAC](https://pages.nist.gov/800-63-3/sp800-63b.html#FIPS198-1))
    - any approved hash function in [SP 800-107](https://pages.nist.gov/800-63-3/sp800-63b.html#SP800-107)
    - Secure Hash Algorithm 3 ([SHA-3](https://pages.nist.gov/800-63-3/sp800-63b.html#FIPS202))
    - [CMAC](https://pages.nist.gov/800-63-3/sp800-63b.html#SP800-38B)
    - Keccak Message Authentication Code (KMAC)
    - Customizable SHAKE (cSHAKE)
    - [ParallelHash](https://pages.nist.gov/800-63-3/sp800-63b.html#SP800-185)
- **memory hard key derivation function**
  Use memory hard key derivation functions to further increase the needed cost to perform attacks.
- **high cost factor**
  The cost factor (iteration count) of the key derivation function should be as large as verification server performance will allow. (at least 10.000 iterations)



## 여기까지 잘 진행했으면 (A2) Broken Authentication 메뉴의 모든 부분이 초록색 체크박스로 채워져 있을 것이다.

그러나, 왜 인지 Password reset 카테고리의 경우 모든 문제를 풀고 녹색불이 켜졌음에도 불구하고! 초록색 체크박스가 표시되고 있지 않다...


![IMAGE 026](https://user-images.githubusercontent.com/52769104/104093729-bd25a180-52cf-11eb-852d-13cc5b8ca095.png)
