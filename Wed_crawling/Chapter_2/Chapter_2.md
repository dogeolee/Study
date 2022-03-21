# Chaper 2. 파이썬으로 시작하는 크롤링/스크레이핑

### 2-1 파이썬을 사용할 때의 장점
* 언어 자체의 특성 : 읽기 쉬운 프로그래밍 언어
* 강력한 라이브러리 : 풍부한 서브파티 라이브러리
* 스크레이핑 후처리의 편리성 : Numpy와 Scipy등을 기반으로 한 데이터 분석 전용 라이브러리 활용

### 2-2 가상환경 venv 생성하기
* 가상환경 venv 생성하기
<pre>
<code> 
$ python3 -m venv scraping
$ . scraping/bin/activate
</code>
</pre>


* wget이란?
  * HTTP 통신 또는 FTP 통신을 사용해 서버에서 파일 또는 콘텐츠를 다운로드할 때 사용하는 소프트웨어
  * 여러 파일을 한 번에 다운로드하거나 웹 페이지의 링크를 순회하며 여러 콘텐츠를 자동으로 다운로드 할 수 있음

* 설치 방법
<pre>
<code> 
$ sudo apt-get update
$ sudo apt-get install -y wget
</code>
</pre>

* 사용법
<pre>
<code>
$ wget https://wikibook.co.kr/logo.png # 로고파일 다운
$ wget https://wikibook.co.kr/ # html파일 다운
$ wget https://wikibook.co.kr/ -O wikibook_top.html # 저장할 이름 지정
$ wget https://wikibook.co.kr/ -q -O - # 다운로드한 html 출력
</code>
</pre>

### 2-3. 파이썬 기초 지식
* greet.py
<pre>
<code>
import sys

def greet(name):
    print('Hello, {0}!'.format(name))
if len(sys.argv) > 1:
    name = sys.argv[1]
    greet(name)
else:
    greet('world')
</code>
</pre>

* greet-with-comments.py
<pre>
<code>
# import 구문으로 sys 모듈을 읽어 들입니다.
import sys

# def 구문으로 greet() 함수를 정의합니다.
# 들여쓰기돼 있는 줄이 함수의 내용을 나타냅니다.
def greet(name):
    # print() 함수를 사용해 문자열을 출력합니다.
    print('Hello, {0}!'.format(name))  

# if 구문도 들여쓰기로 범위를 나타냅니다.
# sys.argv는 명령줄 매개변수를 나타내는 리스트 형식의 변수입니다.
if len(sys.argv) > 1:
    # if 구문의 조건이 참일 때
    # 변수는 정의하지 않고 곧바로 사용할 수 있습니다.
    name = sys.argv[1]
    # greet() 함수를 호출합니다.
    greet(name)
else:
    # if 구문의 조건이 거짓일 때
    # greet 함수를 호출합니다.
    greet('world')
</code>
</pre>

* if.py
<pre>
<code>
# 변수를 선언합니다.
a = 1

# if 구문으로 처리를 분기합니다.
if a == 1:
    # if 구문의 식이 참일 때 실행합니다.
    print('a is 1')
elif a == 2:
    # elif 절의 식이 참일 때 실행합니다.
    print('a is 2')
else:
    # 어떠한 조건해도 해당하지 않을 때 실행합니다.
    print('a is not 1 nor 2')

# 조건문을 한 줄로 적을 수 있지만 읽기 어려우므로 사용하지 않는 것이 좋습니다.
print('a is 1' if a == 1 else 'a is not 1')
</code>
</pre>

* for_and_while.py
<pre>
<code>
# 변수 x에 in의 오른쪽 리스트가 차례대로 들어갑니다.
# 따라서 블록 내부의 처리가 3번 반복됩니다.
for x in [1, 2, 3]:
    # 1, 2, 3이 차례대로 출력됩니다.
    print(x)  

# 횟수를 지정해서 반복할 때는 range()를 사용합니다
for i in range(10):
    # 0 9가 차례대로 출력됩니다.
    print(i)  

# for 구문으로 dict를 지정하면 키를 기반으로 순회합니다.
d = {'a': 1, 'b': 2}
for key in d:
    value = d[key]
    print(key, value)

# dict의 items() 메서드로 dict 키와 값을 순회합니다.
for key, value in d.items():
    print(key, value)

# while 구문으로 식이 참일 때 반복 처리합니다.
s = 1
while s < 1000:
    # # 1, 2, 4, 8, 16, 32, 64, 128, 256, 512가 차례대로 출력됩니다.
    print(s)
    s = s * 2
</code>
</pre>

* try_and_with.py
<pre>
<code>
d = {'a': 1, 'b': 2}
try:
    # 예외가 발생할 가능성이 있는 처리를 넣습니다.
    print(d['x'])
except KeyError:
    # try 절 내부에서 except 절에 작성된 예외(현재 예제에서는 KeyError)가 발생하면
    # except 절이 실행됩니다. 여기서는 키가 존재하지 않을 때의 처리 내용을 지정했습니다.
    print('x is not found')

# open() 함수의 반환값은 변수 f에 할당되며 with 블록 내부에서 사용합니다.
# 이렇게 사용하면 블록을 벗어날 때 f.close()가 자동으로 호출됩니다.
with open('index.html') as f:
    print(f.read())
</code>
</pre>

* def.py
<pre>
<code>
# add라는 이름의 함수를 정의합니다.
# 이 함수는 매개변수로 a와 b를 받고 더한 뒤 반환합니다.
def add(a, b):
    return a + b  # return 구문으로 값을 반환합니다.

# 함수를 호출할 때는 함수 이름 뒤에 괄호를 입력하고
# 내부에 매개변수를 지정합니다.
print(add(1, 2))  # 3라고 출력합니다.

# <매개변수>=<값>이라는 형태로도 매개변수를 지정할 수 있습니다.
# 이를 키워드 매개변수라고 합니다.
print(add(1, b=3))  # 4라고 출력합니다.
</code>
</pre>

* class.py
<pre>
<code>
# Rect라는 이름의 클래스를 지정합니다.
class Rect:
    # 인스턴스가 생성될 때 호출되는 특수한 메서드를 정의합니다.
    def __init__(self, width, height):
        self.width = width    # width 속성에 값을 할당합니다.
        self.height = height  # height 속성에 값을 할당합니다.
    # 사각형의 넓이를 계산하는 메서드를 정의합니다.
    def area(self):
        return self.width * self.height

r = Rect(100, 20)
print(r.width, r.height, r.area())   # 100 20 2000을 출력합니다.

# Rect를 상속받아 Square 클래스를 정의합니다.
class Square(Rect):
    def __init__(self, width):
        # 부모 클래스의 메서드를 호출합니다.
        super().__init__(width, width)
</code>
</pre>

* import.py
<pre>
<code>
# sys 모듈을 현재 이름 공간으로 읽어 들입니다.
import sys 

# datetime 모듈에서 date 클래스를 읽어 들입니다.
from datetime import date

# sys 모듈의 argv라는 변수로 명령줄 매개변수 리스트를 추출하고 출력합니다.
print(sys.argv)
# date 클래스의 today() 메서드로 현재 날짜를 추출합니다.
print(date.today())
</code>
</pre>

### 2-4. 웹 페이지 추출하기
* urlopen_encoding.py
<pre>
<code>
import sys
from urllib.request import urlopen
f = urlopen('http://www.hanbit.co.kr/store/books/full_book_list.html')

# HTTP 헤더를 기반으로 인코딩 방식을 추출합니다(명시돼 있지 않을 경우 utf-8을 사용하게 합니다).
encoding = f.info().get_content_charset(failobj="utf-8")
# 인코딩 방식을 표준 오류에 출력합니다.
print('encoding:', encoding, file=sys.stderr)

# 추출한 인코딩 방식으로 디코딩합니다.
text = f.read().decode(encoding)
# 웹 페이지의 내용을 표준 출력에 출력합니다.
print(text)
</code>
</pre>

* urlopen_meta.py
<pre>
<code>
import re
import sys
from urllib.request import urlopen

f = urlopen('http://www.hanbit.co.kr/store/books/full_book_list.html')
# bytes 자료형의 응답 본문을 일단 변수에 저장합니다.
bytes_content = f.read()  

# charset은 HTML의 앞부분에 적혀 있는 경우가 많으므로
# 응답 본문의 앞부분 1024바이트를 ASCII 문자로 디코딩해 둡니다.
# ASCII 범위 이위의 문자는 U+FFFD(REPLACEMENT CHARACTER)로 변환되어 예외가 발생하지 않습니다.
scanned_text = bytes_content[:1024].decode('ascii', errors='replace')

# 디코딩한 문자열에서 정규 표현식으로 charset 값을 추출합니다.
match = re.search(r'charset=["\']?([\w-]+)', scanned_text)
if match:
    encoding = match.group(1)
else:
    # charset이 명시돼 있지 않으면 UTF-8을 사용합니다.
    encoding = 'utf-8'

# 추출한 인코딩을 표준 오류에 출력합니다.
print('encoding:', encoding, file=sys.stderr)

# 추출한 인코딩으로 다시 디코딩합니다.
text = bytes_content.decode(encoding)
# 응답 본문을 표준 출력에 출력합니다.
print(text)
</code>
</pre>

### 2-5. 웹 페이지에서 데이터 추출하기
* scrape_re.py
<pre>
<code>
import re
from html import unescape

# 이전 절에서 다운로드한 파일을 열고 html이라는 변수에 저장합니다.
with open('dp.html') as f:
    html = f.read()

# re.findall()을 사용해 도서 하나에 해당하는 HTML을 추출합니다.
for partial_html in re.findall(r'<td class="left"><a.*?</td>', html, re.DOTALL):
    # 도서의 URL을 추출합니다.
    url = re.search(r'', partial_html).group(1)
    url = 'http://www.hanbit.co.kr' + url
    # 태그를 제거해서 도서의 제목을 추출합니다.
    title = re.sub(r'<.*?>', '', partial_html)
    title = unescape(title)
    print('url:', url)
    print('title:', title)
    print('---')
</code>
</pre>

* scrape_rss.py
<pre>
<code>
# ElementTree 모듈을 읽어 들입니다.
from xml.etree import ElementTree

# parse() 함수로 파일을 읽어 들이고 ElementTree 객체를 만듭니다.
tree = ElementTree.parse('rss.xml')

# getroot() 메서드로 XML의 루트 요소를 추출합니다.
root = tree.getroot()

# findall() 메서드로 요소 목록을 추출합니다.
# 태그를 찾습니다(자세한 내용은 RSS를 열어 참고해주세요).
for item in root.findall('channel/item/description/body/location/data'):
    # find() 메서드로 요소를 찾고 text 속성으로 값을 추출합니다.
    tm_ef = item.find('tmEf').text
    tmn = item.find('tmn').text
    tmx = item.find('tmx').text
    wf = item.find('wf').text
    print(tm_ef, tmn, tmx, wf) # 출력합니다.
</code>
</pre>

### 2-6. 데이터 저장하기
* save_csv_join.py
<pre>
<code>
# 첫 번째 줄에 헤더를 작성합니다.
print('rank,city,population')  

# join() 메서드의 매개변수로 전달한 list는 str이어야 하므로 주의해 주세요.
print(','.join(['1', '상하이', '24150000']))
print(','.join(['2', '카라치', '23500000']))
print(','.join(['3', '베이징', '21516000']))
print(','.join(['4', '텐진', '14722100']))
print(','.join(['5', '이스탄불', '14160467']))
</code>
</pre>

* save_csv.py
<pre>
<code>
import csv

# 파일을 엽니다. newline=''으로 줄바꿈 코드의 자동 변환을 제어합니다.
with open('top_cities.csv', 'w', newline='') as f:
    # csv.writer는 파일 객체를 매개변수로 지정합니다.
    writer = csv.writer(f)  
    # 첫 번째 줄에는 헤더를 작성합니다.
    writer.writerow(['rank', 'city', 'population'])  
    # writerows()에 리스트를 전달하면 여러 개의 값을 출력합니다.
    writer.writerows([
        [1, '상하이', 24150000],
        [2, '카라치', 23500000],
        [3, '베이징', 21516000],
        [4, '텐진', 14722100],
        [5, '이스탄불', 14160467],
    ])
</code>
</pre>

* save_csv_dict.py
<pre>
<code>
import csv

with open('top_cities.csv', 'w', newline='') as f:
    # 첫 번째 매개변수에 파일 객체
    # 두 번째 매개변수에 필드 이름 리스트를 지정합니다.
    writer = csv.DictWriter(f, ['rank', 'city', 'population'])
      # 첫 번째 줄에 헤더를 입력합니다.
    writer.writeheader()
    # writerows()로 여러 개의 데이터를 딕셔너리 형태로 작성합니다.
    writer.writerows([
        {'rank': 1, 'city': '상하이', 'population': 24150000},
        {'rank': 2, 'city': '카라치', 'population': 23500000},
        {'rank': 3, 'city': '베이징', 'population': 21516000},
        {'rank': 4, 'city': '텐진', 'population': 14722100},
        {'rank': 5, 'city': '이스탄불', 'population': 14160467},
    ])
</code>
</pre>

* save_json.py
<pre>
<code>
import json

cities = [
    {'rank': 1, 'city': '상하이', 'population': 24150000},
    {'rank': 2, 'city': '카라치', 'population': 23500000},
    {'rank': 3, 'city': '베이징', 'population': 21516000},
    {'rank': 4, 'city': '텐진', 'population': 14722100},
    {'rank': 5, 'city': '이스탄불', 'population': 14160467},
]

print(json.dumps(citles))
</code>
</pre>

* save_sqlite3.py
<pre>
<code>
import sqlite3

# top_cities.db 파일을 열고 연결을 변수에 저장합니다.
conn = sqlite3.connect('top_cities.db')

# 커서를 추출합니다.
c = conn.cursor()

# execute() 메서드로 SQL 구문을 실행합니다.
# 스크립트를 여러 번 사용해도 같은 결과를 출력할 수 있게 cities 테이블이 존재하는 경우 제거합니다.
c.execute('DROP TABLE IF EXISTS cities')
# cities 테이블을 생성합니다.
c.execute('''
    CREATE TABLE cities (
        rank integer,
        city text,
        population integer
    )
''')

# execute() 메서드의 두 번째 매개변수에는 파라미터를 지정할 수 있습니다.
# SQL 내부에서 파라미터로 변경할 부분(플레이스홀더)은 ?로 지정합니다.
c.execute('INSERT INTO cities VALUES (?, ?, ?)', (1, '상하이', 24150000))

# 파라미터가 딕셔너리일 때는 플레이스홀더를 :<이름> 형태로 지정합니다.
c.execute('INSERT INTO cities VALUES (:rank, :city, :population)',
          {'rank': 2, 'city': '카라치', 'population': 23500000})

# executemany() 메서드를 사용하면 여러 개의 파라미터를 리스트로 지정해서
# 여러 개(현재 예제에서는 3개)의 SQL 구문을 실행할 수 있습니다.
c.executemany('INSERT INTO cities VALUES (:rank, :city, :population)', [
    {'rank': 3, 'city': '베이징', 'population': 21516000},
    {'rank': 4, 'city': '텐진', 'population': 14722100},
    {'rank': 5, 'city': '이스탄불', 'population': 14160467},
])

# 변경사항을 커밋(저장)합니다.
conn.commit()

# 저장한 데이터를 추출합니다.
c.execute('SELECT * FROM cities')
# 쿼리의 결과는 fetchall() 메서드로 추출할 수 있습니다.
for row in c.fetchall():
    # 추출한 데이터를 출력합니다.
    print(row)

# 연결을 닫습니다.
conn.close()
</code>
</pre>

### 2-7. 파이썬으로 스크레이핑하는 흐름
* python_scraper.py
<pre>
<code>
import re
import sqlite3
from urllib.request import urlopen
from html import unescape

def main():
    """
    메인 처리입니다.
    fetch(), scrape(), save() 함수를 호출합니다.
    """
    html = fetch('http://www.hanbit.co.kr/store/books/full_book_list.html')
    books = scrape(html)
    save('books.db', books)

def fetch(url):
    """
    매개변수로 전달받을 url을 기반으로 웹 페이지를 추출합니다.
    웹 페이지의 인코딩 형식은 Content-Type 헤더를 기반으로 알아냅니다.
    반환값: str 자료형의 HTML
    """
    f = urlopen(url)
    # HTTP 헤더를 기반으로 인코딩 형식을 추출합니다.
    encoding = f.info().get_content_charset(failobj="utf-8")
    # 추출한 인코딩 형식을 기반으로 문자열을 디코딩합니다.
    html = f.read().decode(encoding)
    return html

def scrape(html):
    """
    매개변수 html로 받은 HTML을 기반으로 정규 표현식을 사용해 도서 정보를 추출합니다.
    반환값: 도서(dict) 리스트
    """
    books = []
    # re.findall()을 사용해 도서 하나에 해당하는 HTML을 추출합니다.
    for partial_html in re.findall(r'<td class="left"><a.*?</td>', html, re.DOTALL):
        # 도서의 URL을 추출합니다.
        url = re.search(r'', partial_html).group(1)
        url = 'http://www.hanbit.co.kr' + url
        # 태그를 제거해서 도서의 제목을 추출합니다.
        title = re.sub(r'<.*?>', '', partial_html)
        title = unescape(title)
        books.append({'url': url, 'title': title})
    
    return books

def save(db_path, books):
    """
    매개변수 books로 전달된 도서 목록을 SQLite 데이터베이스에 저장합니다.
    데이터베이스의 경로는 매개변수 dp_path로 지정합니다.
    반환값: None(없음)
    """
    # 데이터베이스를 열고 연결을 확립합니다.
    conn = sqlite3.connect(db_path)
    # 커서를 추출합니다.
    c = conn.cursor()
    # execute() 메서드로 SQL을 실행합니다.
    # 스크립트를 여러 번 실행할 수 있으므로 기존의 books 테이블을 제거합니다.
    c.execute('DROP TABLE IF EXISTS books')
    # books 테이블을 생성합니다.
    c.execute('''
        CREATE TABLE books (
            title text,
            url text
        )
    ''')
    # executemany() 메서드를 사용하면 매개변수로 리스트를 지정할 수 있습니다.
    c.executemany('INSERT INTO books VALUES (:title, :url)', books)
    # 변경사항을 커밋(저장)합니다.
    conn.commit()
    # 연결을 종료합니다.
    conn.close()

# python 명령어로 실행한 경우 main() 함수를 호출합니다.
# 이는 모듈로써 다른 파일에서 읽어 들였을 때 main() 함수가 호출되지 않게 하는 것입니다.
# 파이썬 프로그램의 일반적인 작성 방식입니다.
if __name__ == '__main__':
    main()
</code>
</pre>

 
