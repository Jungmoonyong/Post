---
title: Dreamhack Mitigation Same Origin Policy
date: 2022-02-10 00:41:30
tags: Dreamhack
categories: Dreamhack
---

## Summary
- Same Origin Policy : 동일 출처 정책 , 현재 페이지의 출처가 아닌 다른 출처로부터 온 데이터를 읽지 못하게 하는 브라우저의 보안 메커니즘
- Same Origin : 현재 페이지와 동일한 출처
- Cross Origin : 현재 페이지와 다른 출처
- Cross Origin Resource Sharing : 교차 출처 리소스 공유 , SOP 의 제한을 받지 않고 Cross Origin 의 데이터를 처리 할 수 있도록 해주는 메커니즘

Basckground : Cookie & Session 코드에서 쿠키에는 인증 상태를 나타내는 민감한 정보가 보관되며 브라우저 내부에 저장된다고 배웠습니다

그리고 브라우저가 서비스에 접속할 때 브라우저는 자동으로 쿠키를 헤더에 포함시켜 요청을 보낸다고 했습니다

이 덕분에 우리는 포털 사이트 , SNS 와 같은 서비스에 한 번 로그인한 후 일정 기간은 로그인하지 않고도 바로 서비스를 사용할 수 있습니다

하지만 이용자가 악의적인 페이지를 접속했을 때 , 페이지가 자바스크립트를 이용해 이용자의 SNS 웹 서비스로 요청을 보낸다면 어떻게 될까요 ?

우리가 이해한 대로라면 , 브라우저는 요청을 보낼 때 헤더에 해당 웹 서비스 쿠키르,ㄹ 포함시킬 것입니다

따라서 자바스크립트로 요청을 보낸 페이지는 로그인 된 이용자의 SNS 응답을 받을 것입니다
이에 더해 마음대로 페이지 접속자의 SNS 에 글을 올리거나 , 삭제하고 , SNS 메신저를 읽는 것도 가능할 것입니다

이와 같은 문제를 방지하기 위해 동일 출처 정책 보안 메커니즘이 생겼습니다
SOP 는 클라이언트 사이드 웹 보안에 있어 중요한 요소입니다

실제로 클라이언트 사이드 공격은 이 SOP 를 우회하기 위한 것이라고 해도 과언이 아닙니다

## Same Origin Policy
브라우저는 인증 정보로 사용될 수 있는 쿠키를 브라우저 내부에 보관합니다
그리고 이용자가 웹 서비스에 접속할 때 , 브라우저는 해당 웹 서비스에서 사용하는 인증 정보인 쿠키를 HTTP 요청에 포함시켜 전달합니다

이와 같은 특징은 사이트에 직접 접속하는 것에만 한정되지 않습니다
브라우저는 웹 리소스를 통해 간접적으로 타 사이트에 접근할 때도 인증 정보인 쿠키를 함께 전송하는 특징을 가지고 있습니다

이 특징 때문에 악의적인 페이지가 클라이언트의 권한을 이용해 대상 사이트에 HTTP 요청을 보내고 , HTTP 응답 정보를 획득하는 코드를 실행할 수 있습니다

이는 정보 유출과 같은 보안 위협이 생길 수 있는 요소가 됩니다
따라서 클라이언트 입장에서는 가져온 데이터를 악의적인 페이지에서 읽을 수 없도록 해야합니다

이것이 바로 브라우저의 보안 메커니즘인 동일 출처 정책 ( Same Origin Policy ) 입니다

## SOP 의 오리진 구분 방법
이제 브라우저가 가져온 정보의 출처인 오리진을 어떻게 구분하는지 알아보곘습니다
먼저 , 오리진은 프로토콜 ( Protocol , Scheme ), 포트 , 호스트 로 구성됩니다

구성 요소가 모두 일치해야 동일한 오리진이라고 합니다
`https://same-origin.com/` 라는 오리진과 아래 URL 을 비교했을 때 결과는 다음과 같습니다

URL : https://same-origin.com/frame.html
결과 : Same Origin
이유 : Path 만 다름

URL : http://same-origin.com/frame.html
결괴 : Cross Origin
이유 : Scheme 가 다름

URL : https://cross.same-origin.com/frame.html
결과 : Cross Origin
이유 : Host 가 다름

URL : https://same-origin.com:1234/
결과 : Cross Origin
이유 : Port 가 다름

## SOP 실습
SOP 는 Cross Origin 이 아닌 Same Origin 일 때만 정보를 읽을 수 있도록 해줍니다

예시로 우측에 `https://dreamhack.io` 에서 javascript 를 이용해 SOP 를 테스트 하는 코드를 작성하였습니다

여기서 `window.open` 은 새로운 창을 띄우는 함수이며 , `object.location.href` 는 객체가 가리키고 있는 URL 주소를 읽어오는 코드입니다

단 한번에 복사 붙여넣기 하는 것이 아닌 한줄씩 순서대로 입력해보세요 !

Same Origin
```javascript
sameNewWindow = window.open('https://dreamhack.io/lecture');
console.log(sameNewWindow.location.href);
결과: https://dreamhack.io/lecture
```

Cross Origin
```javascript
crossNewWindow = window.open('https://theori.io');
console.log(crossNewWindow.location.href);
결과: Origin 오류 발생
```

### Cross Origin 데이터 읽기 / 쓰기
위와 같이 외부 출처에서 불러온 데이터를 읽으려고 할 때는 오류가 발생해 읽지 못합니다

하지만 읽는 것 외에 데이터를 쓰는 것은 문제 없이 동작합니다
즉 아래와 같은 코드는 오류가 발생하지 않습니다

```javascript
crossNewWindow = window.open('https://theori.io');
crossNewWindow.location.href = "https://dreamhack.io";
```

## SOP 데모
지금까지 SOP 가 무엇인지 알아보고 , 오리진을 구분하는 방법에 대해 배웠습니다

아래 예시 코드는 다음 장의 모듈 구성 코드로 , 실습해보기 전에 살펴보겠습니다

### 코드 동작 설명
1. 두 번째 줄의 iframe 은 현재 웹 페이지 안에 또 다른 하나의 웹 페이지를 삽입하는 HTML 태그입니다
src 요소를 설정함으로써 삽입할 웹 페이지의 주소가 결정됩니다
2. 10 번째 줄의 onload 는 이벤트 핸들러로써 , 해당 객체가 성공적으로 로드되었을 때 동작합니다
본 예시에서는 10 ~ 23 번 줄이 iframe 객체에 페이지가 로드되면 동작하는 코드입니다
3. 14 ~ 15 번째 줄은 로드가 완료되면 iframe 내에 삽입된 주소에서 `secret - element` 객체의 값인 `treasure` 를 읽어와 콘솔에 출력하는 동작을 수행합니다

```html
<!-- iframe 객체 생성 -->
<iframe src="" id="my-frame"></iframe>
<!-- Javascript 시작 -->
<script>
/* 2번째 줄의 iframe 객체를 myFrame 변수에 가져옵니다. */
let myFrame = document.getElementById('my-frame')
/* iframe 객체에 주소가 로드되는 경우 아래와 같은 코드를 실행합니다. */
myFrame.onload = () => {
    /* try ... catch 는 에러를 처리하는 로직 입니다. */
    try {
        /* 로드가 완료되면, secret-element 객체의 내용을 콘솔에 출력합니다. */
        let secretValue = myFrame.contentWindow.document.getElementById('secret-element').innerText;
        console.log({ secretValue });
    } catch(error) {
        /* 오류 발생시 콘솔에 오류 로그를 출력합니다. */
        console.log({ error });
    }
}
/* iframe객체에 Same Origin, Cross Origin 주소를 로드하는 함수 입니다. */
const loadSameOrigin = () => { myFrame.src = 'https://same-origin.com/frame.html'; }
const loadCrossOrigin = () => { myFrame.src = 'https://cross-origin.com/frame.html'; }
</script>
<!--
버튼 2개 생성 (Same Origin 버튼, Cross Origin 버튼)
-->
<button onclick=loadSameOrigin()>Same Origin</button><br>
<button onclick=loadCrossOrigin()>Cross Origin</button>
<!--
frame.html의 코드가 아래와 같습니다.
secret-element라는 id를 가진 div 객체 안에 treasure라고 하는 비밀 값을 넣어두었습니다.
-->
<div id="secret-element">treasure</div>
```

## SOP 실습
이제 우리가 배운 SOP 의 동작을 떠올리며 , 실제 오른쪽 탭의 버튼을 클릭하고 동작을 살펴봅시다

iframe 에 Same Origin 이 아닌 Cross Origin 을 넣었다면 세 번째 동작 과정이 실패할 것입니다

### Same Origin 버튼
앞선 과정이 모두 성공적으로 수행되어 treasure 라는 값이 출력됩니다

### Cross Origin 버튼
앞선 과정 중 세 번째 과정이 실패하여 Cross Origin 의 데이터에 접근할 수 없다는 오류가 출력됩니다

## Cross Origin Resource Sharing
SOP 는 클라이언트 사이트 웹 보안에서 중요한 요소입니다
하지만 브라우저가 이러한 SOP 에 구애 받지 않고 외부 출처에 대한 접근을 허용해주는 경우가 존재합니다

예를 들면 , 이미지나 자바스크립트 , CSS 등의 리소스를 불러오는 `<img>` , `<style>` , `<script>` 등의 태그는 SOP 영향을 받지 않습니다

위 경우들 외에도 웹 서비스에서 동일 출처 정책인 SOP 를 완화하여 다른 출처의 데이터를 처리 해야 하는 경우도 있습니다

예를 들어 특정 포털 사이트가 카페 , 블로그 , 메일 서비스를 아래의 주소로 운영하고 있다고 합시다 각 서비스의 Host 가 다르기 때문에 브라우저는 각 사이트의 오리진이 다르다고 인식합니다

- 카페: https://cafe.dreamhack.io
- 블로그: https://blog.dreamhack.io
- 메일: https://mail.dreamhack.io
- 메인: https://dreamhack.io

이러한 환경에서 , 이용자가 수신한 메일의 개수를 메인 페이지에 출력하려면 , 개발자는 메인 페이지에서 메일 서비스에 관련된 리소스를 요청하도록 해야합니다

이 때 두 사이트는 오리진이 다르므로 SOP 적용받지 않고 리소스를 공유할 방법이 필요합니다

위와 같은 상황에서 자원을 공유하기 위해 사용할 수 있는 공유 방법을 교차 출처 리소스 공유 ( Cross Origin Resource Sharing ) 라고 합니다

교차 출처의 자원을 공유하는 방법은 CROS 와 관련된 HTTP 헤더를 추가하여 전송하는 방법을 사용합니다

이 외에도 JSON with Padding 방법을 통해 CORS 를 대체할 수 있습니다

## CORS
교차 출처 리소스 공유는 HTTP 헤더에 기반하여 Cross Origin 간에 리소스를 공유하는 방법입니다

발신측에서 CORS 헤더를 설정해 요청하면 , 수신측에서 헤더를 구분해 정해진 규칙에 맞게 데이터틀 가져갈 수 있도록 설정합니다

Figure2 와 Figure3 은 각각 웹 리소스를 요청하는 발신측 코드의 일부와 발신측의 HTTP 요청입니다

표를 살펴보면 발신측에서 POST 방식으로 HTTP 요청을 보냈으나 OPTIONS 메소드를 가진 HTTP 요청이 전달된 것을 확인할 수 있습니다

이를 CORS preflight 라고 하며 , 수신측에 웹 리소스를 요청해도 되는지 질의하는 과정입니다

Figure3 을 살펴보면 , Access - Control - Request 로 시작하는 헤더가 존재하는 것을 확인할 수 있습니다

해당하는 헤더 뒤에 따라오는 Method 와 Headers 는 각각 메소드와 헤더를 추가적으로 사용할 수 있는지 질의합니다

위처럼 질의하면 서버는 Figure4 와 같이 응답하며 , 다음은 응답 결과에 대한 설명입니다

Header : Access - Control - Allow - Origin
설명 : 헤더 값에 해당하는 Origin 에서 들어오는 요청만 처리합니다

Header : Access - Control - Allow - Methods 
설명 : 헤더 값에 해당하는 메소드의 요청만 처리합니다

Header : Access - Control - Allow - Credentials
설명 : 쿠키 사용 여부를 판단합니다 예시에 쿠키의 사용을 허용합니다

Header : Access - Control - Allow - Headers
설명 : 헤더 값에 해당하는 헤더의 사용 가능 여부를 나타냅니다

위 과정을 마치면 브라우저는 수신측의 응답이 발신측의 요청과 상용하는지 확인하고 , 그때야 비로소 POST 요청을 보내 수신측의 웹 리소스를 요청하는 HTTP 요청을 보냅니다

Figure 2 웹 리소스 요청 코드
```javascript
/*
    XMLHttpRequest 객체를 생성합니다. 
    XMLHttpRequest는 웹 브라우저와 웹 서버 간에 데이터 전송을
    도와주는 객체 입니다. 이를 통해 HTTP 요청을 보낼 수 있습니다.
*/
xhr = new XMLHttpRequest();
/* https://theori.io/whoami 페이지에 POST 요청을 보내도록 합니다. */
xhr.open('POST', 'https://theori.io/whoami');
/* HTTP 요청을 보낼 때, 쿠키 정보도 함께 사용하도록 해줍니다. */
xhr.withCredentials = true;
/* HTTP Body를 JSON 형태로 보낼 것이라고 수신측에 알려줍니다. */
xhr.setRequestHeader('Content-Type', 'application/json');
/* xhr 객체를 통해 HTTP 요청을 실행합니다. */
xhr.send("{'data':'WhoAmI'}");
```

Figure 3 발신측의 HTTP 요청
```
OPTIONS /whoami HTTP/1.1
Host: theori.io
Connection: keep-alive
Access-Control-Request-Method: POST
Access-Control-Request-Headers: content-type
Origin: https://dreamhack.io
Accept: */*
Referer: https://dreamhack.io/
```

Figure 4 서버의 응답
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://dreamhack.io
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Credentials: true
Access-Control-Allow-Headers: Content-Type
```

## JSON with Padding
좀 전에 이미지나 자바스크립트 , CSS 등의 리소스는 SOP에 구애 받지 않고 외부 출처에 대해 접근을 허용한다고 하였습니다

JSONP 방식은 이러한 특징을 이용해 `<script>` 태그로 Cross Origin의 데이터를 불러옵니다
하지만 `<script>` 태그 내에서는 데이터를 자바스크립트의 코드로 인식하기 때문에 Callback 함수를 활용해야 합니다

Cross Origin에 요청할 때 callback 파라미터에 어떤 함수로 받아오는 데이터를 핸들링할지 넘겨주면 , 대상 서버는 전달된 Callback으로 데이터를 감싸 응답합니다.

예시 코드 동작을 함께 살펴봅시다. Figure5를 살펴보면 , 13 번째 줄에서 Cross Origin의 데이터를 불러옵니다

이 때 callback 파라미터로 myCallback을 함께 전달합니다

Cross Origin에서는 응답할 데이터를 myCallback 함수의 인자로 전달될 수 있도록 myCallback으로 감싸 Javascript 코드를 반환해줍니다. 반환된 코드는 요청측에서 실행되기 때문에 3~6번 줄에서 정의된 myCallback 함수가 전달된 데이터를 읽을 수 있습니다

다만 JSONP는 CORS가 생기기 전에 사용하던 방법으로 현재는 거의 사용하지 않는 추세이기 때문에 , 새롭게 코드를 작성할 때에는 CORS를 사용해야 합니다

Figure 5 웹 리소스 요청 코드
```javascript
<script>
/* myCallback이라는 콜백 함수를 지정합니다. */
function myCallback(data){
    /* 전달받은 인자에서 id를 콘솔에 출력합니다.*/
	console.log(data.id)
}
</script>
<!--
https://theori.io의 스크립트를 로드하는 HTML 코드입니다.
단, callback이라는 이름의 파라미터를 myCallback으로 지정함으로써
수신측에게 myCallback 함수를 사용해 수신받겠다고 알립니다.
-->
<script src='http://theori.io/whoami?callback=myCallback'></script>
```

Figure 6 웹 리소스 요청에 따른 응답 코드
```
/*
수신측은 myCallback 이라는 함수를 통해 요청측에 데이터를 전달합니다.
전달할 데이터는 현재 theori.io에서 클라이언트가 사용 중인 계정 정보인
{'id': 'dreamhack'} 입니다. 
*/
myCallback({'id':'dreamhack'});
```

## Same Origin Policy Quiz
> 1. B
2. A
3. A
4. A , B , E

* * *

## Reference
Reference : https://dreamhack.io