# Chapter 7. 크롤러의 지속적 운용과 관리
### 7-1. 크롤러를 서버에서 실행하기
지속적으로 실행할 크롤러는 서버에서 실행시키는 것이 좋다. 서버를 사용하면 스케줄을 지정해서 정기적으로 크롤러를 실행할 수 있게 된다. 
* 가상 서버 만들기
크롤러를 서버에서 실행하기 위해 Amazon EC2를 사용한다. 
  * Amazon EC2에서 가상 머신을 만드는 과정\
  AWS 계정 만들기 -> IAM 사용자 만들기 -> 키 페어 만들기 -> 가상 서버 만들고 실행하기\
  * AWS 계정 만들기 : https://aws.amazon.com/ko/ 에서 계정을 생성한다. 계정을 생성하면 매니지먼트 콘솔에 로그인 한다.
  * IAM 사용자 만들기 : AWS를 사용할 때 메일 주소와 비밀번호로 로그인 하는 것은 권장하지 않는다. 사용 권한이 제한된 IAM 사용자를 만들고 사용하는 것을 권장한다. 매니지먼트 콘솔에서 [Identity & Access Management] 링크 클릭 후 사용자를 만든다. 생성된 사용자를 클리학고, [Permission] 탭에서 사용자에게 권한을 부여한다.이후 해당 계정으로 로그인 한 뒤 키 페어를 생성한다.
  * 키 페어를 생성한 뒤 가상 서버를 실행한다. 실행한뒤 $ chmod 600 <다운로드 받은 키 페어의 경로> 로 퍼머션을 변경한다.
  <pre>
  <code>
  $ chmod 600 <다운로드 받은 키 페어의 경로>
  $ ssh -i <다운로드 받은 키 페어의 경로> ubuntu@<인스턴스의 IP 주소>
  ~/.ssh/config 에  ssh ec2 설정 입력
  
  # 초기 설정
  $ sudo dpkg-reconfigure tzdata
  $ sudo service crond restart
  $ sudo apt-get update
  $ sudo apt-get upgrade -y
  
  # 인스턴스 종료
  $ sudo shutdown -h now
  </code>
  </pre>

* 서버에 디플로이하기
로컬 머신에서 작성할 스크립트를 서버에서 실행하려면 서버에도 파이썬 실행 환경을 구성해야 하며, 스크립트를 업로드해야 한다. 
  * 파이썬 실행 환경 구축하기\
  API를 사용해 파이썬을 설치 -> venv 모듈로 가상 환경을 생성 -> 생성한 가상 환경에 접속
  <pre>
  <code>
  # 로컬 머신에소 조작
  $ pip freeze
  cssselect==0.9.1
  lxml==3.4.2
  requests==2.7.0
  이를 텍스트 파일로 저장
  
  # 로컬 머신에서 조작
  $ pip freeze > requirements.txt
  
  # 서버에서 조작 
  $ pip install -r requirements.txt
  $ sudo apt-get insrall -y libxml2-dev libxslt-dev libpython3-dev zlib1g-dev
  </code>
  </pre>
  
* 서버에 파일 업로드하기
  * rsync로 파일 업로드 하기
  <pre>
  <code>
  # 로컬 머신에서 조작
  $ sudo apt-get install -y rsync

  # 기본 명령어 형태
  $ rsync [옵션] <전송할 파일의 경로> <전송 대상 경로>
  $ rsync [옵션] <전송할 파일의 경로> [사용자@]<전송 대상 호스트>:<전송 대상 경로>
  </code>
  </pre>
  
  * rsync의 대표적인 옵션
  
  |옵션|설명|
  |---|---|
  |-a, --archive|아카이브 모드로 복사|
  |-v, --verboxe|실행 상태를 자세하게 출력|
  |-r, --recursive|디렉터리를 재귀적으로 복사|
  |-l, --links|심볼릭 링크를 심볼릭 링크로 복사|
  |-p, --perms|퍼머션을 유지한 상태로 복사|
  |-t, --times|파일 시간을 유지한 상태로 복사|
  |-g, --group|파일의 그룹을 유지한 상태로 복사|
  |-o, --owner|파일의 사용자를 유지한 상태로 복사|
  |-D|디바이스 파일 또는 특별한 파일을 그대로 복사|
  |--exclude=\<패턴>|\<패턴>에 맞는 파일을 복사하지 않게 함|
  |-c, --checksum|파일의 변경을 검출|
  
### 7-2. 크롤러를 정기적으로 실행하기
크롤러를 정기적으로 실행하면 변화가 있을 때 통지하거나 시간에 따른 데이터 변화를 확인할 수 있다. 리눅스에서 프로그램을 정기적으로 실행하기 위해 Cron이라는 소프트웨어를 사용한다.\
* Cron 설정
Cron은 시간, 날짜, 요일을 지정해서 프로그램을 실행할 때 사용하는 소프트웨어이다. 
* 오류 통지
서버에서 지동으로 실행되는 프로그램은 명령어를 직접 입력해서 실행하는 경우에 비해 오류가 났을 때 눈치채기 어렵다. 오류가 발생했을 때 메일로 통지하는 방법에 대한 설명이다.
  * Postfix 설치
  <pre>
  <code>
  $ sudo apt-get install -y postfix mailutils
  
  # 설치 시 설정
  General type of mail configuration>Satellite system, System mail names>기본상태, SMTP relay host>[smtp.gmail.com]:587
  
  # /etc/postfix/main.cf 설정 추가
  smtp_sasl_auth_enable = yes
  smtp_sasl_password_maps = hash:/etc/postfix/sasl_password
  smtp_sasl_security_options = noanonymous
  smtp_use_tls = yes
  
  # /etc/postfix/sasl_passwd 설정 추가
  [dmtp.gmail.com]:587 <Google 계정 사용자 이름>@gmail.com:<Google 계정 비밀번호>
  
  $ sudo chmod 600 /etc/postfix/sasl_passwd
  $ sudo postmap /etc/postfix/sasl_passwd
  
  $ sudo service postfix reload
  
  # 메일 테스트
  $ echo "test mail from postfix" | mail -s "Test" <자신의 메일 주소>
  </code>
  </pre>

### 7-3. 크롤링과 스크레이핑 분리하기
* 메시지 큐 RQ 사용 방법
메시지 큐는 큐를 사용해 메시지를 전달하는 방식이다. 일반적으로 메시지를 보내는 쪽과 받는 쪽은 서로 다른 스레드, 프로세스, 호스트이다. 메시지를 보내는 송신자는 비동기적으로 메시지를 큐에 송신하고, 메시지를 받는 수신자는 큐에 있는 메시지를 순차적으로 처리한다. 
  * Redis 설치와 실행
  <pre>
  <code>
  $ brew install redis
  $ redis-server
  $ sudo apt-get install -y redis-server
  
  # RQ 설치
  $ pip install rq
  </code>
  </pre>
  
  * 잡을 큐에 넣는 스크립트
  <pre>
  <code>
  from redis import Redis
  from rq import Queue
  from tasks import add

  # localhost의 TCP 포트 6379에 있는 Redis에 접속합니다.
  # 이러한 매개변수는 기본값이므로 생략해도 됩니다.
  conn = Redis('localhost', 6379)

  # default라는 이름의 Queue 객체를 추출합니다.
  # 이 이름도 기본값이므로 생략해도 됩니다
  q = Queue('default', connection=conn)

  # 함수와 매개변수를 지정하고 잡을 추가합니다.
  q.enqueue(add, 3, 4)
  </code>
  </pre>
  
* 메시지 큐로 연동하기
메시지 큐를 크롤링에 연동하면 Redis의 메모리 소비를 줄일 수 있고, 한 번 스크레이핑이 성공한 이후에도 HTML 이 남으므로 이후에 다시 스크레이핑할 수 있다. 또한 큐에 작업을 넣은 이후 HTML 에 가해진 변경을 워커에서 확인할 수 있다.
  * 크롤링 처리
  <pre>
  <code>
  import time
  import re
  import sys

  import requests
  import lxml.html
  from pymongo import MongoClient
  from redis import Redis
  from rq import Queue

  def main():
      """
      크롤러의 메인 처리
      """
      q = Queue(connection=Redis())
      # 로컬 호스트의 MongoDB에 접속
      client = MongoClient('localhost', 27017)
      # scraping 데이터베이스의 ebook_htmls 콜렉션을 추출합니다.
      collection = client.scraping.ebook_htmls
      # key로 빠르게 검색할 수 있게 유니크 인덱스를 생성합니다.
      collection.create_index('key', unique=True)

      session = requests.Session()
      # 목록 페이지를 추출합니다.
      response = requests.get('http://www.hanbit.co.kr/store/books/new_book_list.html')
      # 상세 페이지의 URL 목록을 추출합니다.
      urls = scrape_list_page(response)
      for url in urls:
          # URL로 키를 추출합니다.
          key = extract_key(url)
          # MongoDB에서 key에 해당하는 데이터를 검색합니다.
          ebook_html = collection.find_one({'key': key})
          # MongoDB에 존재하지 않는 경우에만 상세 페이지를 크롤링합니다.
          if not ebook_html:
              time.sleep(1)
              print('Fetching {0}'.format(url), file=sys.stderr)
              # 상세 페이지를 추출합니다.
              response = session.get(url)
              # HTML을 MongoDB에 저장합니다.
              collection.insert_one({
                  'url': url,
                  'key': key,
                  'html': response.content,
              })
              # 큐에 잡을 주가합니다.
              # result_ttl=0을 매개변수로 지정해서
              # 태스크의 반환값이 저장되지 않게 합니다.
              q.enqueue('scraper_tasks.scrape', key, result_ttl=0)

  def scrape_list_page(response):
      """
      목록 페이지의 Response에서 상세 페이지의 URL을 추출합니다.
      """
      root = lxml.html.fromstring(response.content)
      root.make_links_absolute(response.url)
      for a in root.cssselect('.view_box .book_tit a'):
          url = a.get('href')
          yield url

  def extract_key(url):
      """
      URL에서 키(URL 끝의 p_code)를 추출합니다.
      """
      m = re.search(r"p_code=(.+)", url)
      return m.group(1)

  if __name__ == '__main__':
      main()
  </code>
  </pre>
  
  * 스크레이핑 처리
  <pre>
  <code>
  import re
  import lxml.html
  from pymongo import MongoClient

  def scrape(key):
      """
      워커로 실행할 대상
      """
      # 로컬 호스트의 MongoDB에 접속합니다.
      client = MongoClient('localhost', 27017)

      # scraping 데이터베이스의 ebook_htmls 콜렉션을 추출합니다.
      html_collection = client.scraping.ebook_htmls

      # MongoDB에서 key에 해당하는 데이터를 찾습니다.
      ebook_html = html_collection.find_one({'key': key})
      ebook = scrape_detail_page(key, ebook_html['url'], ebook_html['html'])

      # ebooks 콜렉션을 추출합니다.
      ebook_collection = client.scraping.ebooks

      # key로 빠르게 검색할 수 있게 유니크 인덱스를 생성합니다.
      ebook_collection.create_index('key', unique=True)

      # ebook을 저장합니다.
      ebook_collection.insert_one(ebook)

  def scrape_detail_page(key, url, html):
      """
      상세 페이지의 Response에서 책 정보를 dict로 추출하기
      """
      root = lxml.html.fromstring(html)
      ebook = {
          'url': response.url,
          'key': key,
          'title': root.cssselect('.store_product_info_box h3')[0].text_content(),
          'price': root.cssselect('.pbr strong')[0].text_content(),
          'content': [normalize_spaces(p.text_content())
              for p in root.cssselect('#tabs_3 .hanbit_edit_view p')
              if normalize_spaces(p.text_content()) != ""]
      }
      return ebook

  def normalize_spaces(s):
      """
      연결돼 있는 공백을 하나의 공백으로 변경합니다.
      """
      return re.sub(r'\s+', ' ', s).strip()
  </code>
  </pre>
  
* 메시지 큐 운용하기

### 7-4. 크롤링 성능 향상과 비동기 처리
* 멀티 스레드와 멀티 프로세스
* 비동기 I/O를 사용해 효율적으로 크롤링하기

### 7-5. 클라우드 활용하기
* 클라우드의 장점
* AWS SDK 사용하기
* 클라우드 스토리지 사용하기
