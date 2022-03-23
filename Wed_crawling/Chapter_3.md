# Chaper 3. 주요 라이브러리 활용

### 3-1. 라이브러리 설치
* pip 으로 설치하기
  * PyPI에 공개돼 있는 라이브러리를 설치할 때는 pip 라는 도구를 사용

  <pre>
  <code>
  $ pip install <라이브러리 이름>
  $ pip3 install <라이브러리 이름>
  </code>
  </pre>


### 3.2. 웹 페이지 간단하게 추출하기(Requests)
<pre>
<code>
$ pip install requests

>>> import requests # 라이브러리 호출
>>> r = requests.get('https://hanbit.co.kr') # get() 함수로 웹 페이지 추출
>>> r.status_code # status_code 속성으로 HTTP 상테 코드 확인
>>> r.headers['content-type'] # headers 속성으로 HTTP 헤더를 딕셔너리로 추출
>>> r.encoding  # encoding 속성으로 HTTP 헤더를 기반으로 인코딩을 추출
>>> r.text # text 속성으로 str 자료형으로 디코딩된 응답 본문 추출
>>> r.content # content 속성으로 bytes 자료형 응답 본문 추출
>>> r.json # json 형식의 응답을 간단하게 디코드
>>> r = requests.post('http://httpbin.org/post', data={'key1': 'value1'}) # post 메서드로 전송
>>> r = requests.get('https://httpbin.org/get', headers={'user-agent': 'my-crawler/1.0 (+foo@example.com)'}) # HTTP 헤더 추가 저장
>>> r = requests.get('https://api.github.com/user', auth=('<github의 사용자 ID>', '<github의 비밀번호>') # Basic 인증은 키워드 매개변수 auth로 지정
>>> r = requests.get('https://httpbin.org/get',params={'key1': 'value1'}) # URL 매개변수 params로 지정

>>> s = requests.Session() # HTTP 헤더를 여러 번 사용할 때는 Session 객체를 사용
>>> s.headers.update({'user-agent': 'my-crawler/1.0 (+foo@example.com)'})
</code>
</pre>

### 3.3. HTML 스크레이핑
* XPath와 CSS 선택자 비교

|대상 요소|XPath|CSS|
|---|---|---|
|title 요소|//title|title|
|body 요소의 후선 중 h1 요소|//body//h1|body h1|
|body 요소의 자식 중 h1 요소|//body/h1|body > h1|
|body 요소 내부의 모든 자식 요소|//body/*|body > *|
|id 속성이 "main"인 요소|id("main") 또는 //*[@id="main"]|#main|
|class 속성으로 "active"를 포함하고 있는 요소|li[@class and contains(concat(' ', ormalize-space(@class), ' '), ' active')]|li.active|
|type 속성이 "text"인 input 요소|//input[@type="text"]|input[type="text"]|
|href 속성이 "https://"로 시작하는 요소|//a[starts-with(@href, "https://")]|a[href^="https://"]|
|src 속성이 ".jpg"로 끝나는 img 요소|//img[ends-with(@src, ".jpg")]|img[src$=".jpg"]|
|요소 내부에 "개요"라는 텍스트 노드가 포함된 h2 요소|//h2[contains(.,"개요")]|h2:contains("개요")]|
|요소 바로 아래에 "개요"라는 텍스트가 있는 h2 요소|//h2[text()="개요"]|불가능|

* lxml로 스크레이핑하기
  * 설치
  <pre>
  <code>
  $ sudo apt-get install -y libxml2-dev libxslt-dev libpython3-dev zlib1g-dev
  $ pip install lxml
  $ pip install cssselect
  </code>
  </pre>
 
  * scrape_by_lxml.py
  <pre>
  <code>
  import lxml.html

  # HTML 파일을 읽어 들이고, getroot() 메서드로 HtmlElement 객체를 생성합니다.
  tree = lxml.html.parse('full_book_list.html')
  html = tree.getroot() 

  # cssselect() 메서드로 a 요소의 리스트를 추출하고 반복을 돌립니다.
  for a in html.cssselect('a'):
      # href 속성과 글자를 추출합니다.
      print(a.get('href'), a.text)
  </code>
  </pre>
  
* Beautiful Soup로 스크레이핑하기
  * 설치
  <pre>
  <code>
  $ pip install beautifulsoup4
  </code>
  </pre>
  
  * scrape_by_bs4.py
  <pre>
  <code>
  from bs4 import BeautifulSoup

  # HTML 파일을 읽어 들이고 BeautifulSoup 객체를 생성합니다.
  with open('full_book_list.html') as f:
      soup = BeautifulSoup(f, 'html.parser')

  # find_all() 메서드로 a 요소를 추출하고 반복을 돌립니다.
  for a in soup.find_all('a'):
      # href 속성과 글자를 추출합니다.
      print(a.get('href'), a.text)
  </code>
  </pre>

### 3.4. RSS 스크레이핑
* 설치
<pre>
<code>
$ pip install feedparser
</code>
</pre>

* scrape_by_feedparser.py
<pre>
<code>
import feedparser

# 알라딘 도서 RSS를 읽어 들입니다.
d = feedparser.parse('http://www.aladin.co.kr/rss/special_new/351')

# 항목을 순회합니다.
for entry in d.entries:
    print('이름:', entry.title)
    print('타이틀:', entry.title)
    print()
</code>
</pre>


### 3.5. 데이터베이스에 저장하기
* MySQL에 데이터 저장하기
  * MySQL은 오픈소스 관계형 데이터베이스이며 SQLite와 다르게 클라이언트/서버형 아키텍처를 채용
  * 스크레이핑한 데이터를 저장할 때 SQL 구문을 활용하여 유연하게 데이터를 추출 가능
  * 설치
  <pre>
  <code>
  $ sudo apt-get install -y mysql-server libmysqlclient-dev
  $ mysqld --version
  
  $ sudo service mysql status
  $ sudo service mysql start
  
  $ mysql -u root -p
  $ pip install mysqlclient
  </code>
  </pre>
  
  * save_mysql.py
  <pre>
  <code>
  import MySQLdb

  # MySQL 서버에 접속하고 연결을 변수에 저장합니다.
  # 사용자 이름과 비밀번호를 지정한 뒤 scraping 데이터베이스를 사용(USE)합니다.
  # 접속에 사용할 문자 코드는 utf8mb4로 지정합니다.
  conn = MySQLdb.connect(db='scraping', user='scraper', passwd='password', charset='utf8mb4')

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
  # SQL 내부에서 파라미터로 변경할 부분(플레이스홀더)은 %s로 지정합니다.
  c.execute('INSERT INTO cities VALUES (%s, %s, %s)', (1, '상하이', 24150000))

  # 파라미터가 딕셔너리일 때는 플레이스홀더를 %(<이름>)s 형태로 지정합니다.
  c.execute('INSERT INTO cities VALUES (%(rank)s, %(city)s, %(population)s)',
            {'rank': 2, 'city': '카라치', 'population': 23500000})

  # executemany() 메서드를 사용하면 여러 개의 파라미터를 리스트로 지정해서
  # 여러 개(현재 예제에서는 3개)의 SQL 구문을 실행할 수 있습니다.
  c.executemany('INSERT INTO cities VALUES (%(rank)s, %(city)s, %(population)s)', [
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
  
* MongoDB에 데이터 저장하기
  * NoSQL의 일종으로, 문서형이라고 부르는 데이터베이스
  * 유연한 데이터 구조, 높은 쓰기 성능, 사용하기 쉽다는 특징
  * 데이터 구조는 계층 구조를 가지고 있으며, 하나의 데이터베이스는 여러 개의 콜렉션을 가지며, 하나의 콜렉션은 여러 개의 문서를 갖는다.
  * 설치
  <pre>
  <code>
  $ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
  $ echo "deb http://repo.mongodb.org/apt/ubuntu "$(lsb_release -sc)"/mongodb-org/3.0 multiverse" | sudo tee /etc/apt/source.list.d/mongodb-org3.0.list
  $ sudo apt-get update
  $ sudo apt-get install -y mongodb-org
  
  $ sudo service mongod status
  $ pip install pymongo
  </code>
  </pre>
  
  * save_mongo.py
  <pre>
  <code>
  import lxml.html
  from pymongo import MongoClient

  # HTML 파일을 읽어 들이고 
  # getroot() 메서드를 사용해 HtmlElement 객체를 추출합니다.
  tree = lxml.html.parse('full_book_list.html')
  html = tree.getroot()

  client = MongoClient('localhost', 27017)
  db = client.scraping  # scraping 데이터베이스를 추출합니다.
  collection = db.links  # links 콜렉션을 추출합니다.

  # 스크립트를 여러 번 사용해도 같은 결과를 출력할 수 있게 콜렉션의 문서를 제거합니다.
  collection.delete_many({})

  # cssselect() 메서드로 a 요소의 목록을 추출합니다.
  for a in html.cssselect('a'):
      # href 속성과 링크의 글자를 추출해서 저장합니다.
      collection.insert_one({
          'url': a.get('href'),
          'title': a.text.strip(),
      })

  # 콜렉션의 모든 문서를 _id 순서로 정렬해서 추출합니다.
  for link in collection.find().sort('_id'):
      print(link['_id'], link['url'], link['title'])
  </code>
  </pre>
  

### 3.6. 크롤러와 URL
* URL의 구조
예시) http://example.com/main/index?q=python#lead

|http://|example.com|/main/index|?q=python|#lead|
|---|---|---|---|---|
|schema|authority|path|query|flagment|

|URL 구성요소|설명|
|---|---|
|스키마|http 또는 https와 같은 프로토콜을 나타냄|
|어서리티|// 뒤에 나오는 일반적인 호스트 이름을 나타냄. 사용자 이름, 비밀전호, 포트 번호 등을 포함하는 경우도 존재|
|경로|/로 시작하는 해당 호스트 내부에서의 리소스 경로를 나타냄|
|쿼리|? 뒤에 나오는 경로와는 다른 방법으로 리소스를 표현하는 방법. 존재하지 않는 경우도 있음|
|플래그먼트|# 뒤에 나오는 리소스 내부의 특정 부분등을 나타냄. 존재하지 않는 경우도 있음|

 ### 3-7. 파이썬을 크롤러 만들기
 * 목록 페이지에서 URL 목록 추출하기
   * python_crawler_1.py
   <pre>
   <code>
   import requests
   import lxml.html

   response = requests.get('http://www.hanbit.co.kr/store/books/new_book_list.html')
   root = lxml.html.fromstring(response.content)
   for a in root.cssselect('.view_box a'):
       url = a.get('href')
       print(url)
   </code>
   </pre>
   
   * python_crawler_2.py
   <pre>
   <code>
   import requests
   import lxml.html

   response = requests.get('http://www.hanbit.co.kr/store/books/new_book_list.html')
   root = lxml.html.fromstring(response.content)

   # 모든 링크를 절대 URL로 변환합니다.
   root.make_links_absolute(response.url)

   # 선택자를 추가해서 명확한 선택을 할 수 있게 합니다.
   for a in root.cssselect('.view_box .book_tit a'):
       url = a.get('href')
       print(url)
   </code>
   </pre>
   
   * python_crawler_3.py
   <pre>
   <code>
   import requests
   import lxml.html
   def main():
       """
       크롤러의 메인 처리
       """
       # 여러 페이지에서 크롤링할 것이므로 Session을 사용합니다.
       session = requests.Session()  
       # scrape_list_page() 함수를 호출해서 제너레이터를 추출합니다.
       response = session.get('http://www.hanbit.co.kr/store/books/new_book_list.html')
       urls = scrape_list_page(response)
       # 제너레이터는 list처럼 사용할 수 있습니다.
       for url in urls:
           print(url)

   def scrape_list_page(response):
       root = lxml.html.fromstring(response.content)
       root.make_links_absolute(response.url)
       for a in root.cssselect('.view_box .book_tit a'):
           url = a.get('href')
           # yield 구문으로 제너레이터의 요소 반환
           yield url

   if __name__ == '__main__':
       main()
   </code>
   </pre>
 
* 상세 페이지에서 스크레이핑하기
  * python_crawler_4.py
   <pre>
   <code>
   import requests
   import lxml.html

   def main():
       # 여러 페이지에서 크롤링할 것이므로 Session을 사용합니다.
       session = requests.Session()  
       response = session.get('http://www.hanbit.co.kr/store/books/new_book_list.html')
       urls = scrape_list_page(response)
       for url in urls:
           response = session.get(url)  # Session을 사용해 상세 페이지를 추출합니다.
           ebook = scrape_detail_page(response)  # 상세 페이지에서 상세 정보를 추출합니다.
           print(ebook)  # 책 관련 정보를 출력합니다.
           break  # 책 한 권이 제대로 되는지 확인하고 종료합니다.

   def scrape_list_page(response):
       root = lxml.html.fromstring(response.content)
       root.make_links_absolute(response.url)
       for a in root.cssselect('.view_box .book_tit a'):
           url = a.get('href')
           yield url

   def scrape_detail_page(response):
       """
       상세 페이지의 Response에서 책 정보를 dict로 추출합니다.
       """
       root = lxml.html.fromstring(response.content)
       ebook = {
           'url': response.url,
           'title': root.cssselect('.store_product_info_box h3')[0].text_content(),
           'price': root.cssselect('.pbr strong')[0].text_content(),
           'content': [p.text_content()\
               for p in root.cssselect('#tabs_3 .hanbit_edit_view p')]
       }
       return ebook

   if __name__ == '__main__':
       main()
   </code>
   </pre>
   
   * python_crawler_5.py
   <pre>
   <code>
   import requests
   import lxml.html

   def main():
       # 여러 페이지에서 크롤링할 것이므로 Session을 사용합니다.
       session = requests.Session()  
       response = session.get('http://www.hanbit.co.kr/store/books/new_book_list.html')
       urls = scrape_list_page(response)
       for url in urls:
           response = session.get(url)  # Session을 사용해 상세 페이지를 추출합니다.
           ebook = scrape_detail_page(response)  # 상세 페이지에서 상세 정보를 추출합니다.
           print(ebook)  # 책 관련 정보를 출력합니다.
           break  # 책 한 권이 제대로 되는지 확인하고 종료합니다.

   def scrape_list_page(response):
       root = lxml.html.fromstring(response.content)
       root.make_links_absolute(response.url)
       for a in root.cssselect('.view_box .book_tit a'):
           url = a.get('href')
           yield url

   def scrape_detail_page(response):
       """
       상세 페이지의 Response에서 책 정보를 dict로 추출합니다.
       """
       root = lxml.html.fromstring(response.content)
       ebook = {
           'url': response.url,
           'title': root.cssselect('.store_product_info_box h3')[0].text_content(),
           'price': root.cssselect('.pbr strong')[0].text_content(),
           'content': [normalize_spaces(p.text_content())
               for p in root.cssselect('#tabs_3 .hanbit_edit_view p')
               if normalize_spaces(p.text_content()) != '']
       }
       return ebook

   def normalize_spaces(s):
       """
       연결돼 있는 공백을 하나의 공백으로 변경합니다.
       """
       return re.sub(r'\s+', ' ', s).strip()

   if __name__ == '__main__':
       main()
   </code>
   </pre>
   
* 상세 페이지 크롤링하기
<pre>
<code>
import time # time 모듈을 임포트합니다.
import re 
import requests
import lxml.html

def main():
    session = requests.Session()
    response = session.get('http://www.hanbit.co.kr/store/books/new_book_list.html')
    urls = scrape_list_page(response)
    for url in urls:
        time.sleep(1) # 1초 동안 휴식합니다.
        response = session.get(url)
        ebook = scrape_detail_page(response)
        print(ebook)

def scrape_list_page(response):
    root = lxml.html.fromstring(response.content)
    root.make_links_absolute(response.url)
    for a in root.cssselect('.view_box .book_tit a'):
        url = a.get('href')
        yield url

def scrape_detail_page(response):
    """
    상세 페이지의 Response에서 책 정보를 dict로 추출합니다.
    """
    root = lxml.html.fromstring(response.content)
    ebook = {
        'url': response.url,
        'title': root.cssselect('.store_product_info_box h3')[0].text_content(),
        'price': root.cssselect('.pbr strong')[0].text_content(),
        'content': [normalize_spaces(p.text_content())
            for p in root.cssselect('#tabs_3 .hanbit_edit_view p')
            if normalize_spaces(p.text_content()) != '']
    }
    return ebook

def normalize_spaces(s):
    """
    연결돼 있는 공백을 하나의 공백으로 변경합니다.
    """
    return re.sub(r'\s+', ' ', s).strip()

if __name__ == '__main__':
    main()
</code>
</pre>

* 스크레이핑한 데이터 저장하기
<pre>
<code>
import time
import re 
import requests
import lxml.html
from pymongo import MongoClient

def main():
    """
    크롤러의 메인 처리
    """
    # 크롤러 호스트의 MongoDB에 접속합니다.
    client = MongoClient('localhost', 27017)
    # scraping 데이터베이스의 ebooks 콜렉션
    collection = client.scraping.ebooks 
    # 데이터를 식별할 수 있는 유일키를 저장할 key 필드에 인덱스를 생성합니다.
    collection.create_index('key', unique=True)

    # 목록 페이지를 추출합니다.
    response = requests.get('http://www.hanbit.co.kr/store/books/new_book_list.html')
    # 상세 페이지의 URL 목록을 추출합니다.
    urls = scrape_list_page(response)
    for url in urls:
        # URL로 키를 추출합니다.
        key = extract_key(url)
        # MongoDB에서 key에 해당하는 데이터를 검색합니다.
        ebook = collection.find_one({'key': key})
        # MongoDB에 존재하지 않는 경우만 상세 페이지를 크롤링합니다.
        if not ebook:
            time.sleep(1)
            response = requests.get(url)
            ebook = scrape_detail_page(response)
            # 책 정보를 MongoDB에 저장합니다.
            collection.insert_one(ebook)
        # 책 정보를 출력합니다.
        print(ebook)

def scrape_list_page(response):
    """
    목록 페이지의 Response에서 상세 페이지의 URL을 추출합니다.
    """
    root = lxml.html.fromstring(response.content)
    root.make_links_absolute(response.url)
    for a in root.cssselect('.view_box .book_tit a'):
        url = a.get('href')
        yield url

def scrape_detail_page(response):
    """
    상세 페이지의 Response에서 책 정보를 dict로 추출합니다.
    """
    root = lxml.html.fromstring(response.content)
    ebook = {
        'url': response.url,
        'key': extract_key(response.url),
        'title': root.cssselect('.store_product_info_box h3')[0].text_content(),
        'price': root.cssselect('.pbr strong')[0].text_content(),
        'content': "생략"
    }
    return ebook

def extract_key(url):
    """
    URL에서 키(URL 끝의 p_code)를 추출합니다.
    """
    m = re.search(r"p_code=(.+)", url)
    return m.group(1)

def normalize_spaces(s):
    """
    연결돼 있는 공백을 하나의 공백으로 변경합니다.
    """
    return re.sub(r'\s+', ' ', s).strip()
    
if __name__ == '__main__':
    main()
</code>
</pre>
