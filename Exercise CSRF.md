---
title: Dreamhack Exercise CSRF
date: 2022-02-17 17:12:26
tags: Dreamhack
categories: Dreamhack
---

## 문제 목표 및 기능 요약
CSRF - 1 문제의 목표는 CSRF 를 통해 관리자 계정으로 특정 기능을 실행시키는 것입니다

- / : 인덱스 페이지 입니다
- /vuln : 이용자가 입력한 값을 출력합니다
이 때 XSS 가 발생할 수 있는 키워드는 필터링합니다
- /memo : 이용자가 메모를 남길 수 있으며 , 작성한 메모를 출력합니다
- /admin/notice_flag : 메모에 FLAG 를 작성하는 기능입니다
이 기능은 로컬호스트에서 접속해야 하고 , 사이트 관리자만 사용할 수 있습니다
- /flag : 전달된 URL 에 임의 이용자가 접속하게끔 합니다

## 웹 서비스 분석

### 엔드 포인트 : /vuln
Figure1 은 /vuln 페이지를 구성하는 코드입니다
코드를 살펴보면 이용자가 전달한 param 파라미터의 값을 출력합니다 

이 때 , 이용자의 파라미터에 frame , script , on 세 가지의 악성 키워드가 포함되어 있으면 이를 '*' 문자로 치환합니다

키워드 필터링은 XSS 공격을 방지하기 위한 목적으로 존재합니다
본 문제의 의도는 XSS 공격이 아닌 CSRF 공격을 통해서 관리자의 기능을 수행하는 것이기 때문에 일부 키워드를 필터링 하였습니다

Figure1 vuln 함수
```python
@app.route("/vuln") # vuln 페이지 라우팅 (이용자가 /vuln 페이지에 접근시 아래 코드 실행)
def vuln():
    param = request.args.get("param", "").lower()   # 이용자가 입력한 param 파라미터를 소문자로 변경
    xss_filter = ["frame", "script", "on"]  # 세 가지 필터링 키워드
    for _ in xss_filter:
        param = param.replace(_, "*")   # 이용자가 입력한 값 중에 필터링 키워드가 있는 경우, '*'로 치환
    return param    # 이용자의 입력 값을 화면 상에 표시
```

### 엔드 포인트 : /memo
Figure2 는 /memo 페이지를 구성하는 코드입니다
코드를 살펴보면 , 이용자가 전달한 memo 파라미터의 값을 기록하고 , render_template 함수를 통해 출력합니다

Figure2 memo 함수
```python
@app.route('/memo') # memo 페이지 라우팅
def memo(): # memo 함수 선언
    global memo_text # 메모를 전역변수로 참조
    text = request.args.get('memo', '') # 이용자가 전송한 memo 입력값을 가져옴
    memo_text += text + '\n' # 메모의 마지막에 새 줄 삽입 후 메모에 기록
    return render_template('memo.html', memo=memo_text) # 사이트에 기록된 메모를 화면에 출력
```

### 엔드 포인트 : /admin/notice_flag
Figure3 은 /admin/notice_flag 페이지를 구성하는 코드입니다
코드를 살펴보면 로컬 호스트 (127.0.0.1) 에서 접근하고 , userid 파라미터가 admin 일 경우 메모에 FLAG 를 작성하고 , 조건을 만족하지 못하면 접근 제한 메시지가 출력됩니다

/admin/notice_flag 페이지 자체는 모두가 접근할 수 있고 , userid 파라미터에 admin 값을 넣는 것도 가능합니다

하지만 일반 유저가 해당 페이지에 접근할 때의 IP 주소는 조작할 수 없습니다

따라서 일반 유저의 IP 가 자신의 컴퓨터를 의미하는 로컬 호스트 IP 가 되는 것은 불가능하기 때문에 , 이 페이지에 단순히 접근하는 것만으로는 플레그를 획득할 수 없습니다

로컬 호스트는 컴퓨터 네트워크에서 사용하는 호스트명으로 자기자신의 컴퓨터를 의미합니다

로컬 호스트를 IPv4 방식으로 표현했을 때에는 127.0.0.1 , IPv6 로 표현했을 때에는 00:00:00:00:00:00:00:01 로 표현합니다

Figure3 admin_notice_flag 함수
```python
@app.route('/admin/notice_flag') # notice_flag 페이지 라우팅
def admin_notice_flag():
    global memo_text # 메모를 전역변수로 참조
    if request.remote_addr != '127.0.0.1': # 이용자의 IP가 로컬호스트가 아닌 경우
        return 'Access Denied' # 접근 제한
    if request.args.get('userid', '') != 'admin': # userid 파라미터가 admin이 아닌 경우
        return 'Access Denied 2' # 접근 제한
    memo_text += f'[Notice] flag is {FLAG}\n' # 위의 조건을 만족한 경우 메모에 FLAG 기록
    return 'Ok' # Ok 반환
```

### 엔드 포인트 : /flag
Figure4 는 /flag 페이지를 구성하는 코드입니다
코드를 살펴보면 , 메소드에 따라 다른 기능을 수행합니다

### GET
이용자에게 URL 을 입력받는 페이지를 제공합니다

### POST
param 파라미터 값을 가져와 check_csrf 함수의 인자로 넣고 호출합니다
check_csrf 함수는 인자를 다시 CSRF 취약점이 발생하는 URL 의 파라미터를 설정하고 , read_url 함수를 이용해 방문합니다

이 때 방문하는 URL 은 서버가 동작하고 있는 로컬 호스트의 이용자가 방문하는 시나리오이기 때문에 127.0.0.1 의 호스트로 접속하게 됩니다

read_url 함수는 셀레늄을 이용해 URL 을 방문합니다

Figure4 flag 함수
```python
@app.route("/flag", methods=["GET", "POST"])    # flag 페이지 라우팅 (GET, POST 요청을 모두 받음)
def flag():
    if request.method == "GET": # 이용자의 요청이 GET 메소드인 경우
        return render_template("flag.html") # 이용자에게 링크를 입력받는 화면을 출력
    elif request.method == "POST":  # 이용자의 요청이 POST 메소드인 경우
        param = request.form.get("param", "")   # param 파라미터를 가져온 후,
        if not check_csrf(param):   # 관리자에게 접속 요청 (check_csrf 함수)
            return '<script>alert("wrong??");history.go(-1);</script>'
        return '<script>alert("good");history.go(-1);</script>'
def check_csrf(param, cookie={"name": "name", "value": "value"}):
    url = f"http://127.0.0.1:8000/vuln?param={urllib.parse.quote(param)}"  # 로컬 URL 설정
    return read_url(url, cookie)  # URL 방문
def read_url(url, cookie={"name": "name", "value": "value"}):
    cookie.update({"domain": "127.0.0.1"})  # 관리자 쿠키가 적용되는 범위를 127.0.0.1로 제한되도록 설정
    try:
        options = webdriver.ChromeOptions() # 크롬 옵션을 사용하도록 설정
        for _ in [
            "headless",
            "window-size=1920x1080",
            "disable-gpu",
            "no-sandbox",
            "disable-dev-shm-usage",
        ]:
            options.add_argument(_) # 크롬 브라우저 옵션 설정
        driver = webdriver.Chrome("/chromedriver", options=options) # 셀레늄에서 크롬 브라우저 사용
        driver.implicitly_wait(3)   # 크롬 로딩타임을 위한 타임아웃 3초 설정
        driver.set_page_load_timeout(3) # 페이지가 오픈되는 타임아웃 시간 3초 설정
        driver.get("http://127.0.0.1:8000/")    # 관리자가 CSRF-1 문제 사이트 접속
        driver.add_cookie(cookie)   # 관리자 쿠키 적용
        driver.get(url) # 인자로 전달된 url에 접속
    except Exception as e:
        driver.quit()   # 셀레늄 종료
        print(str(e))
        # return str(e)
        return False    # 접속 중 오류가 발생하면 비정상 종료 처리
    driver.quit()   # 셀레늄 종료
    return True # 정상 종료 처리
```

## 취약점 분석
/vuln 기능은 이용자의 입력 값을 페이지에 출력합니다
이때 입력값에서 frame , script , on 세 가지의 키워드를 필터링하기 때문에 XSS 공격은 불가능합니다

하지만 필터링 키워드 이외의 꺽쇠 < , > 를 포함한 다른 키워드와 태그는 사용할 수 있기 때문에 CSRF 공격을 수행할 수 있습니다

Figure1 csrf 함수
```python
@app.route("/vuln") # vuln 페이지 라우팅 (이용자가 /vuln 페이지에 접근시 아래 코드 실행)
def vuln():
    param = request.args.get("param", "").lower()   # 이용자가 입력한 param 파라미터를 소문자로 변경
    xss_filter = ["frame", "script", "on"]  # 세 가지 필터링 키워드
    for _ in xss_filter:
        param = param.replace(_, "*")   # 이용자가 입력한 값 중에 필터링 키워드가 있는 경우, '*'로 치환
    return param    # 이용자의 입력 값을 화면 상에 표시
```

## 익스플로잇
이 문제에서는 /vuln 페이지에서 CSRF 공격이 가능합니다
따라서 공격 코드가 삽입된 /vuln 페이지를 다른 이용자가 방문할 경우 , 의도하지 않은 페이지로 요청을 전송하는 시나리오의 익스플로잇을 구성해야 합니다

플레그를 얻기 위해서는 /admin/notice_flag 페이지를 로컬 호스트에서 접근해야 합니다
이를 위해 CSRF 공격으로 /vuln 페이지를 방문하는 로컬 호스트 이용자가 /admin/notice_flag 페이지로 요청을 전송하도록 공격 코드를 작성해야 합니다

로컬 호스트 환경의 이용자가 임의 페이지를 방문하게 하려면 /flag 페이지를 이용해야 합니다

### 테스트베드 생성
먼저 CSRF 취약점 발생 여부를 확인하기 위해 HTTP 응답을 받을 웹 서버가 필요합니다

만약 외부에서 접근 가능한 웹 서버가 없다면 드림핵에서 제공하는 드림핵 툴즈 서비스를 사용할 수 있습니다

서비스에서 제공하는 Request Bin 기능은 랜덤한 URL 을 제공하고 제공된 URL 에 대해 이용자의 접속 기록을 저장하기 때문에 XSS , CSRF 공격을 테스트하기 좋습니다

## CSRF 취약점 테스트
테스트베드를 생성했다면 CSRF 공격 코드를 작성하고 , 취약점 발생 여부를 확인합니다

지난 코스에서 배운 `<img>` 태그를 사용해 테스트베드 URL 에 접속하는 공격 코드를 작성한 뒤 , 취약점이 발생하는 페이지에 해당 코드를 삽입합니다

https://tools.dreamhack.games 에서 주소를 생성합니다

Figure6 , 7 를 확인해보면 공격 코드에 의해 이미지가 화면에 출력되었으며 , 생성한 테스트베드에 요청이 온 것을 볼 수 있습니다

```html
<img src="https://jugwwka.request.dreamhack.games">
```

![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/d39dd916fe91e3d559ee81a2c35391e4a187841b08cb6ded0930a7bddbeef478.png)

![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/a7eb765fe64226cbc91f79896cd1e482e46cb22ddc0ddd79fdd2eb46b134cd24.png)

## CSRF 공격 코드 작성 및 실행
어떠한 요청도 보내지 않은 상태로 메모 메뉴에 들어가면 Figure8 과 같이 hello 라는 메모만 기록되어 있습니다

로컬 호스트에 위치하는 이용자가 /admin/notice_flag 페이지를 방문하도록 해야하기 때문에 아래와 같이 공격 코드를 작성해야 합니다

이때 , userid 파라미터가 admin 인지 검사하는 부분이 존재하기 때문에 해당 문자열을 포함해야 합니다

```html
<img src="/admin/notice_flag?userid=admin" />
```

공격 코드를 작성했다면 , Figure9 와 같이 flag 페이지에서 공격 코드를 전송합니다

성공적으로 전송했다면 로컬 호스트에서 http://127.0.0.1:8000/vuln?param=<img src="/admin/notice_flag?userid=admin"/>에 접속하게 됩니다

Figure10 과 같이 memo 페이지에 접근하면 앞서 수행한 CSRF 공격으로 관리자가 /admin/notice_flag 페이지를 방문한 것을 확인할 수 있습니다

![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/fffc5a092c53ec7dda14ac6fbe6b1cff0ea773293381c8eb77fd22502d235f8a.png)

![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/f1e2c01eca83b1398257fefda3967483da38bd3b77c21f095b8b73a17b82270e.png)

![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/91f575f609d04bddc7670aa7d7fad92c558cba39272bff208092dafba956f54a.png)

* * *

## Reference
Reference : https://dreamhack.io