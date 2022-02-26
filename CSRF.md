---
title: Dreamhack CSRF
date: 2022-02-17 16:36:25
tags: Dreamhack
categories: Dreamhack
---

## Summary
- CSRF : 사이트 간 요청 위조 이용자가 자신의 의지와는 무관하게 공격자가 의도한 행위를 특정 사이트에 요청하게 만드는 공격

이용자의 신원 정보가 포함된 쿠키는 일종의 서명과 같은 역할을 합니다
일상에서 서명은 서명자가 문서의 내용에 동의했다는 것을 함축합니다

이에 따라 서명된 문서는 효력을 갖게 되고 , 이용자의 권한도 부여 받습니다

쿠키도 마찬가지입니다
이용자의 식별 정보가 포함된 쿠키는 클라이언트에서 보내진 요청이 이용자로부터 왔으며 , 이용자가 동의했고 , 따라서 요청에 이용자의 권한이 부여돼야함을 의미합니다

실생활과 밀접한 웹 서비스가 많아지고 있기 때문에 , 서명을 신중하게 관리하는 것 만큼 중요한 웹 서비스의 쿠키도 잘 보관해야 합니다

서명과 관련된 범죄는 서명을 날조하거나 , 서명된 문서를 위조하는 것 등이 있습니다
이러한 범죄는 웹 해킹의 공격 기법들과도 대응합니다

전자는 이전 코스에서 살펴보면 쿠키를 탈취하는 공격과 그리고 후자는 이번 코스에서 배울 교차 사이트 요청 위조 와 비슷합니다

서명된 문서를 위조하는 것은 이미 서명된 문서의 내용을 조작하는 것을 말합니다
공백이 있는 문서에 서명하지 말아야 하는 것이 이러한 위험을 피하기 위함입니다

나중에 누군가 그 종이에 나는 A 에게 전재산을 양도한다 따위의 문장을 적어서 내용을 위조해도 , 서명으로 인해 효력을 가질 수 있기 때문입니다

CSRF 는 위와 비슷하게 이용자를 속여서 의도치 않은 요청에 동의하게 하는 공격을 말합니다
그럴듯한 페이지를 만들어서 이용자의 입력을 유도하고 , 이용자가 값을 입력하면 이를 은행이나 중요 포털 사이트 등으로 전송하여 마치 이용자가 동의한 것과 같은 요청을 발생시킵니다

만약 이용자가 자동 로그인 등의 기능을 사용하여 브라우저에 세션 쿠키를 저장하고 있었다면 , 실제로 계좌 이체가 발생하거나 비밀번호 초기화가 이뤄질 수도 있습니다

## CSRF
이전 코스에서 웹 서비스는 쿠키 또는 세션을 사용해 이용자를 식벽한다고 배웠습니다
임의 이용자의 쿠키를 사용할 수 있다면 이는 곧 임의 이용자의 권한으로 웹 서비스의 기능을 사용할 수 있습니다

CSRF 는 임의 이용자의 권한으로 임의 주소에 HTTP 요청을 보낼 수 있는 취약점입니다
공격자는 임의 이용자의 권한으로 서비스 기능을 사용해 이득을 취할 수 있습니다

예를들어 이용자의 계정으로 임의 금액을 송금해 금전적인 이득을 취하거나 비밀번호를 변경해 계정을 탈취하고 관리자 계정을 공격해 공지사항 작성 등으로 혼란을 야기할 수 있습니다

Figure2 는 CSRF 취약점이 존재하는 예제 코드로 송금 기능을 수행합니다
코드를 살펴보면 이용자로부터 예금주와 금액을 입력받고 송금을 수행합니다

이때 계좌 비밀번호 , OTP 등을 사용하지 않기 때문에 로그인한 이용자는 추가 인증 정보 없이 해당 기능을 이용할 수 있습니다

이용자가 HTTP 요청을 보내도록 하는 방법은 다양합니다

Figure1 이용자의 송금 요청
```
GET /sendmoney?to=dreamhack&amount=1337 HTTP/1.1
Host: bank.dreamhack.io
Cookie: session=IeheighaiToo4eenahw3
```

Figure2 
```python
# 이용자가 /sendmoney에 접속했을때 아래와 같은 송금 기능을 웹 서비스가 실행함.
@app.route('/sendmoney')
def sendmoney(name):
    # 송금을 받는 사람과 금액을 입력받음.
    to_user = request.args.get('to')
	amount = int(request.args.get('amount'))
	
	# 송금 기능 실행 후, 결과 반환	
	success_status = send_money(to_user, amount)
	
	# 송금이 성공했을 때,
	if success_status:
	    # 성공 메시지 출력
		return "Send success."
	# 송금이 실패했을 때,
	else:
	    # 실패 메시지 출력
		return "Send fail."
```

## CSRF 동작
![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/72e8464a88feedb75847c49e0e41a66cfb66fb810f592696e2bbc4edd931cc86.png)

CSRF 공격에 성공하기 위해서는 공격자가 작성한 악성 스크립트를 이용자가 실행해야합니다
이는 공격자가 이용자에게 메일을 보내거나 게시판에 글을 작성해 이용자가 이를 조회하도록 유도하는 방법이 있습니다

여기서 말하는 악성 스크립트는 HTTP 요청을 보내는 코드로 아래에서 요청을 보내는 스크립트를 작성하는 방법에 대해 알아보겠습니다

CSRF 공격 스크립트는 HTML 또는 Javascript 를 통해 작성할 수 있습니다

Figure5 는 HTML 으로 작성한 스크립트입니다
이미지를 불러오는 `img` 태그를 사용하거나 페이지에 입력된 양식을 전송하는 `form` 태그를 사용하는 방법이 있습니다

이 두 개의 태그를 사용해 HTTP 요청을 보내면 HTTP 헤더인 Cookie 에 이용자의 인증 정보가 포함됩니다

Figure3 은 img 태그를 사용한 스크립트의 예시입니다
해당 태그는 이미지의 크기를 줄일 수 있는 옵션을 제공합니다
이를 활용하면 이용자에게 들키지 않고 임의 페이지에 요청을 보낼 수 있습니다

Figure4 는 Javascript 로 작성된 스크립트의 예시입니다
새로운 창을 띄우고 , 현재 창의 주소를 옮기는 등의 행위가 가능합니다

Figure3 HTML img 태그 공격 코드 예시
```html
<img src='http://bank.dreamhack.io/sendmoney?to=dreamhack&amount=1337' width=0px height=0px>
```

Figure4 Javascript 공격 코드 예시
```javascript
/* 새 창 띄우기 */
window.open('http://bank.dreamhack.io/sendmoney?to=dreamhack&amount=1337');
/* 현재 창 주소 옮기기 */
location.href = 'http://bank.dreamhack.io/sendmoney?to=dreamhack&amount=1337';
location.replace('http://bank.dreamhack.io/sendmoney?to=dreamhack&amount=1337');
```

## CSRF 실습
오른쪽은 앞서 확인했던 Figure2 코드로 작성된 CSRF 실습 모듈입니다

해당 모듈은 일부 Javascript 실행이 제한되어 있으므로 HTML 태그를 통해 계좌 잔고를 1,000,000 원 이상으로 늘려보세요

```html
<img src="/sendmoney?to=dreamhack&amount=1337">
<img src=1 onerror="fetch('/sendmoney?to=dreamhack&amount=1337');">
<link rel="stylesheet" href="/sendmoney?to=dreamhack&amount=1337">
```

## XSS 와 CSRF 의 차이
XSS 와 CSRF 는 스크립트를 페이지에 작성해 공격한다는 점에서 매우 유사합니다
그렇다면 두 취약점의 공통점과 차이점을 알아보겠습니다

### 공통점
두 개의 취약점은 모두 클라이언트를 대상으로 하는 공격이며 이용자가 악성 스크립트가 포함된 페이지에 접속하도록 유도해야 합니다

### 차이점

두개의 취약점은 공격에 있어 서로 다른 목적을 가집니다
XSS 는 인증 정보인 세션 및 쿠키 탈취를 목적으로 하는 공격이며 공격할 사이트의 오리진에서 스크립트를 실행시킵니다

CSRF 는 이용자가 임의 페이지에 HTTP 요청을 보내는 것을 목적으로 하는 공격입니다
또한 공격자는 악성 스크립트가 포함된 페이지에 접근한 이용자의 권한으로 서비스의 임의 기능을 실행할 수 있습니다

## CSRF Quiz
> 1. A
2. A
3. B
4. B

* * *

## Reference
Reference : https://dreamhack.io