---
title: Dreamhack Background Exercise Cookie
date: 2022-02-09 01:57:34
tags: Dreamhack
categories: Dreamhack
---

## 문제 목표 및 기능 요약
해당 문제의 목표는 관리자 권한을 획득해 FLAG 를 획득하는 것입니다

문제에서는 다음 두 페이지를 제공합니다
- / : 이용자의 username 을 출력하고 관리자 계정인지 확인합니다
- /login : username , password 를 입력받고 로그인 합니다

## 웹 서비스 분석

### 엔드포인트 : /
Figure1 은 인덱스 페이지를 구성하는 코드입니다
해당 페이지에서는 요청에 포함된 쿠키를 통해 이용자를 식별합니다
만약 쿠키에 존재하는 username 이 admin 인 경우 FLAG 를 출력합니다

```python
@app.route('/') # / 페이지 라우팅 
def index():
    username = request.cookies.get('username', None) # 이용자가 전송한 쿠키의 username 입력값을 가져옴
    if username: # username 입력값이 존재하는 경우
        return render_template('index.html', text=f'Hello {username}, {"flag is " + FLAG if username == "admin" else "you are not admin"}') # "admin"인 경우 FLAG 출력, 아닌 경우 "you are not admin" 출력
    return render_template('index.html')
```

### 엔드 포인트 : /login
Figure2 는 로그인 페이지를 구성하는 코드입니다
코드를 살펴보면 메소드에 따른 요청마다 다른 기능을 수행하는 것을 알 수 있습니다

- GET : username 과 password 를 입력할 수 있는 로그인 페이지를 제공합니다
- POST : 이용자가 입력한 username 과 password 입력 값을 users 변숫값과 비교합니다
코드를 살펴보면 손님 계정의 비밀번호는 guest 관리자 계정의 비밀번호는 파일에서 읽어온 FLAG 임을 알 수 있습니다

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

## 취약점 분석
Figure3 을 살펴보면 이용자의 계정을 나타내는 username 변수가 요청에 포함된 쿠키에 의해 결정되어 문제가 발생합니다

쿠키는 클라이언트의 요청에 포함되는 정보로 이용자가 임의로 조작할 수 있습니다

서버는 별다른 검증 없이 이용자 요청에 포함된 쿠키를 신뢰하고 이용자 인증 정보를 식별하기 때문에 공격자는 쿠키에 타 계정 정보를 삽입해 계정을 탈취할 수 있습니다

## 익스플로잇
본 문제를 해결하기 위해서는 쿠키에 존재하는 username 을 admin 문자열로 조작해야 합니다

Figure4 와 같이 브라우저의 개발자 도구를 사용하면 쿠키의 정보를 확인하거나 수정할 수 있습니다

![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/b096921e0ee1008217d3b00672228bd1418fc11711a06b8dccaf7e061a775e92.png)

username 을 admin 으로 변경해 서버에 요청하면 Figure5 와 같이 FLAG 를 획득할 수 있습니다

![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/68a4be9ac07ebaa76ab59228b0c975f0d3c82b43d842e395259e909b5e0044e0.png)

## Cookie
>DH{7952074b69ee388ab45432737f9b0c56}

## Session - Basic
>DH{8f3d86d1134c26fedf7c4c3ecd563aae3da98d5c}

## Reference
Reference : https://dreamhack.io
