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
