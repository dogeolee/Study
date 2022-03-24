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
* MySQL에 데이터 저장하기

### 6-5. Scrapy 설정
* 설정 방법
* 크롤링 대상에 폐를 끼치지 않기 위한 설정 항목
* 병렬 처리와 관련된 설정 항목
* HTTP 요청과 관련된 설정
* HTTP 캐시 설정 항목
* 오류 처리와 관련된 설정
* 프락시 사용하기

### 6-6. Scrapy 확장하기
* 다운로드 처리 확장하기
* Spider의 동작 확장하기

### 6-7. 크롤링으로 데이터 수집하고 활용하기
* 음식점 정보 수집
* 불특정 다수의 웹사이트 크롤링하기

### 6-8. 이미지 수집과 활용
* 플리커에서 이미지 수집하기
* OpenCV로 얼굴 이미지 추출하기
