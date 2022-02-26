---
title: Dreamhack Background Web Basics
date: 2022-02-06 00:13:03
tags: Dreamhack
categories: Dreamhack
---

## Summary
- HTTP : 웹 서버와 클라이언트가 리소스를 교환하기 위해 사용하는 프로토콜 클라이언트가 요청하면, 서버가 응답하는 방식
- HTTP 메시지 : HTTP 서버와 클라이언트가 교환하는 데이터 헤드와 바디로 구성되며 각 줄은 CRLF 로 구분됨
헤드 : 메세지에 대한 정보 , 헤드의 끝에는 CRLF 한 줄이 있음
바디 : 클라이언트가 서버에게 , 또는 서버가 클라이언트에게 전달할 데이터
- HTTP 요청 : 클라이언트가 서버에게 특정 동작을 요청하는 메세지
메소드 : 요청 URI 가 가리키는 리소스에 대해 서버가 수행 했으면 하는 동작을 지정
요청 URI : 메소드의 대상이 되는 리소스를 지정
- HTTP 응답 : 요청을 처리한 결과 및 이유 그리고 클라이언트에 전송할 웹 리소스를 포함하는 메세지
상태코드 : 요청을 처리한 결과
처리 사유 : 상태 코드가 발생한 원인
- HTTPS : TLS 를 이용하여 HTTP 의 약점을 보완한 프로토콜



## HTTP / HTTPS

### 인코딩
컴퓨터의 모든 데이터는 0과 1 로 구성됩니다
사과가 사과를 가리키는 데 약속이 필요하듯, 0과 1로 우리의 문자를 표현하는 것도 일종의 약속 덕분입니다

이런 약속들을 특별히 인코딩 표준이라고 부르는데 대표적으로 아스키와 유니코드가 있습니다

아스키는 7비트 데이터에 대한 인코딩 표준입니다
이를 이용하면 알파벳과 특수 문자 등을 표현할 수 있습니다

예를들어 아스키에서 1 한개, 0 다섯 개, 1 한 개를 이어 붙이면 A 로 해석됩니다
이에 따라 1000001 이라는 데이터를 아스키로 변환하면 A 가 됩니다

컴퓨터가 개발된 초기에 각 문자권마다 고유의 인코딩 표준을 사용했는데 가끔 소프트웨어를 실행했을 때 글자가 --- 등으로 출력되는 것이 인코딩이 호환되지 않아 발생하는 문제입니다

이러한 어려움을 해결하고자 유니ㅗ드라는 새로운 표준이 만들어졌습니다
Uni 라는 접두사가 나타내듯, 유니코드는 모든 언어의 문자를 하나의 표준에 담겠다는 목표로 제정되었습니다

유니코드에서 한 문자는 최대 32 개의 비트로 표현되는데 32 비트로 표현할 수 있는 정보의 가짓수는 2 32, 대략 42억 개입니다

### 통신 프로토콜
웹 서버에 있는 리소스를 클라이언트가 받아 보려면 클라이언트는 웹에게 특정 리소스를 지정하여 제공해달라고 요청해야 합니다

그러면 서버가 요청을 이해하고, 대응되는 동작을 통해 클라이언트에게 리소스를 반환합니다
여기서 클라이언트의 행위를 Request 서버의 행위를 Response 라고 합니다

프로토콜은 위와 같이 규격화된 상호작용에 적용되는 약속을 이릅니다

### HTTP
HTTP 란 서버와 클라이언트의 데이터 교환을 요청과 응답 형식으로 정의한 프로토콜입니다
HTTP 의 기본 메커니즘은 클라이언트가 서버에게 요청하면 서버가 응답하는 것입니다

웹 서버는 HTTP 서버를 HTTP 서비스 포트에 대기시킵니다
이 포트는 일반적으로 TCP / 80 또는 TCP / 8080 입니다

클라이언트가 서비스 포트에 HTTP 요청을 전송하면 이를 해석하여 적절한 응답을 반환합니다

### 네트워크 포트와 서비스 포트
네트워크 포트란 네트워크에서 서버와 클라이언트가 정보를 교환하는 추상화된 장소를 의미합니다

포트에는 항구라는 의미가 있는데 클라이언트가 서버의 포트에 접근하여 데이터를 내려놓고 서버가 클라이언트에 보낼 데이터를 실어서 돌려보내는 장면을 연상하면 포트의 기능을 이해할 수 있습니다

네트워크를 설명하는 맥락에서는 네트워크를 생략하여 포트라고 부르기도 합니다

서비스 포트는 네트워크 포트 중에서 특정 서비스가 점유하고 있는 포트를 이릅니다
예를 들어 HTTP 가 80 포트를 점유하고 있다면 HTTP 의 서비스 포트는 80 번 포트가 됩니다

포트로 데이터를 교환하는 방식은 전송 계층의 프로토콜을 따릅니다
대표적으로는 TCP 와 UDP 가 있습니다

TCP 로 데이터를 전송하려는 서비스에 UDP 클라이언트가 접근하면 데이터가 교환되지 않습니다

반대의 경우도 마찬가지입니다
그래서 서비스포트를 표기할 때는 서비스가 사용하는 전송 계층 프로토콜을 같이 표기하기도 합니다

예를들어 HTTP 의 서비스 포트가 TCP / 80 이라고 하면 HTTP 서비스를 80 번 포트에서 TCP 로 제공하고 있다는 뜻입니다

포트의 개수는 운영체제에서 정의하기 나름입니다
그러나 현대의 윈도우나 리눅스 맥 운영체제는 0 번 부터 65535 번까지 총 65536 개의 같은 수의 네트워크 포트를 사용합니다

포트 중 0 번 부터 1023 번 포트는 잘 알려진 포트 또는 특권 포트 라고 합니다
문자 그대로 각 포트 번호에 유명한 서비스가 등록되어있습니다

대표적으로 22 번 포트에는 SSH, 80 에는 HTTP, 443 에는 HTTPS 가 할당되어 있습니다

잘 알려진 포트에 서비스를 실행하려면 관리자 권한이 필요합니다
따라서 클라이언트는 이 대역에서 실행 중인 서비스들은 관리자의 것이라고 신뢰할 수 있습니다

## HTTP 메시지
![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/4434bb5d339e95b7b64167b98571114c011e88bf05143422aa272b8bd943bba2.png)

HTTP 메시지에는 클라이언트가 전송하는 HTTP 요청 그리고 서버가 반환하는 HTTP 응답이 있습니다
기능과 세부 구조에서는 차이가 있지만 크게 보면 이들은 HTTP 헤드와 바디로 구성된다는 공통점이 있습니다

### HTTP 헤드
HTTP 헤드이 각 줄은 CRLF 로 구분되며 첫 줄은 시작 줄 Start - line, 나머지 줄은 헤더 Header 라고 부릅니다
헤드의 끝은 CRLF 한줄로 나타냅니다

시작 줄의 역할은 요청과 응답에서 큰 차이가 있습니다

헤더는 필드와 값으로 구성되며 HTTP 메시지 또는 바디의 속성을 나타냅니다
하나의 HTTP 메시지에는 0 개 이상의 헤더가 있을 수 있습니다

### HTTP 바디
HTTP 바디는 헤드의 끝을 나타내는 CRLF 뒤 모든 줄을 말합니다
클라이언트나 서버에게 전송하려는 데이터가 바디에 담깁니다

## HTTP 요청
HTTP 요청은 서버에게 특정 동작을 요구하는 메시지입니다
서버는 해당 동작이 실현 가능한지 클라이언트가 그러한 동작을 요청할 권한이 있는지 등을 검토하고 적절할 때만 이를 처리합니다

### 시작줄
```
GET
 
/index.html
 
HTTP/1.1
Host: dreamhack.io
Connection: keep-alive
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36
```

```
POST
 
/index.html
 
HTTP/1.1
Host: dreamhack.io
Connection: keep-alive
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36
Content-Length: 19
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
foo=bar&hello=world
```
HTTP 요청의 시작 줄은 메소드, 요청 URI, 그리고 HTTP 버전으로 구성됩니다
각각은 띄어쓰기로 구분합니다

메소드는 URI 가 가리키는 리소스를 대상으로 서버가 수행하길 바라는 동작을 나타냅니다
HTTP 표준에 정의된 메소드는 8개 있으나 자주 사용하는 GET 과 POST 메소드만 설명하겠습니다

GET 은 리소스를 가져오라는 메소드입니다
이용자가 브라우저에 웹 서버의 주소를 입력하거나 하이퍼링크를 클릭하면, 새로운 페이지를 렌더링하기 위해 리소스가 필요합니다

이때 브라우저는 GET 요청을 서버에 전송하여 리소스를 받아옵니다

POST 는 리소스로 데이터를 보내라는 메소드입니다
전송할 데이터는 보통 HTTP 바디에 포함됩니다
로그인할 때 입력하는 ID 와 비밀번호, 게시판에 작성하는 글 등이 POST 로 서버에 보내집니다

### 헤더와 바디
시작 줄을 제외한 헤더와 바디는 HTTP 메시지에서 설명한 것과 같습니다

## HTTP 응답
```
GET
 
/index.html
 
HTTP/1.1
Host: dreamhack.io
Connection: keep-alive
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36
```
```
HTTP/1.1
 
200 OK
Server: Apache/2.4.29 (Ubuntu)
Content-Length: 61
Connection: Keep-Alive
Content-Type: text/html

<!doctype html>
<html>
<head>
</head>
<body>
</body>
</html>
```
HTTP 응답은 HTTP 요청에 대한 결과를 반환하는 메시지입니다
요청을 수행했는지 하지 않았는지 안했다면 이유는 무엇인지와 같은 상태 정보 그리고 클라이언트에게 전송할 리소스가 응답에 포함됩니다

### 시작 줄
HTTP 응답의 시작 줄은 HTTP 버전, 상태 코드, 그리고 처리 사유로 구성됩니다

HTTP 버전은 서버에서 사용하는 HTTP 프로토콜의 버전을 나타냅니다
그리고 상태 코드는 요청에 대한 처리결과를 세 자릿수로 나타냅니다

HTTP 표준인 RFC 2616 은 대략 40 여개의 상태 코드를 정의하고 있는데
각각은 첫 번째 자릿수에 따라 5 개의 클래스로 분류합니다
처리 사유는 상태 코드가 발생한 이유를 짧게 기술한 것입니다

1xx
요청을 제대로 받았고, 처리가 진행 중임

2xx
요청이 제대로 처리됨
200: 성공

3xx
요청을 처리하려면, 클라이언트가 추가 동작을 취해야 함
302: 다른 URL로 갈 것

4xx
클라이언트가 잘못된 요청을 보내어 처리에 실패했습니다
400: 요청이 문법에 맞지 않음
403: 클라이언트가 리소스에 요청할 권한이 없음
404: 리소스가 없음

5xx
클라이언트의 요청은 유효하지만, 서버에 에러가 발생하여 처리에 실패했습니다
500: 요청을 처리하다가 에러가 발생함
503: 서버가 과부하로 인해 요청을 처리할 수 없음

## HTTPS
HTTP 의 응답과 요청은 평문으로 전달됩니다
만약 누군가 이를 가로챈다면 중요한 정보가 유출될 수 있습니다

예를들어 로그인할 때 전송한 POST 요청에는 대개 이용자의 ID 와 비밀번호가 포함됩니다
공격자가 중간에 이를 가로채면 이용자의 계정이 탈취될 수 있습니다

HTTPS 는 TLS 프로토콜을 도입하여 이런 문제점을 보완합니다
TLS 는 서버와 클라이언트 사이에 오가는 모든 HTTP 메시지를 암호화합니다

공격자가 중간에 메시지를 탈취하더라도 이를 해석하는 것은 불가능하며 결과적으로 HTTP 통신이 도청과 변조로부터 보호됩니다

웹 서버의 URL 이 http:// 로 시작되면 HTTP https:// 로 시작되면 HTTPS 프로토콜을 사용합니다

## Web

### 웹
인터넷을 기반으로 구현된 서비스 중 HTTP 를 이용하여 정보를 공유하는 서비스를 웹이라 합니다
여기서 정보를 제공하는 주체를 웹 서버 정보를 받는 이용자를 웹 클라이언트 라고 합니다

### 웹 리소스
웹 리소스란 웹에 갖춰진 정보 자산을 의미합니다

웹 브라우저의 주소창에 http://test.com/index.html 주소를 입력하면 test.com 에 존재하는 /index.html 경로의 리소스를 가져오라는 의미입니다

모든 웹 리소스는 URI 를 가지며 이를 이용하여 식별됩니다

- HTML
웹 문서의 뼈와 살을 담당합니다
태그와 속성을 통한 구조화된 문서 작성을 지원합니다

- CSS
웹 문서의 생김새를 지정합니다

- JavaScript
웹 문서의 동작을 정의합니다
이용자가 버튼을 클릭했을 때 어떻게 반응할지 이용자가 데이터를 입력하면 어디로 전송할지 등 JS 로 구현할 수 있습니다
JS 는 이용자의 브라우저에서 실행되는데 클라이언트가 실행하는 코드라고 하여 Client Side Script 라고도 부릅니다

### 웹 클라이언트와 서버의 통신
이용자가 브라우저를 이용하여 웹 서버에 접속합니다
브라우저는 이용자의 요청을 해석하여 HTTP 형식으로 웹 서버에 리소스를 요청합니다

HTTP 로 전달된 이용자의 요청을 해석합니다
해석한 이용자의 요청에 따라 적절한 동작을 합니다
이용자에게 전달할 리소스를 HTTP 형식으로 이용자에게 전달합니다

브라우저는 서버에게 응답받은 HTML CSS JS 등의 웹 리소스를 시각화하여 이용자에게 보여줍니다

## Web Quiz
>1. CSS
2. HTML
3. O
4. X
5. O
6. Javascript
7. X

## HTTP / HTTPS Quiz
>1. 443
2. Header
3. 80



* * *

## Reference
Reference : https://dreamhack.io/