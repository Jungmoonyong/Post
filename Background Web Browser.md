---
title: Dreamhack Background Web Browser
date: 2022-02-06 11:47:42
tags: Dreamhack
categories: Dreamhack
---

## Summary
- 웹 브라우저 : 웹 브라우저는 HTTP / S 로 이용자의 웹 서버의 통신을 중개하며 , 서버로 부터 전달받은 다양한 웹 리소스들을 가공해 이용자에게 효과적으로 전달합니다
이용자가 다양한 프로토콜들을 알지 못해도 쉽게 웹을 사용할 수 있도록 도와줍니다
- URL : URL 은 리소스의 위치를 나타내는 문자열로 , 브라우저는 이를 사용하여 서버에 특정 리소스를 요청할 수 있습니다
- DNS : Host 의 도메인 이름을 IP 로 변환하거나 IP 를 도메인 이름으로 변환합니다
- 웹 렌더링 : 서버로 부터 받은 리소스를 이용자에게 시각화 하는 것을 말합니다

## 웹 브라우저
웹은 인터넷이라는 글로벌 네트워크 위에 구현되어 있으며 정해진 프로토콜을 기반으로 통신합니다

이용자가 dreamhack.io 를 입력했을때 다음과 같이 동작합니다
1. 웹 브라우저의 주소창에 입력된 주소를 해석
2. dreamhack.io 에 해당하는 주소 탐색 ( DNS 요청 )
3. HTTP 를 통해 dreamhack.io 에 요청
4. dreamhack.io 의 HTTP 응답 수신
5. 리소스 다운로드 및 웹 렌더링 HTML CSS Javascript

## URL
![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/12895a254539b5a98d3e22c6af025882215271a3231744279101323df838a1a2.png)

URL 은 웹에 리소스의 위치를 표현하는 문자열입니다
브라우저로 특정 웹 리소스에 접근할 때는 URL 를 사용하여 이를 서버에게 요청합니다

```
http://example.com:80/path?search=1#fragment
```
Scheme : 웹 서버와 어떤 프로토콜로 통신할지 나타냅니다
Host : 접속할 웹 서버의 주소에 대한 정보를 가지고 있습니다
Port : 접속할 웹 서버의 포트에 대한 정보를 가지고 있습니다
Path : 접근할 웹 서버의 리소스 경로로 / 로 구분됩니다
Query : 웹 서버에 전달하는 파라미터이며 URL 에서 ? 뒤에 위치합니다
Fragment : 메인 리소스에 존재하는 서브 리소스를 접근할 때 이를 식별하기 위한 정보를 담고 있습니다 # 문자 뒤에 위치합니다

## Domain Name
URL 구성 요소 중 Host 는 웹 브라우저가 접속할 웹 서버의 주소를 나타냅니다
Host 는 Domain Name , IP Address 의 값을 가질 수 있습니다

IP Address 는 네트워크 상에서 통신이 이루어질 때 장치를 식별하기 위해 사용되는 주소입니다
IP Address 는 외우기 어려우므로 도메인을 사용합니다

Domain Name 을 Host 값으로 이용할 때 브라우저는 Domain Name Server 에 Domain Name 을 질의하고 DNS 가 응답한 IP Address 를 사용합니다

웹 브라우저에서 http://example.com 에 접속할 경우 DNS 에 질의해 얻은 example.com 의 IP 와 통신합니다

```
$ nslookup example.com
Server:		8.8.8.8
Address:	8.8.8.8#53
Non-authoritative answer:
Name:	example.com
Address: 93.184.216.34
```

## 웹 렌더링
웹 렌더링은 서버로부터 받은 리소스를 이용자에게 시각화 하는 행위를 말합니다
서버의 응답을 받은 웹 브라우저는 리소스의 타입을 확인하고 적절한 방식으로 이용자에게 전달합니다

예를들어 서버로부터 HTML 과 CSS 를 받으면 브라우저는 HTML 을 파싱하고 CSS 를 적용하여 이용자에게 보여줍니다

웹 렌더링은 웹 렌더링 엔진에 의해서 이뤄지는데 브라우저별로 서로 다른 엔진을 사용합니다
사파리는 웹킷 , 크롬은 블링크 , 파이어폭스는 개코 엔진을 사용합니다

## 개발자 도구
개발자 도구를 실행하려면 브라우저를 열고 F12 를 누릅니다

빨간색 : 요소 검사 및 디바이스 툴바
주황색 : Elements: 페이지를 구성하는 HTML 검사

Console: 자바 스크립트를 실행하고 결과를 확인할 수 있음
Sources: HTML, CSS, JS 등 페이지를 구성하는 리소스를 확인하고 디버깅할 수 있음
Network: 서버와 오가는 데이터를 확인할 수 있음
Performance
Memory
Application: 쿠키를 포함하여 웹 어플리케이션과 관련된 데이터를 확인할 수 있음
Security
Lighthouse

노란색 : 현재 페이지에서 발생한 에러 및 경고 메시지
녹색 : 개발자 도구 설정

## 요소 검사
요소 검사를 활용하면 특정 요소의 개괄적인 정보를 파악하고 이와 관련된 코드를 쉽게 찾을 수 있습니다

## 디바이스 툴바
디바이스 툴바를 활용하면 현재 브라우저의 화면 비율 및 User - Agent 를 원하는 값으로 변경할 수 있습니다

## Elements
 
### HTML 읽기
현재 페이지를 구성하면 HTML 의 코드를 읽을 수 있습니다

### HTML 수정
코드를 선택한 상태로 단축키 F2 를 누르거나 더블 클릭하면 이를 수정할 수 있습니다

요소 검사 기능을 같이 활용하면 수정할 코드를 빠르게 선택할 수 있어 편리합니다

## Console
콘솔은 프론트엔드의 자바스크립트 코드에서 발생한 각종 메세지를 출력하고 이용자가 입력한 자바스크립트 코드를 실행도 해주는 도구입니다

콘솔을 사용하면 브라우저에서 자바스크립트를 실행하고 결과를 확인할 수 있습니다
```javascript
// "hello" 문자열을 출력하는 alert 함수를 실행합니다.
alert("hello");
// prompt는 popup box로 이용자 입력을 받는 함수입니다.
// 이용자가 입력한 데이터는 return value로 설정됩니다.
var value = prompt('input')
// confirm 는 확인/취소(yes/no)를 확인하는 이용자로부터 입력 받는 함수입니다.
// 이용자의 선택에 따라 Boolean(true/false)타입의 return value를 가집니다.
var true_false = confirm('yes or no ?');
// document.body를 변경합니다.
document.body.innerHTML = '<h1>Refresh!</h1>';
// document.body에 새로운 html 코드를 추가합니다.
document.body.innerHTML += '<h1>HI!</h1>';
```

## Sources
현재 페이지를 구성하는 웹 리소스들을 확인할 수 있습니다

## Sources: Debug
또한 Source 탭에서는 원하는 자바스크립트를 디버깅할 수도 있습니다
Figure5 는 이용자가 입력한 name, num 에 따라 실행 흐름이 바뀌는 코드인데 이를 다음과 같은 방법으로 디버깅할 수 있습니다

1. 원하는 코드 라인을 클릭하여 해당 라인에 브레이크 포인트를 설정합니다
2. 임의의 데이터를 입력하면 해당 브레이크포인트에서 실행이 멈춥니다
3. Scope 에서 현재 활당된 변수들을 확인하고 값을 변경할 수 있습니다

```javascript
</!DOCTYPE html>
<html>
<head>
    <title>JS Debug</title>
</head>
<body>
    <input type='text' id='input-name' placeholder='name'><br/>
    <input type='text' id='input-num' placeholder='num'><br/>
    <!-- 버튼 클릭 시 button_click함수가 실행됩니다. -->
    <input type='button' onclick='button_click()' value="Click">
<script>
    /*
     name과 num에 대한 변수를 검증하는 함수입니다.
     name이 'dreamhack', num이 31337인 경우 "congratulations !" 문자열을 출력합니다.
    */
    function compare(name, num){
        if(name == 'dreamhack'){
            if(num == 31337){
                console.log("congratulations !");
                return;
            }
        }
        console.log("No !");
    }
    /*
     버튼 클릭 시 실행되는 함수입니다.
     'input-name', 'input-num'의 값을 가져와 compare함수를 실행합니다.
    */
    function button_click() {
        var name = document.getElementById('input-name').value;
        var num = parseInt(document.getElementById('input-num').value);
        compare(name, num);
    }
</script>
</body>
</html>
```

## Network
서버와 오가는 데이터를 확인할 수 있습니다

빨간색 : 로깅 중지 및 로그 전체 삭제
주황색 : 로그 필터링 및 검색

옵션
- Preserve log : 새로운 페이지로 이동해도 로그를 삭제하지 않습니다
- Disable cache : 이미 캐시되니 리소스도 서버에 요청합니다

노랑색 : 네트워크 로그
파랑색 : 네트워크 로그 요약 정보

## Network: Copy
로그를 우클릭하고 Copy 에서 원하는 형태로 복사할 수 있습니다

Copy as fetch 로 HTTP Request 를 복사하고 Console 패널에 붙여서 실행하면 동일한 요청을 서버에 재전송할 수 있습니다

## Application
쿠키 , 캐시 , 이미지 , 폰트 , 스타일 시드 등 웹 애플리케이션과 관련된 리소스를 조회할 수 있습니다

Cookies 에서는 브라우저에 저장된 쿠키 정보를 확인하고 수정할 수 있습니다

## Console Drawer
개발자 도구에 새로운 콘솔창을 열어 가시성과 효율성을 높일 수 있습니다

## 페이지 소스 보기
페이지 소스 보기를 통해 페이지와 관련된 소스 코드들을 모두 확인할 수 있습니다

## Secret browsing mode
시크릿 모드에서는 새로운 브라우저 세션이 생성되며 브라우저를 종료했을 때 다음 항목이 저장되지 않습니다

- 방문기록
- 쿠키 및 사이트 데이터
- 양식에 입력한 정보
- 웹사이트에 부여된 권한

일반적으로 브라우저의 탭들은 쿠키를 공유하는데 시크릿 모드로 생성한 탭은 쿠키를 공유하지 않습니다
이를 이용하면 같은 사이트를 여러 세션으로 사용할 수 있어서 다수의 계정으로 서비스를 점검할 때 유용합니다

## Web Quiz
> 1. 웹 브라우저
2. X

## Browser DevTools
> 1. X
2. 시크릿 모드
3. O
4. Console.log

## Reference
Reference : https://dreamhack.io/