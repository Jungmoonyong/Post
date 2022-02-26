---
title: Basic Python Syntax
date: 2022-02-01 16:44:38
tags: Technology
categories: Technology
---

## 변수
```Python
a = 1
b = '하이'
c = 1.5
print(a)
print(b)
print(a + c)

d = int(c)
print(d)

a = int(200.15)
b = int(5.5)
print(a + b)
print(a * b)
print(a / b)

a = '100'
b = 50
print(int(a) + b)

a = 'Hello' * 3
print(a)

b = a + 'World'
print(b)
```

## 입력 문자열 처리
```Python
name = '정문용'
phone = '010 - 1234 - 1234'

print(name, '의 전화번호는 ', phone, '입니다 ')

name = input('이름을 입력 : ')

print(name)

name = input('이름을 입력 : ')
phone = input('전화번호 입력 : ')

print(name, '의 전화번호는 ', phone, '입니다')

a = int(input('국어 점수 입력 : '))
b = int(input('수학 점수 입력 : '))

score = a + b

print('시험 성적의 합은 ', score, '입니다')
print('시험 성적의 평균 ', score/2, '입니다')
```

## 조건문
```Python
if 2 > 1:
    print('hello') 

if not 2 < 1:
    print('hello')

if 1 > 0 and 2 > 1:
    print('hello')

if 1 > 0 or 2 < 1:
    print('hello')

a = 3

if a > 2:
    print('hello')

if a > 5:
    print('hello')
elif a == 3:
    print('hello2')
else:
    print('hi')

score = int(input('성적을 입력 : '))

if score >= 70:
    print('합격')
else:
    print('불합격')

score = int(input('성적을 입력 : '))

if score >= 90:
    print('A')
elif score >= 80:
    print('B')
else:
    print('C')

score1 = int(input('국어 성적을 입력 : '))
score2 = int(input('영어 성적을 입력 : '))
score3 = int(input('수학 성적을 입력 : '))

score_sum = score1 + score2 + score3
score_avg = score_sum/3

if score_avg >= 80:
    print('합격')
else:
    print('불합격')

import random
options = ['왼쪽', '중앙', '오른쪽']
choice = random.choice(options)
user = input('어디를 수비하겠습니까 : ')

if choice == user:
    print('성공')
else:
    print('실패')
```

## 함수
```Python
def chat():
    print('짱구 : 안녕 ? 넌 몇살이니? ')
    print('철수 : 나 ? 나는 5')

chat()

def chat(name1, name2):
    print('%s : 안녕 ? 넌 몇살이니? ' % name1)
    print('%s : 나 ? 나는 5' % name2)

chat('훈이', '수지')

def chat(name1, name2, age):
    print('%s : 안녕 ? 넌 몇살이니? ' % name1)
    print('%s : 나 ? 나는 %d' % (name2, age))

chat('훈이', '수지', 5)

def dsum(a, b):
    result = a + b
    return result

d = dsum(1, 2)
print(d)

def sayHello(name, age):
    if age < 10:
        print('안녕' + name)
    elif age <= 20 and age > 10:
        print('안녕하세요' + name)
    else:
        print('안녕하십니까' + name)

sayHello('짱구', 5)
sayHello('문용', 17)
sayHello('신형식', 30)

def add_members():
    print('신규 사용자 입력 메뉴 입니다')
    members_list.append(input('이름을 입력 : '))

def delete_members():
    print('사용자를 삭제 하는 메뉴 입니다')
    input('이름을 입력 : ')

def print_members():
    print('사용자를 출력 중 입니다')
    print(members_list)

members_list = []

print('아래 메뉴중에서 선택하세요')
print('[ 1 ] 신규 사용자 입력')
print('[ 2 ] 사용자 삭제')

menu_input = int(input('메뉴 번호 입력 : '))

if(menu_input == 1):
    add_members()
elif(menu_input == 2):
    delete_members()

print_members()
```

## 반복문
```Python
for i in range(3):
    print(i)
    print('짱구 : 하이 철수')
    print('철수 : 그냥 있어')

i = 0
while i < 3:
    print(i) # 0
    print('짱구 : 하이 철수')
    print('철수 : 그냥 있어')
    i = i + 1 # i = 0 + 1 = 1
    
i = 0
while True:
    print(i) # 0
    print('짱구 : 하이 철수')
    print('철수 : 그냥 있어')
    i = i + 1

    if i > 2:
        break

i = 0
for i in range(100):
    print(i) # 0
    print('짱구 : 하이 철수')
    print('철수 : 그냥 있어')
    i = i + 1 # i = 0 + 1 = 1

    if i > 2:
        break

for i in range(3):
    print(i) # 0
    print('짱구 : 하이 철수')
    print('철수 : 그냥 있어')

    if i == 1:
        continue

    print('훈이 : 안녕 짱구와 철수야')

for i in range(5):
    print('for')

for i in range(1,10,1):
    print(i)

names = ['문용', '짱구', '철수', '훈이']

for name in names:
    if '문용' in names:
        print(name)

if '문용' in names:
    print(names.index('문용'))

import random

number = random.randint(1,100)

for i in range(10):
    num_input = int(input('정답 입력  : '))
    if(number > num_input):
        print('입력한 것이 작습니다')
    elif(number < num_input):
        print('입력한 것이 큽니다')
    else:
        print('정답입니다')

num = int(input('구구단 입력 : '))

i = 1

while i <= 9:
    print(num, '*', i, '=', num * i)
    i = i + 1
```

## 리스트
```Python
a = [1,2,3,4]
b = ['hello', 'world']
c = ['hello', 1,2,3]
print(a) 
print(b)
print(c)
print(a + b)

print(a[0])

a[0] = 10
print(a)

num_elements = len(a)
print(num_elements)

a = [4,1,2,3]

b = sorted(a)
print(b)

a = [1,2,3,4]

for n in a:
    print(n)

a = [4,1,2,3]
b = ['hello', 'world']

for c in b:
    print(c)

print(a.index(1))
print('hello' in b)

if 'hello' in b:
    print('hello')

slist = ['국어', '영어', '수학', '과학']
score = []

score.append(90)
score.append(80)

print(slist)
print(slist[0])
print(slist[0], '점수는 ', score[0])

slist = ['국어', '영어', '수학', '과학']
score = []

score.append(input('국어 점수 : '))
score.append(input('영어 점수 : '))
score.append(input('수학 점수 : '))
score.append(input('과학 점수 : '))
print(score)
```

## 딕셔너리
```Python
a = {
    'name' : '짱구',
    'age' : 20,
}

print(a['name'])
print(a['age'])

a = {
    0 : '짱구',
    1 : '철수',
    'age' : 20,
}

print(a[0])
print('age' in a)
print(a.keys())
print(a.values())

a = {
    0 : '짱구',
    1 : '철수',
    'age' : 20,
}

for key in a:
    print(key)
    print(a[key])

a[0] = '훈이'
print(a)

a['test'] = '짱구'
print(a)

phonebook = {}

phonebook['정문용'] = '010 - 1234 -1234'

phonebook['짱구'] = '010 - 1234 - 1234'

print(phonebook['정문용'])

print(phonebook.keys())

print(phonebook.values())

menu = {'짜장면' : 5000, '짬뽕' : 6000, '탕수육' : 12000}

menu_sum = 0

answer = '예'

while answer == '예':
    input_menu = input('메뉴를 선택 : ')
    menu_sum = menu_sum + menu[input_menu]
    answer = input('주문을 입력 : ')

print('결제 금액은 ', menu_sum)
```

## 내장 함수
```Python
numbers = [10,20,30,40,50,60,70]

print('합', sum(numbers))
print('최대값', max(numbers))
print('카운트', len(numbers))

student_lst = []
students = 5

for i in range(students):
    score = int(input('성적 입력 : '))
    student_lst.append(score)

print('성적 평균 = ', sum(student_lst)/len(student_lst))
print('성적 최대 점수 = ', max(student_lst))
```

## 랜덤 함수 활용
```Python
import random

# print(random.random()) ramdom 함수로 난수 생성 0 ~ 1
# i = random.randrange(1, 5) # randrange 함수로 특정한 숫자들을 사이에서 발생하는데 정수로 발생 1 ~ 4
# i = random.randrange(2) # 0 ~ 1

i = random.randrange(1, 3)

if i == 1:
    print('홀')
else:
    print('짝')

menu = '비빔밥', '육계장', '오뎅국'

print(random.choice(menu)) # menu 에 있는 것을 랜덤으로 고름
```

### 랜덤 함수 게임
```Python
import random

count = 0

while 1: # 틀릴때까지 동작
    x = random.randint(1, 100) # randint 함수로 난수 발생
    y = random.randint(1, 100)

    print(x, '+', y, '= ???')

    answer = int(input('정답 : '))

    if answer == x+y:
        print('정답')
        count = count + 1
    else:
        print('오답')
        break

print(count, '번 맞췄습니다')
```

## txt 한글 문자열 파일 보기
```Python
import os

print(os.path.isfile('python_file.txt')) # path 와 isfile 함수로 python_file.txt 파일이 있냐
file = open('python_file.txt', 'r', encoding='UTF8') # txt 파일 오픈 한글 인코딩
lines = file.readlines() # 파일을 한줄씩 불러옴

# r : 읽기만
# w : 열고 파일 사용

for line in lines:
    print(line.rstrip()) # rstrip 함수로 줄바꿈 기호 삭제

file.close() # 파일 종료

file = open('python_file.txt', 'r', encoding='UTF8') # txt 파일 오픈 한글 인코딩
lines = file.readlines() # 파일을 한줄씩 불러옴

for line in lines:
    print(line.rstrip()) # rstrip 함수로 줄바꿈 기호 삭제
```

```txt
test
test123
test123123
```

## 디렉터리 와 파일 정보 리스트
```Python
savefile = open('save_file.txt', 'w', encoding='UTF8')

savefile.write('정문용 010 - 0000 - 0000')
savefile.write('테스트 010 - 0000 - 0001')
savefile.write('테스투 010 - 0000 - 0002')
savefile.close()

file = open('python_file.txt', 'r', encoding='UTF8') # txt 파일 오픈 한글 인코딩
lines = file.readlines() # 파일을 한줄씩 불러옴

for line in lines:
    print(line.rstrip()) # rstrip 함수로 줄바꿈 기호 삭제

file.close()
```

## with 키워드를 이용한 파일 읽기
```Python

try:
    with open('save_file.txt', 'r', encoding='UTF8') as file: # with 로 파일 오픈 하고 as 로 file 이라고 하는걸로 정의
    # 파일을 바로 가져옴
        for line in file:
            print(line.rstrip())

except IOError as e: # 파일을 여는 과정에서 오류가 발생할시 IOError 발생
    print('파일을 열 수 없습니다', e)

except Exception as e:
    print('오류 발생')
```

## HTTP 배너그래핑
```Python
import requests

url='http://puerta.kr' # URL 지정

response = requests.get(url) # req 값에 get 으로 url 주소를 불러온 것을 res 로 가져오겠다
print(response.status_code) # 상태 코드
print(response.text) # 결과 페이지 

import urllib.request

url = input('URL : ') # URL 지정

http_req = urllib.request.urlopen(url) # urllib 를 request 값을 가져와서 url 를 열겠다
if http_req.code == 200: # 상태 코드가 200 이면 http_req 의 헤더를 출력하라
    print(http_req.headers)
```