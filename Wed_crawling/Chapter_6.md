# Chapter 6. Scrapy 프레임워크
### 6-1. Scrapy 개요
Scrapy는 크롤링/스크레이핑을 위한 파이썬 프레임워크이며, 풍부한 기능이 제공되므로 사용자는 페이지에서 데이터를 추출하는 본질적인 작업에만 집중할 수 있음.
* Scrapy 설치
<pre>
<code>
$ sudo apt-get install -y libssl-dev libffi-dev
$ pip install scrapy
$ scrapy version
</code>
</pre>
* Scrapinghub 블로그에서 글의 타이틀을 추출하는 Spider
<pre>
<code>
import scrapy
 
class BlogSpider(scrapy.Spider):
    # spider의 이름
    name = 'blogspider'

    # 크롤링을 시작할 URL 리스트
    start_urls = ['https://blog.scrapinghub.com']
 
    def parse(self, response):
        """
        최상위 페이지에서 카테고리 페이지의 링크를 추출합니다.
        """
        for url in response.css('ul li a::attr("href")').re('.*/tag/.*'):
            yield scrapy.Request(response.urljoin(url), self.parse_titles)

    def parse_titles(self, response):
        """
        카페고리 페이지에서 카테고리 타이틀을 모두 추출합니다.
        """
        for post_title in response.css('div.post-header > h2 > a::text').extract():
            yield {'title': post_title}

</code>
</pre>

### 6-2. Spider 만들고 실행하기
* Scrapy 프로젝트 만들기
<pre>
<code>
$ scrapy startproject myproject
$ cd myproject
</code>
</pre>
* Item 만들기
  * Items는 Spider가 추출할 데이터를 저장할 객체이며, Spider처럼 추출한 데이터를 저장하기 위한 dict를 사용해도 괜찮지만 Item을 사용하면 다음과 같은 장점이 발생함
    * 여러 종류의 데이터를 추출했을 때 클래스를 기반으로 객체를 판별할 수 있음
    * 미리 정의한 필드에 데이터를 입력하므로, 필드 이름을 잘못 적는 실수를 줄일 수 있음
    * 메서드를 정의할 수 있음
  * 뉴스 헤드라인을 저장할 Item 객체
  <pre>
  <code>
  # -*- coding: utf-8 -*-

  # Define here the models for your scraped items
  #
  # See documentation in:
  # http://doc.scrapy.org/en/latest/topics/items.html

  import scrapy


  class Headline(scrapy.Item):
      """
      뉴스 헤드라인을 나타내는 Item 객체
      """
      title = scrapy.Field()
      body = scrapy.Field()
  </code>
  </pre>
  
* Scrapy Shell로 인터랙티브하게 스크레이핑하기
  * Engadget의 기사 내용을 추출하는 Spider
  <pre>
  <code>
  import scrapy

  # Item의 Headline 클래스를 읽어 들입니다.
  from myproject.items import Headline

  class NewsSpider(scrapy.Spider):
      name = 'news'
      # 크롤링 대상 도메인 리스트
      allowed_domains = ['engadget.com']
      # 크롤링을 시작할 URL 리스트
      start_urls = ['http://engadget.com/']
      def parse(self, response):
          """
          메인 페이지의 토픽 목록에서 링크를 추출하고 출력합니다.
          """
          link = response.css('a.o-hit__link::attr("href")').extract()
          for url in link:
              # 광고 페이지 제외
              if url.find("products") == 1: 
                  continue
              # 의미 없는 페이지 제외
              if url == "#": 
                  continue
              # 기사 페이지
              yield scrapy.Request(response.urljoin(url), self.parse_topics)

      def parse_topics(self, response):
          item = Headline()
          item['title'] = response.css('head title::text').extract_first()
          item['body'] = " ".join(response.css('.o-article_block p')\
              .xpath('string()')\
              .extract())
          yield item
  </code>
  </pre>

* Spider 실행하기
<pre>
<code>
$ scrapy crawl news -o mews.jl
</code>
</pre>
  * Spider를 실행할 때 지정할 수 있는 옵션
  
  |옵션|설명|
  |---|---|
  |--help, -h|도움말 출력|
  |-a \<NAME>=\<VALUE>|Spider의 __init__() 메서드에 키워드 매개변수를 전달|
  |--outout=\<FILE>, -o \<FILE>|추출한 item을 저장할 파일 경로 지정|
  |--output-format=\<FORMAT>, -t \<FORMAT>|추출한 item을 저장할 때의 형식 지정|
  |--logfile=\<FILE>|로그 출력 대상 경로를 지정|
  |--loglevel=\<LEVEL>, -L \<LEVEL>|로그 레벨을 지정|
  |--nolog|로그 출력을 하지 않음|
  |--profile=\<FILE>|프로필 통계를 출력할 경로 지정|
  |--pidfile=\<FILE>|프로세스 ID를 지정한 경로의 파일에 출력|
  |--set=\<NAME>=\<VALUE>, -s \<NAME>=\<VALUE>|설정을 지정|
  |--pdb|예외가 발생했을 때 pdb로 디버그를 시작|

### 6-3. 실질적인 크롤링
* 크롤링으로 링크 순회하기
  * Engadget 뉴스 기사의 토픽을 추출하는 CrawlSpider
  <pre>
  <code>
  from scrapy.spiders import CrawlSpider, Rule
  from scrapy.linkextractors import LinkExtractor

  # Item의 Headline 클래스를 읽어 들입니다.
  from myproject.items import Headline

  class NewsSpider(scrapy.Spider):
      name = 'news'
      # 크롤링 대상 도메인 리스트
      allowed_domains = ['engadget.com']
      # 크롤링을 시작할 URL 리스트
      start_urls = ['http://engadget.com/']
      # 링크 순회를 위한 규칙 리스트
      rules = [
          # 토픽 페이지를 추출한 뒤 응답을 parse_topics() 메서드에 전달합니다.
          Rule(LinkExtractor(allow=r'/\d{4}/\d{2}/\d{2}/.+$'), callback='parse_topics'),
      ]

      def parse_topics(self, response):
          item = Headline()
          item['title'] = response.css('head title::text').extract_first()
          item['body'] = " ".join(response.css('.o-article_block p')\
              .xpath('string()')\
              .extract())
          yield item
  </code>
  </pre>

* XML 사이트맵을 사용해 크롤링하기
  * SitemapSpider 만들고 실행하기 : 한빛네트워크의 도서를 추출하는 SitemapSpider
  <pre>
  <code>
  from scrapy.spiders import SitemapSpider

  class HanbitSpider(SitemapSpider):
      name = "hanbit"
      allowed_domains = ["hanbit.co.kr"]
      # XML 사이트맵을 지정합니다.
      # robots.txt에서 Sitemap 디렉티브를 사용하고 있다면
      # robots.txt의 링크를 지정해도 됩니다.
      sitemap_urls = [
          "http://hanbit.co.kr/sitemap.xml",
      ]
      # 사이트맵 디렉티브에서 순회할 링크의 정규 표현식을 지정합니다.
      # sitemap_follow를 지정하지 않으면 모든 링크를 순회합니다.
      sitemap_follow = [
          r'post-2015-',
      ]
      # 사이트맵에 포함돼 있는 URL을 처리할 콜백을 지정합니다.
      # 규칙은 (<정규 표현식>, <처리할 콜백 함수>) 형태의 튜플을 지정합니다.
      # sitemap_rules를 지정하지 않으면 모든 URL을 parse() 메서드에 전달합니다.
      sitemap_rules = [
          (r'/2015/\d\d/\d\d/', 'parse_book'),
      ]

      def parse_post(self, response):
          # 책 페이지에서 제목을 추출합니다.
          yield {
              'title': response.css('.store_product_info_box h3::text').extract_first(),
          }
  </code>
  </pre>

### 6-4. 추출한 데이터 처리하기
* Item Pipeline 개요
  * Item Pipeline는 Spider에서 추출한 Item에 대해 임의의 처리를 할 수 있게 해주는 컴포넌트
  * Pipeline은 특정한 이름의 메서드를 가진 클래스이며, 일반적으로 프로젝트 내부에 존재하는 pipelines.py에 클래스를 정의함
  * 기본 형태 : process_item(self, item, spider)
  * 특정 시점에 호출하기
    * open_spider(self, spider) : Spider(spider)를 실행할 때 호출
    * close_spider(self, spider) : Spider(spider)가 종료될 때 호출
    * from_crawler(cls, crawler) : 이 클래스 매서드가 존재하면 Pipeline이 인스턴스화 될 때 호출

* 데이터 검증
  * Item을 검증하는 Pipeline
  <pre>
  <code>
  from scrapy.exceptions import DropItem
  
  class ValidationPipeline(odject):
    """
    Item을 검증하는 Pipeline
    """
    def process_item(self, item, spider):
      if not item['title']:
      # title 필드가 추출되지 않으면 제거합니다.
      # DropItem()의 매개변수로 제거 이류룰 나타내는 메시지를 입력합니다.
      raise DropItem("Missing title')
    # title 필드가 제대로 추출된 경우
    return item
  </code>
  </pre>
  
* MongoDB에 데이터 저장하기
<pre>
<code>
from pymongo import MongoClient
class MongoPipeline(object):
    """
    Itemd을 MongoDB에 저장하는 Pipeline
    """
    def open_spider(self, spider):
        """
        Spider를 시작할 때 MongoDB에 접속합니다.
        """
        # 호스트와 포트를 지정해서 클라이언트를 생성합니다.
        self.client = MongoClient('localhost', 27017)
         # scraping-book 데이터베이스를 추출합니다.
        self.db = self.client['scraping-book']
        # items 콜렉션을 추출합니다.
        self.collection = self.db['items']
    
    def close_spider(self, spider):
        """
        Spider가 종료될 때 MongoDB 접속을 끊습니다.
        """
        self.client.close()
    def process_item(self, item, spider):
        """
        Item을 콜렉션에 추가합니다.
        """
        # insert_one()의 매개변수에는 item을 깊은 복사를 통해 전달합니다.
        self.collection.insert_one(dict(item))
        return item
</code>
</pre>

* MySQL에 데이터 저장하기
<pre>
<code>
import MySQLdb
class MySQLPipeline(object):
    """
    Item을 MySQL에 저장하는 Pipeline
    """
    
    def open_spider(self, spider):
        """
        Spider를 시작할 때 MySQL 서버에 접속합니다.
        items 테이블이 존재하지 않으면 생성합니다.
        """
        # settings.py에서 설정을 읽어 들입니다.
        settings = spider.settings
        params = {
            'host': settings.get('MYSQL_HOST', 'localhost'),
            'db': settings.get('MYSQL_DATABASE', 'scraping'),
            'user': settings.get('MYSQL_USER', ''),
            'passwd': settings.get('MYSQL_PASSWORD', ''),
            'charset': settings.get('MYSQL_CHARSET', 'utf8mb4'),
        }
        # MySQL 서버에 접속합니다.
        self.conn = MySQLdb.connect(**params) 
        # 커서를 추출합니다.
        self.c = self.conn.cursor() 
        # items 테이블이 존재하지 않으면 생성합니다.
        self.c.execute('''
            CREATE TABLE IF NOT EXISTS items (
                id INTEGER NOT NULL AUTO_INCREMENT,
                title CHAR(200) NOT NULL,
                PRIMARY KEY (id)
            )
        ''')
        # 변경을 커밋합니다.
        self.conn.commit()
    
    def close_spider(self, spider):
        """
        Spider가 종료될 때 MySQL 서버와의 접속을 끊습니다.
        """
        self.conn.close()
    def process_item(self, item, spider):
        """
        Item을 items 테이블에 삽입합니다.
        """
        self.c.execute('INSERT INTO items (title) VALUES (%(title)s)', dict(item))
        self.conn.commit()
        return item
</code>
</pre>

### 6-5. Scrapy 설정
* 설정 방법
  * 명령줄 옵션으로 설정하기 : $ scrapy crawl wiredjp -s DOWNLOAD_DELAY=3
  * Spider에 직접 설정하기 : Spider의 custom_settings 속성을 사용해 직접 설정을 작성
  * 프로젝트 설정 파일로 설정하기 : 프로젝트의 settinds.py에 설정을 작성
  * 서브 명령어의 기본값 설정하기
  * 글로벌 기본값 설정하기 : scrapy.settings.default_settings 모듈
  
* 크롤링 대상에 폐를 끼치지 않기 위한 설정 항목
  * DOWNLOAD_DELAY : 같은 웹사이트에 요청을 여러 번 보낼 때 요청 대기 시간을 설정, 1.0이상 권장
  * RANDOMIZE_DOWNLOAD_DELAY : 웹 페이지의 다운로드 간격을 무작위로 설정할지 지정
  * ROBOTSTXT_OBEY : 웹사이트에 robots.txt에 따를지 설정, True/False
  
* 병렬 처리와 관련된 설정 항목
  * CONCURRENT_REQUESTS : 기본값(16), 동시에 병렬 처리할 요청의 최대 개수
  * CONCURRENT_REQUESTS_PER_DOMAIN : 기본값(8), 특정 웹사이트의 도메인에 동시 병렬 처리할 요청의 최대 개수
  * CONCURRENT_REQUESTS_PER_IP : 기본값(0), 특정 웹사이트의 IP에 동시 병렬 처리할 요청의 최대 개수
  
* HTTP 요청과 관련된 설정
  * USER_AGENT : 기본값('Scrapy/\<VERSION> (+http://scrapy.org)'), HTTP 요청에 포함되는 User-Agent 헤더의 값을 지정
  * COOKIES_ENABLED : 기본값(True), Cookie를 활성화할지 나타냄
  * COOKIES_DEBUG : 기본값(False), True로 지정하면 Cookie 값을 로그에 출력
  * REFERER_ENABLED : 기본값(True), 요청에 Referer헤더를 자동으로 포함할지 나타냄
  * DEFAULT_REQUEST_HEADERS : HTTP 요청에 포함될 헤더를 dict로 지정
  
* HTTP 캐시 설정 항목
  * HTTPCACHE_ENABLED : 기본값(False), True로 설정하면 HTTP 캐시를 활성화
  * HTTPCACHE_EXPIRATION_SECS : 기본값(0), 캐시의 유효 기간을 초 단위로 지정
  * HTTPCACHE_DIR : 기본값('httpcache'), 캐시를 저장할 디렉터리 경로를 나타냄
  * HTTPCACHE_IGNORE_HTTP_CODES : 기본값([]), 응답을 캐시하지 않을 HTTP 상태 코드를 리스트로 지정
  * HTTPCACHE_IGNORE_SCHEMES : 기본값(['file']), 응답을 캐시하지 않을 URL 스키마를 리스트로 지정
  * HTTPCACHE_IGNORE_MISSING : 기본값(False), True로 지정하면 요청이 캐시돼 있지 않은 경우 요청을 무시
  * HTTPCACHE_POLICY : 캐시 정책을 나타내는 클래스
  
* 오류 처리와 관련된 설정
  * RETRY_ENABLED : 기본값(True), 타임아웃 또는 HTTP 상태코드 500 등이 발생했을 때 자동으로 재시도할지 지정
  * RETRY_TIMES : 기본값(2), 재시도 횟수의 최댓값을 지정
  * RETRY_HTTP_CODES : 기본값([500, 502, 503, 504, 408]), 재시도 대상 HTTP 상태 코드를 리스트로 지정
  * HTTPERROR_ALLOWED_CODES : 기본값([]), 오류로 취급하지 않을 HTTP 상태 코드를 리스트로 지정
  * HTTPERROR_ALLOWED_ALL : 기본값(False), True로 지정하면 모든 상태 코드를 오류로 취급하지 않고, Spider의 콜백 함수에서 처리할 수 있게 됨
  
* 프락시 사용하기

### 6-6. Scrapy 확장하기
* 다운로드 처리 확장하기
  * Downloader Middleware는 인터페이스이며, 다음과 같은 3개의 메서드 중에 1개 이상을 가진 클래스의 객체를 의미
    * process_request(self, request, spider)
    * process_response(self, request, response, spider)
    * process_exception(self, request, exception, spider)
    * 기본 Downloader Middleware
    
    |클래스 이름|설명|순서|
    |---|---|---|
    |RobotTxtMiddleware|robots.txt를 기반으로 요청을 필터링|100|
    |HttpAuthMiddleware|Basic 인증과 관련된 헤더를 추가|300|
    |DownloadTimeoutMiddleware|다운로드 타임아웃을 설정|350|
    |UserAgentMiddleware|User-Agent 헤더를 추가|400|
    |RetryMiddleware|오류가 발생했을 떄 재시도|500|
    |DafaultHeadersMiddleware|기본 HTTP 헤더를 추가|550|
    |AjaxCrawMiddleware|Ajax 크롤링하는 페이지를 다시 크롤링|560|
    |MetaRefreshMiddleware|meta refresh 태그로 리다이렉트 처리|580|
    |HttpCompressionMiddleware|압축된 응답을 처리|590|
    |RedirectMiddleware|리다이렉트를 처리|600|
    |CookiesMiddleware|Cookle를 처리|700|
    |HttpProxyMiddleware|HTTP 프락시를 설정|750|
    |ChunkedTransferMiddleware|Chunked 상태의 응답을 디코딩|830|
    |DownloaderStats|Downloader 통계를 수집|850|
    |HttpCacheMiddleware|HTTP 캐시를 처리|900|
    
* Spider의 동작 확장하기
  * Spider Middleware를 사용하면 Spider의 콜뱍 함수 처리를 래핑해서 확장할 수 있으며, 다음과 같은 4개의 메서드 중에 1개 이상을 가진 클래스의 객체를 의미
    * process_spider_input(self, response, spider)
    * process_spider_output(self, response, result, spider)
    * process_spider_exception(self, response, exception, spider)
    * process_start_requests(self, start_requests, spider)
    * 기본 Spider Middleware
    
    
    |클래스 이름|설명|순서|
    |---|---|---|
    |HttpErrorMiddleware|응답이 오류일 때 제거|50|
    |OffsiteMiddleware|allowed_domain 이외의 도메인을 크롤링하지 않게 함|500|
    |RefererMiddleware|Referer 헤더를 추가|700|
    |UrlLengthMiddleware|너무 긴 URL을 제거|800|
    |DepthMiddleware|링크 순회하는 깊이와 관련된 처리를 함|900|
    
### 6-7. 크롤링으로 데이터 수집하고 활용하기
* 음식점 정보 수집
  * 서울 음식점 정보를 수집하는 Spider
  <pre>
  <code>
  import re
  from scrapy.spiders import CrawlSpider, Rule
  from scrapy.linkextractors import LinkExtractor
  from myproject.items import Restaurant

  class VisitSeoulSpider(CrawlSpider):
      name = "visitseoul"
      allowed_domains = ["korean.visitseoul.net"]
      start_urls = ['http://korean.visitseoul.net/eat?curPage=1']
      rules = [
          # 9페이지까지 순회합니다.
          # 정규 표현식 \d를 \d+로 지정하면 모든 페이지를 순회합니다.
          Rule(LinkExtractor(allow=r'/eat\?curPage=\d$')),
          # 음식점 상세 페이지를 분석합니다.
          Rule(LinkExtractor(allow=r'/eat/\w+/\d+'),
               callback='parse_restaurant'),
      ]

      def parse_restaurant(self, response):
          """
          음식점 정보 페이지를 파싱합니다.
          """
          # 정보를 추출합니다.
          name = response.css("#pageheader h3")\
              .xpath("string()").extract_first().strip()
          address = response.css("dt:contains('주소') + dd")\
              .xpath("string()").extract_first().strip()
          phone = response.css("dt:contains('전화번호') + dd")\
              .xpath("string()").extract_first().strip()
          station = response.css("th:contains('지하철') + td")\
              .xpath("string()").extract_first().strip()

          # 위도 경도를 추출합니다.
          try:
              scripts = response.css("script:contains('var lat')").xpath("string()").extract_first()
              latitude = re.findall(r"var lat = '(.+)'", scripts)[0]
              longitude = re.findall(r"var lng = '(.+)'", scripts)[0]
          except Exception as exception:
              print("예외 발생")
              print(exception)
              print()

          # 음식점 객체를 생성합니다.
          item = Restaurant(
              name=name,
              address=address,
              phone=phone,
              latitude=latitude,
              longitude=longitude,
              station=station
          )
          yield item
  </code>
  </pre>

* 불특정 다수의 웹사이트 크롤링하기
  * 불특정 다수의 페이지를 크롤링하는 Spider
  <pre>
  <code>
  import scrapy
  from myproject.items import Page
  from myproject.utils import get_content

  class BroadSpider(scrapy.Spider):
      name = "broad"
      start_urls = (
          # 하테나 북마크 엔트리 페이지
          'http://b.hatena.ne.jp/entrylist',
      )

      def parse(self, response):
          """
          하테나 북마크의 엔트리 페이지를 파싱합니다.
          """
          # 각각의 웹 페이지 링크를 추출합니다.
          for url in response.css('a.entry-link::attr("href")').extract():
              # parse_page() 메서드를 콜백 함수로 지정합니다.
              yield scrapy.Request(url, callback=self.parse_page)
          # of 뒤의 숫자를 두 자리로 지정해 5페이지(첫 페이지, 20, 40, 60, 80)만 추출하게 합니다.
          url_more = response.css('a::attr("href")').re_first(r'.*\?of=\d{2}$')
          if url_more:
              # url_more의 값은 /entrylist로 시작하는 상대 URL이므로
              # response.urljoiin() 메서드를 사용해 절대 URL로 변경합니다.
              # 콜백 함수를 지정하지 않았으므로 응답은 기본적으로
              # parse() 메서드에서 처리하게 됩니다.
              yield scrapy.Request(response.urljoin(url_more))

      def parse_page(self, response):
          """
          각 페이지를 파싱합니다.
          """
          # utils.py에 정의돼 있는 get_content() 함수로 타이틀과 본문을 추출합니다.
          title, content = get_content(response.text)
          # Page 객체로 반환합니다.
          yield Page(url=response.url, title=title, content=content)
  </code>
  </pre>

### 6-8. 이미지 수집과 활용
* 플리커에서 이미지 수집하기
  * 플리커에서 이미지를 다운로드하는 Spider
  <pre>
  <code>
  import os
  from urllib.parse import urlencode
  import scrapy
  class FlickrSpider(scrapy.Spider):
      name = "flickr"
      # Files Pipeline으로 다운로드하는 이미지 파일은 allowed_domains에
      # 제한을 받으므로 allowed_domains에 'staticflickr.com'을 추가해야 합니다
      allowed_domains = ["api.flickr.com"]

      # 키워드 매개변수로 Spider 매개변수 값을 받습니다.
      def __init__(self, text='sushi'):
          # 부모 클래스의 __init__()을 실행합니다.
          super().__init__()
          # 환경변수와 Spider 매개변수 값을 사용해 start_urls를 조합합니다.
          # urlencode() 함수는 매개변수로 지정한 dict의 키와 값을 URI 인코드해서
          # key1=value1&key2=value2라는 문자열로 반환해 줍니다.
          self.start_urls = [
              'https://api.flickr.com/services/rest/?' + urlencode({
                  'method': 'flickr.photos.search',
                  'api_key': os.environ['FLICKR_API_KEY'],
                  'text': text,
                  'sort': 'relevance',
                  # CC BY 2.0, CC BY-SA 2.0, CC0를 지정합니다.
                  'license': '4,5,9',  
              }),
          ]
      def parse(self, response):
          """
          API의 응답을 파싱해서 file_urls라는 키를 포함한 dict를 생성하고 yield합니다.
          """
          for photo in response.css('photo'):
              yield {'file_urls': [flickr_photo_url(photo)]}

  def flickr_photo_url(photo):
      """
      플리커 사진 URL을 조합합니다.
      참고: https://www.flickr.com/services/api/misc.urls.html
      """
      # 이 경우는 XPath가 CSS 선택자보다 쉬우므로 XPath를 사용하겠습니
      return 'https://farm{farm}.staticflickr.com/{server}/{id}_{secret}_{size}.jpg'.format(
          farm=photo.xpath('@farm').extract_first(),
          server=photo.xpath('@server').extract_first(),
          id=photo.xpath('@id').extract_first(),
          secret=photo.xpath('@secret').extract_first(),
          size='b',
      )
  </code>
  </pre>
  
* OpenCV로 얼굴 이미지 추출하기
  * OpenCV 설치하기
  <pre>
  <code>
  $ brew insrall homebrew/science/opencv3 --with-python3
  
  $ sudo add-apt-repository ppa:orangain/opencv
  $ sudo apt-get update
  $ sudo apt-get install -y python3-opencv opencv-data
  
  $ ln -s /usr/lib/python3.4/dist-packages/cv2.cpython-34m.so scraping/lib/python3.4/site-packages/
  
  $ pip install numpy
  
  $ python -c 'import cv2'
  </code>
  </pre>

  * 이미지에서 얼굴 검출하기
  <pre>
  <code>
  $ wget https://upload.wikimedia.org/wikipedia/commons/5/55/Grace_Hopper.jpg
  
  # 인터랙티브 셸
  >>> import cv2
  >>> classifier = cv2.CascadeClassifier('/usr/share/opencv/haarcascades/haarcascade_frontalface_alt.xml')
  >>> inage = cd2.imread('Grace_Hopper.jpg')
  >>> faces = classifier.detectMultiScale(image)
  >>> for x, y, w, h in faces:
  >>> cv2.imweite('faces.jpg', image)
  >>> cv2.imshow('Faces', image)
  >>> cv2.destroyAllwindows()
  </code>
  </pre>
  
  * 이미지에서 얼굴을 추출하고 저장하는 스크립트
  <pre>
  <code>
  import sys
  import os
  import cv2

  try:
      # 얼굴 검출 전용 특징량 파일의 경로
      cascade_path = sys.argv[1]
  except IndexError:
      # 명령어 매개변수가 부족한 경우에는 사용법을 출력하고 곧바로 종료합니다.
      print('Usage: python extract_faces.py CASCADE_PATH IMAGE_PATH...', file=sys.stderr)
      exit(1)

  # 얼굴 이미지 출력 대상 디렉터리가 존재하지 않으면 생성해 둡니다.
  output_dir = 'faces'
  if not os.path.exists(output_dir):
      os.makedirs(output_dir)

  # 특징량 파일이 존재하는지 확인합니다.
  assert os.path.exists(cascade_path)
  # 특징량 파일의 경로를 지정해 분석 객체를 생성합니다.
  classifier = cv2.CascadeClassifier(cascade_path)

  # 두 번째 이후의 매개변수 파일 경로를 반복 처리합니다.
  for image_path in sys.argv[2:]:
      print('Processing', image_path, file=sys.stderr)

      # 명령어 매개변수에서 얻은 경로의 이미지 파일을 읽어 들입니다.
      image = cv2.imread(image_path)
      # 얼굴 검출을 빠르게 할 수 있게 이미지를 그레이스케일로 변환합니다.
      gray_image = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
      # 얼굴을 검출합니다.
      faces = classifier.detectMultiScale(gray_image)

      # 이미지 파일 이름의 확장자를 제거합니다.
      image_name = os.path.splitext(os.path.basename(image_path))[0]

      # 추출된 얼굴의 리스트를 반복 처리합니다.
      # i는 0부터 시작되는 순번입니다.
      for i, (x, y, w, h) in enumerate(faces):
          # 얼굴 부분만 자릅니다.
          face_image = image[y:y + h, x: x + w]
          # 출력 대상 파일 경로를 생성합니다.
          output_path = os.path.join(output_dir, '{0}_{1}.jpg'.format(image_name, i))
          # 얼굴 이미지를 저장합니다.
          cv2.imwrite(output_path, face_image)
  </code>
  </pre>
  
  * 실행
  <pre>
  <code>
  $ python extract_faces.py /usr/share/opencv/haarcascades/haarcascade_frontalface_alt.xml myproject/images/full/*
  </code>
  </pre>
