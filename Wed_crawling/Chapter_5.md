# Chapter 5. 크롤링/스크레이핑 실전과 데이터 활용
### 5-1. 데이터 세트 추출과 활용
* 위키백과 데이터 세트 
<pre>
<code>
# 다운로드
$ wget http://dumps.wikimedia.org/kowiki/20150801/kowiki-20170801-pages-articles.xml.bz2

# 암축해제
$ bzcat jawiki-20150805-pages-articles1.xml.bz2 | less

# 간단한 문장 추출을 위한 WikiExtractor 다운로드
$ wget https://github.com/attardi/wikiextractor/raw/master/WikiExtractor.py

# --no-templates : 페이지 앞의 템플릿을 생략
# -o : 출력 대상 디렉터리를 지정
# -b : 분할할 파일의 크기를 지정
$ python WikiExtractor.py --no-templates -o articles -b 100M kowiki-20170805pages-articles.xml
</code>
</pre>

* 자연어 처리를 사용한 빈출단어 추출
  * KoNLPy 설치
  <pre>
  <code>
  $ sudo apt-get install g++ openjdk-7-jdk python-dev python3-dev
  $ pip install konlpy
  $ pip install jpype1
  </code>
  </pre>
  
  * KoNLPy로 위키백과 문장에서 빈출 단어 추출하기
  <pre>
  <code>
  import sys
  import os
  from glob import glob
  from collections import Counter
  from konlpy.tag import Kkma

  def main():
      """
      명령라인 매개변수로 지정한
      디렉터리 내부의 파일을 읽어 들이고
      빈출 단어를 출력합니다.
      """
      # 명령어의 첫 번째 매개변수로
      # WikiExtractor의 출력 디렉터리를 지정합니다.
      input_dir = sys.argv[1]
      kkma = Kkma()
      # 단어의 빈도를 저장하기 위한 Counter 객체를 생성합니다.
      # Counter 클래스는 dict를 상속받는 클래스입니다.
      frequency = Counter()
      count_proccessed = 0
      # glob()으로 와일드카드 매치 파일 목록을 추출하고
      # 매치한 모든 파일을 처리합니다.
      for path in glob(os.path.join(input_dir, '*', 'wiki_*')):
          print('Processing {0}...'.format(path), file=sys.stderr)
          # 파일을 엽니다.
          with open(path) as file:
              # 파일 내부의 모든 기사에 반복을 돌립니다.
              for content in iter_docs(file):
                  # 페이지에서 명사 리스트를 추출합니다.
                  tokens = get_tokens(kkma, content)
                  # Counter의 update() 메서드로 리스트 등의 반복 가능 객체를 지정하면
                  # 리스트에 포함된 값의 출현 빈도를 세어줍니다.
                  frequency.update(tokens)
                  # 10,000개의 글을 읽을 때마다 간단하게 출력합니다.
                  count_proccessed += 1
                  if count_proccessed % 10000 == 0:
                      print('{0} documents were processed.'
                          .format(count_proccessed),file=sys.stderr)

      # 모든 기사의 처리가 끝나면 상위 30개의 단어를 출력합니다
      for token, count in frequency.most_common(30):
          print(token, count)

  def iter_docs(file):
      """
      파일 객체를 읽어 들이고
      기사의 내용(시작 태그 <doc>와 종료 태그 </doc> 사이의 텍스트)를 꺼내는
      제너레이터 함수
      """
      for line in file:
          if line.startswith('<doc '):
              # 시작 태그가 찾아지면 버퍼를 초기화합니다.
              buffer = []
          elif line.startswith('</doc>'):
              # 종료 태그가 찾아지면 버퍼의 내용을 결합한 뒤 yield합니다.
              content = ''.join(buffer)
              yield content
          else:
              # 시작 태그/종료 태그 이외의 줄은 버퍼에 추가합니다.
              buffer.append(line)

  def get_tokens(kkma, content):
      """
      문장 내부에 출현한 명사 리스트를 추출하는 함수
      """
      # 명사를 저장할 리스트입니다.
      tokens = []
      node = kkma.pos(content)
      for (taeso, pumsa) in node:
          # 고유 명사와 일반 명사만 추출합니다.
          if pumsa in ('NNG', 'NNP'):
              tokens.append(taeso)
      return tokens

  if __name__ == '__main__':
      main()
  </code>
  </pre>
  
### 5-2. API로 데이터 수집하고 활용하기
* Requests-OAuthlib을 이용한 타임라인 추출
<pre>
<code>
# Requests-OAuthlib 설치
$ pip install requests-oauthlib

# rest_api_with_requests_oauthlib.py
import os
from requests_oauthlib import OAuth1Session

# 환경변수에서 인증 정보를 추출합니다.
CONSUMER_KEY = os.environ['CONSUMER_KEY']
CONSUMER_SECRET = os.environ['CONSUMER_SECRET']
ACCESS_TOKEN = os.environ['ACCESS_TOKEN']
ACCESS_TOKEN_SECRET = os.environ['ACCESS_TOKEN_SECRET']

# 인증 정보를 사용해 OAuth1Session 객체를 생성합니다.
twitter = OAuth1Session(CONSUMER_KEY,
                        client_secret=CONSUMER_SECRET,
                        resource_owner_key=ACCESS_TOKEN,
                        resource_owner_secret=ACCESS_TOKEN_SECRET)

# 사용자의 타임라인을 추출합니다.
response = twitter.get('https://api.twitter.com/1.1/statuses/home_timeline.json')

# API 응답이 JSON 형식의 문자열이므로 response.json()으로 파싱합니다.
# status는 트윗(Twitter API에서는 Status라고 부릅니다)를 나타내는 dict입니다.
for status in response.json():
    # 사용자 이름과 트윗을 출력합니다.
    print('@' + status['user']['screen_name'], status['text'])
</code>
</pre>

* Tweepy를 사용해 타임라인 추출하기
<pre>
<code>
import os
# pip install tweepy
import tweepy

# 환경변수에서 인증 정보를 추출합니다.
CONSUMER_KEY = os.environ['CONSUMER_KEY']
CONSUMER_SECRET = os.environ['CONSUMER_SECRET']
ACCESS_TOKEN = os.environ['ACCESS_TOKEN']
ACCESS_TOKEN_SECRET = os.environ['ACCESS_TOKEN_SECRET']

# 인증 정보를 설정합니다.
auth = tweepy.OAuthHandler(CONSUMER_KEY, CONSUMER_SECRET)
auth.set_access_token(ACCESS_TOKEN, ACCESS_TOKEN_SECRET)

# API 클라이언트를 생성합니다.
api = tweepy.API(auth)

# 사용자의 타임라인을 추출합니다.
public_tweets = api.home_timeline()
for status in public_tweets:
    # 사용자 이름과 트윗을 출력합니다.
    print('@' + status.user.screen_name, status.text)
</code>
</pre>

* Tweepy로 Streaming API 사용하기
<pre>
<code>
import os
import tweepy

# 환경변수에서 인증 정보를 추출합니다.
CONSUMER_KEY = os.environ['CONSUMER_KEY']
CONSUMER_SECRET = os.environ['CONSUMER_SECRET']
ACCESS_TOKEN = os.environ['ACCESS_TOKEN']
ACCESS_TOKEN_SECRET = os.environ['ACCESS_TOKEN_SECRET']

# 인증 정보를 설정합니다.
auth = tweepy.OAuthHandler(CONSUMER_KEY, CONSUMER_SECRET)
auth.set_access_token(ACCESS_TOKEN, ACCESS_TOKEN_SECRET)
class MyStreamListener(tweepy.StreamListener):
    """
    Streaming API로 추출한 트윗을 처리하는 클래스입니다.
    """
    def on_status(self, status):
        """
        트윗을 받을 때 호출되는 메서드
        매개변수로 트윗을 나타내는 Status 객체가 전달됩니다.
        """
        print('@' + status.author.screen_name, status.text)
# 인증 정보와 StreamListener를 지정해서 Stream 객체를 추출합니다.
stream = tweepy.Stream(auth, MyStreamListener())

# 공개돼 있는 트윗을 샘플링한 스트림을 받습니다.
# 키워드 매개변수인 languages로 한국어 트윗만 추출합니다
stream.sample(languages=['ko'])
</code>
</pre>

* Google API Client for Python 사용하기
  * 유튜브 동영상 검색하기
  <pre>
  <code>
  import os

  # pip install google-api-python-client
  from apiclient.discovery import build

  # 환경변수에서 API 키 추출하기
  YOUTUBE_API_KEY = os.environ['YOUTUBE_API_KEY']

  # YouTube API 클라이언트를 생성합니다.
  # build() 함수의 첫 번째 매개변수에는 API 이름
  # 두 번째 매개변수에는 API 버전을 지정합니다.
  # 키워드 매개변수 developerKey에는 API 키를 지정합니다.
  # 이 함수는 내부적으로 https://www.googleapis.com/discovery/v1/apis/youtube/v3/rest라는
  # URL에 접근하고 API 리소스와 메서드 정보를 추출합니다.
  youtube = build('youtube', 'v3', developerKey=YOUTUBE_API_KEY)

  # 키워드 매개변수로 매개변수를 지정하고
  # search.list 메서드를 호출합니다.
  # list() 메서드를 실행하면 googleapiclient.http.HttpRequest가 반환됩니다. 
  # execute() 메서드를 실행하면 실제 HTTP 요청이 보내지며, API 응답이 반환됩니다.
  search_response = youtube.search().list(
      part='snippet',
      q='요리',
      type='video',
  ).execute()

  # search_response는 API 응답을 JSON으로 나타낸 dict 객체입니다.
  for item in search_response['items']:
      # 동영상 제목을 출력합니다.
      print(item['snippet']['title'])
  </code>
  </pre>
  
  * 동영상의 상세한 메타 정보 추출하기
  <pre>
  <code>
  $ curl "https://www.googleapis.com/youtube/v3/video?key=<API 키>&id=muxH23R0DT0&part=snippet,statistics
  </code>
  </pre>
  
  * 동영상 정보를 MongoDB에 저장하고 검색하기
  <pre>
  <code>
  import os
  import sys

  from apiclient.discovery import build
  from pymongo import MongoClient, DESCENDING

  # 환경변수에서 API 키를 추출합니다.
  YOUTUBE_API_KEY = os.environ['YOUTUBE_API_KEY']

  def main():
      """
      메인 처리
      """
      # MongoDB 클라이언트 객체를 생성합니다.
      mongo_client = MongoClient('localhost', 27017)
      # youtube 데이터베이스의 videos 콜렉션을 추출합니다.
      collection = mongo_client.youtube.videos
      # 기존의 모든 문서를 제거합니다.
      collection.delete_many({})

      # 동영상을 검색하고, 페이지 단위로 아이템 목록을 저장합니다.
      for items_per_page in search_videos('요리'):
          save_to_mongodb(collection, items_per_page)

      # 뷰 수가 높은 동영상을 출력합니다.
      show_top_videos(collection)

  def search_videos(query, max_pages=5):
      """
      동영상을 검색하고, 페이지 단위로 list를 yield합니다.
      """
      # YouTube의 API 클라이언트 생성하기
      youtube = build('youtube', 'v3', developerKey=YOUTUBE_API_KEY)  
      # search.list 메서드로 처음 페이지 추출을 위한 요청 전송하기
      search_request = youtube.search().list(
          part='id',  # search.list에서 동영상 ID만 추출해도 괜찮음
          q=query,
          type='video',
          maxResults=50,  # 1페이지에 최대 50개의 동영상 추출
      )
      # 요청이 성공하고 페이지 수가 max_pages보다 작을 때 반복
      # 페이지 수를 제한하는 것은 실행 시간이 너무 길어지는 것을 막기 위해서입니다.
      # 더 많은 페이지를 요청해도 상관없습니다
      i = 0
      while search_request and i < max_pages:
          # 요청을 전송합니다.
          search_response = search_request.execute()
              # 동영상 ID의 리스트를 추출합니다.
              video_ids = [item['id']['videoId'] for item in search_response['items']]
          # videos.list 메서드로 동영상의 상세 정보를 추출합니다.
          videos_response = youtube.videos().list(
              part='snippet,statistics',
              id=','.join(video_ids)
          ).execute()
          # 현재 페이지 내부의 아이템을 yield합니다.
          yield videos_response['items']

          # list_next() 메서드로 다음 페이지를 추출하기 위한 요청을 보냅니다.
          search_request = youtube.search().list_next(search_request, search_response)
          i += 1

  def save_to_mongodb(collection, items):
      """
      MongoDB에 아이템을 저장합니다.
      """
      # MongoDB에 저장하기 전에 이후에 사용하기 쉽게 아이템을 가공합니다.
      for item in items:
          # 각 아이템의 id 속성을 _id 속성으로 사용합니다.
          item['_id'] = item['id']  
          # statistics에 포함된 viewCount 속성 등은 문자열이므로 숫자로 변환합니다.
          for key, value in item['statistics'].items():
              item['statistics'][key] = int(value)

      # 콜렉션에 추가합니다.
      result = collection.insert_many(items)
      print('Inserted {0} documents'.format(len(result.inserted_ids)), file=sys.stderr)

  def show_top_videos(collection):
      """
      MongoDB의 콜렉션 내부에서 뷰 수를 기준으로 상위 5개를 출력합니다.
      """
      for item in collection.find().sort('statistics.viewCount', DESCENDING).limit(5):
          print(item['statistics']['viewCount'], item['snippet']['title'])

  if __name__ == '__main__':
      main()
  </code>
  </pre>
  
### 5-3. 시계열 데이터 수집하고 활용하기
* pandas 와 CSV 파일
  * pandas 설치
  <pre>
  <code>
  $ pip install pandas
  </code>
  </pre>
  
  * read.csv() 함수에 지정할 수 있는 대표적인 키워드 매개변수
  
  |키워드 매개변수|설명|
  |---|---|
  |encoding|파일의 인코딩을 지정|
  |header|헤더로 사용할 줄 번호를 지정|
  |names|열 이름 목록을 지정|
  |skipinitialspace|True로 지정하면, 구분 문자 뒤에 이어지는 공백을 무시|
  |index_col|인덱스로 사용할 열의 번호를 지정|
  |parse_dates|True로 지정하면 인덱스에 사용한 열이 날짜일 경우 파싱|
  |date_parser|날짜를 파싱할 함수를 지정|
  |na_values|NaN으로 간주할 문자열 목록을 지정|
  
* 그래프로 시각화하기
  * matplotlib 설치
  <pre>
  <code>
  $ sudo apt-get build-dep -y python3-matplotlib
  $ sudo apt-get install -y fonts-migmix
  $ pip install matplotlib
  </code>
  </pre>
  
  * 다양한 매개변수를 지정해서 그래프 그리기
  <pre>
  <code>
  import matplotlib

  # 렌더링 백엔드로 데스크톱 환경이 필요 없는 Agg를 사용합니다.
  matplotlib.use('Agg')

  # 한국어를 렌더링할 수 있게 폰트를 지정합니다.
  # macOS와 우분투 모두 정상적으로 출력하도록 2개의 폰트를 지정했습니다.
  # 기본 상태에서는 한국어가 □로 출력됩니다.
  matplotlib.rcParams['font.sans-serif'] = 'NanumGothic,AppleGothic'
  import matplotlib.pyplot as plt

  # plot()의 세 번째 매개변수로 계열 스타일을 나타내는 문자열을 지정합니다.
  # 'b'는 파란색, 'x'는 × 표시 마커, '-'는 마커를 실선으로 연결하라는 의미입니다.
  # 키워드 매개변수 label로 지정한 계열의 이름은 범례로 사용됩니다.
  plt.plot([1, 2, 3, 4, 5], [1, 2, 3, 4, 5], 'bx-', label='첫 번째 함수')

  # 'r'은 붉은색,'o'는 ○ 표시 마커, '--'는 점선을 의미합니다.
  plt.plot([1, 2, 3, 4, 5], [1, 4, 9, 16, 25], 'ro--', label='두 번째 함수')
  # xlabel() 함수로 X축의 레이블을 지정합니다.
  plt.xlabel('X 값')
  # ylabel() 함수로 Y축의 레이블을 지정합니다.
  plt.ylabel('Y 값')
  # title() 함수로 그래프의 제목을 지정합니다.
  plt.title('matplotlib 샘플')
  # legend() 함수로 범례를 출력합니다. loc='best'는 적당한 위치에 출력하라는 의미입니다.
  plt.legend(loc='best')

  # X축 범위를 0~6으로 지정합니다. ylim() 함수를 사용하면 Y축 범위를 지정할 수 있습니다.
  plt.xlim(0, 6)

  # 그래프를 그리고 파일로 저장합니다.
  plt.savefig('advanced_graph.png', dpi=300)
  </code>
  </pre>
  
  * 데이터 시각화하기
  <pre>
  <code>
  from datetime import datetime
  import pandas as pd
  import matplotlib

  matplotlib.use('Agg') 
  matplotlib.rcParams['font.sans-serif'] = 'NanumGothic,AppleGothic' 
  import matplotlib.pyplot as plt

  def main():
      # 1981년과 2014년 사이의 환율과 고용률을 출력해 봅니다. 
      # 조금 이해하기 쉽게 Pandas 대신 기본 숫자 비교와 문자열 비교를 사용해 봤습니다.
      # 환율 정보 읽어 들이기
      df_exchange = pd.read_csv('DEXKOUS.csv', header=1, 
          names=['DATE', 'DEXKOUS'], skipinitialspace=True, index_col=0)
      years = {}
      output = []
      for index in df_exchange.index:
          year = int(index.split('-')[0])
          if (year not in years) and (1981 < year < 2014):
              if df_exchange.DEXKOUS[index] != ".":
                  years[year] = True
                  output.append([year, float(df_exchange.DEXKOUS[index])])
      df_exchange = pd.DataFrame(output)

      # 고용률 통계를 구합니다.
      df_jobs = pd.read_excel('gugik.xlsx') 
      output = []
      stacked = df_jobs.stack()[7]
      for index in stacked.index:
          try:
              if 1981 <= int(index) <= 2014:
                  output.append([int(index), float(stacked[index])])
          except:
              pass
      s_jobs = pd.DataFrame(output)

      # 첫 번째 그래프 그리기
      plt.subplot(2, 1, 1)
      plt.plot(df_exchange[0], df_exchange[1], label='원/달러') 
      plt.xlim(1981, 2014) # X축의 범위를 설정합니다.
      plt.ylim(500, 2500)
      plt.legend(loc='best')

      # 두 번째 그래프 그리기
      print(s_jobs)
      plt.subplot(2, 1, 2) # 3 1 の3 のサブプロットを作成。 
      plt.plot(s_jobs[0], s_jobs[1], label='고용률(%)') 
      plt.xlim(1981, 2014) # X축의 범위를 설정합니다.
      plt.ylim(0, 100) # Y축의 범위를 설정합니다.
      plt.legend(loc='best')
      plt.savefig('historical_data.png', dpi=300) # 이미지를 저장합니다.

  if __name__ == '__main__': 
      main()
  </code>
  </pre>
  
### 5-4. 열린 데이터 수집과 활용
* PDF 에서 데이터 추출하기
  * PDFMiner.six로 PDF에서 텍스트 추출하기
  <pre>
  <code>
  # 설치
  $ wget https://pypi.python.org/packages/source/p/pdfminer.six/pdfminer.six-20160202.zip
  $ unzip pdfminer.six-20160202.zip
  $ cd pdfminer.six-20160202.zip
  $ python setup.py install
  
  # 추출
  $ pdf2txt.py woori.pdf
  </code>
  </pre>
  
* PDF를 파싱해서 텍스트 박스의 내용 출력하기
<pre>
<code>
import sys
from pdfminer.converter import PDFPageAggregator
from pdfminer.layout import LAParams, LTContainer, LTTextBox
from pdfminer.pdfinterp import PDFPageInterpreter, PDFResourceManager
from pdfminer.pdfpage import PDFPage

def find_textboxes_recursively(layout_obj):
    """
    재귀적으로 텍스트 박스(LTTextBox)를 찾고
    텍스트 박스들을 리스트로 반환합니다.
    """
    # LTTextBox를 상속받은 객체의 경우 리스트에 곧바로 넣어서 반환합니다.
    if isinstance(layout_obj, LTTextBox):
        return [layout_obj]
    # LTContainer를 상속받은 객체의 경우 자식 요소를 포함하고 있다는 의미이므로
    # 재귀적으로 자식 요소를 계속 찾습니다.
    if isinstance(layout_obj, LTContainer):
        boxes = []
        for child in layout_obj:
            boxes.extend(find_textboxes_recursively(child))
        return boxes
    # 아무것도 없다면 빈 리스트를 반환합니다.
    return []

# 공유 리소스를 관리하는 리소스 매니저를 생성합니다.
laparams = LAParams()
resource_manager = PDFResourceManager()

# 페이지를 모으는 PageAggregator 객체를 생성합니다.
device = PDFPageAggregator(resource_manager, laparams=laparams)

# Interpreter 객체를 생성합니다.
interpreter = PDFPageInterpreter(resource_manager, device)

# 파일을 바이너리 형식으로 읽어 들입니다.
with open(sys.argv[1], 'rb') as f:
    # PDFPage.get_pages()로 파일 객체를 지정합니다.
    # PDFPage 객체를 차례대로 추출합니다.
    # 키워드 매개변수인 pagenos로 처리할 페이지 번호(0-index)를 리스트 형식으로 지정할 수도 있습니다.
    for page in PDFPage.get_pages(f):
        # 페이지를 처리합니다.
        interpreter.process_page(page)
        # LTPage 객체를 추출합니다.
        layout = device.get_result()
        # 페이지 내부의 텍스트 박스를 리스트로 추출합니다.
        boxes = find_textboxes_recursively(layout)
        # 텍스트 박스를 왼쪽 위의 좌표부터 차례대로 정렬합니다.
        # y1(Y 좌표)는 위에 있을수록 크므로 음수로 변환하게 해서 비교했습니다.
        boxes.sort(key=lambda b: (-b.y1, b.x0))
        for box in boxes:
            # 읽기 쉽게 선을 출력합니다.
            print('-' * 10)
            # 텍스트 박스의 내용을 출력합니다.
            print(box.get_text().strip())
</code>
</pre>

* Linked Open Data를 기반으로 데이터 수집하기
  * SPARQL을 사용해 한국의 박물관 추출하기
  <pre>
  <code>
  # pip install SPARQLWrapper
  from SPARQLWrapper import SPARQLWrapper  

  # SPARQL 엔드 포인트를 지정해서 인스턴스를 생성합니다.
  sparql = SPARQLWrapper('http://ko.dbpedia.org/sparql')

  # 한국의 박물관을 추출하는 쿼리입니다.
  sparql.setQuery('''
  SELECT * WHERE {
      ?s rdf:type dbpedia-owl:Museum .
      ?s prop-ko:소재지 ?address .
  } ORDER BY ?s
  ''')

  # 반환 형식을 JSON으로 지정합니다.
  sparql.setReturnFormat('json')

  # query()로 쿼리를 실행한 뒤 convert()로 파싱합니다.
  response = sparql.query().convert()
  for result in response['results']['bindings']:
      # 출력합니다.
      print(result['s']['value'], result['address']['value'])
  </code>
  </pre>
  
* 웹 페이지 자동 조작
  * RoboBrowser 사용
  <pre>
  <code>
  $ pip install robobrowser chardet
  </code>
  </pre>
  
  * RoboBrowser로 구글 검색하기
  <pre>
  <code>
  from robobrowser import RoboBrowser

  # RoboBrowser 객체를 생성합니다.
  # 키워드 매개변수 parser는 BeautifulSoup()의 두 번째 매개변수와 같습니다.
  browser = RoboBrowser(parser='html.parser')

  # open() 메서드로 구글 메인 페이지를 엽니다.
  browser.open('https://www.google.co.kr/')

  # 키워드를 입력합니다.
  form = browser.get_form(action='/search')
  form['q'] = 'Python'
  browser.submit_form(form, list(form.submit_fields.values())[0])

  # 검색 결과 제목을 추출합니다.
  # select() 메서드는 BeautifulSoup의 select() 메서드와 같습니다.
  for a in browser.select('h3 > a'):
      print(a.text)
      print(a.get('href'))
      print()
  </code>
  </pre>

* 네이버페이 주문 이력 추출하기
<pre>
<code>
import time
import sys
import os
from robobrowser import RoboBrowser

# 인증 정보를 환경변수에서 추출합니다.
NAVER_ID = os.environ['NAVER_ID']
NAVER_PASSWORD = os.environ['NAVER_PASSWORD']

# RoboBrowser 객체를 생성합니다.
browser = RoboBrowser(
    # Beautiful Soup에서 사용할 파서를 지정합니다.
    parser='html.parser',
    # 일반적인 웹 브라우저의 User-Agent(FireFox)를 사용합니다.
    user_agent='Mozilla/5.0 (Macintosh; Intel Mac macOS 10.10; rv:45.0) Gecko/20100101 Firefox/45.0')

def main():
    # 로그인 페이지를 엽니다.
    print('Accessing to sign in page....', file=sys.stderr)
    browser.open('https://nid.naver.com/nidlogin.login')
    
    # 로그인 페이지에 들어가졌는지 확인합니다.
    assert '네이버 : 로그인' in browser.parsed.title.string
    
    # name='frmNIDLogin'이라는 입력 양식을 채웁니다.
    # 입력 양식의 name 속성은 개발자 도구로 확인할 수 있습니다.
    form = browser.get_form(attrs={'name': 'frmNIDLogin'})
    
    # name='id'라는 입력 양식을 채웁니다.
    form['id'] = NAVER_ID
    # name='pw'라는 입력 양식을 채웁니다.
    form['pw'] = NAVER_PASSWORD
    
    # 입력 양식을 전송합니다.
    # 로그인 때 로그인을 막는 것을 회피하고자 몇 가지 추가 정보를 전송합니다.
    print('Signing in...', file=sys.stderr)
    browser.submit_form(form, headers={
        'Referer': browser.url,
        'Accept-Language': 'ko,en-US;q=0.7,en;q=0.3',
    })
    
    # 주문 이력 페이지를 엽니다.
    browser.open('https://order.pay.naver.com/home?tabMenu=SHOPPING&frm=s_order')
    
    # 문제가 있을 경우 HTML 소스코드를 확인할 수 있게 출력합니다.
    # print(browser.parsed.prettify())
    # 주문 이력 페이지가 맞는지 확인합니다.
    assert '네이버페이' in browser.parsed.title.string
    # 주문 이력을 출력합니다.
    print_order_history()

def print_order_history():
    """
    주문 이력을 출력합니다.
    """
    # 주문 이력을 순회합니다: 클래스 이름은 개발자 도구로 확인합니다.
    for item in browser.select('.p_info'):
        # 주문 이력 저장 전용 dict입니다.
        order = {} 
        # 주문 이력의 내용을 추출합니다.
        name_element = item.select_one('span')
        date_element = item.select_one('.date')
        price_element = item.select_one('em')
        # 내용이 있을 때만 저장합니다.
        if name_element and date_element and price_element:
            name = name_element.get_text().strip()
            date = date_element.get_text().strip()
            price = price_element.get_text().strip()
            order[name] = {
                'date': date,
                'price': price
            }
            print(order[name]['date'], '-', order[name]['price'] + '원')

if __name__ == '__main__':
    main()
</code>
</pre>

### 5-6. 자바스크립트를 이용한 페이지 스크레이핑
* 자바스크립트를 사용한 페이지에 대한 대응 방법  
  * Selenium과 PhangomJS 설치
  <pre>
  <code>
  $ pip install selenium==3.0
  $ brew install phantomjs

  $ wget https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-2.1.1-linux-x86_64.tar.bz2
  $ tar xvf phantomjs-2.1.1-linux-x86_64.tar.bz2
  $ sudo cp phantomjs-2.1.1-linux-x86_64/bin/phantomjs /usr/local/bin/
  $ sudo apt-get install -y fonts-nanum*
  </code>
  </pre>

  * Selenium으로 구글 검색하기
  <pre>
  <code>
  from selenium import webdriver
  from selenium.webdriver.common.keys import Keys

  # PhantomJS 모듈의 WebDriver 객체를 생성합니다.
  driver = webdriver.PhantomJS()

  # Google 메인 페이지를 엽니다.
  driver.get('https://www.google.co.kr/')

  # 타이틀에 'Google'이 포함돼 있는지 확인합니다.
  assert 'Google' in driver.title

  # 검색어를 입력하고 검색합니다.
  input_element = driver.find_element_by_name('q')
  input_element.send_keys('Python')
  input_element.send_keys(Keys.RETURN)

  # 타이틀에 'Python'이 포함돼 있는지 확인합니다.
  assert 'Python' in driver.title

  # 스크린샷을 찍습니다.
  driver.save_screenshot('search_results.png')

  # 검색 결과를 출력합니다.
  for a in driver.find_elements_by_css_selector('h3 > a'):
      print(a.text)
      print(a.get_attribute('href'))
      print()
  </code>
  </pre>
  
* PhantomJS 활용하기
  * 네이버 페이 주문 이력 추출하기
  <pre>
  <code>
  import sys
  import time

  from selenium import webdriver
  from selenium.webdriver.common.by import By
  from selenium.webdriver.support import expected_conditions as EC
  from selenium.webdriver.support.ui import WebDriverWait

  # 인증 정보를 환경변수에서 추출합니다.
  NAVER_ID = os.environ['NAVER_ID']
  NAVER_PASSWORD = os.environ['NAVER_PASSWORD']

  def main():
      """
      메인 처리
      """
      # PhantomJS의 WebDriver 객체를 생성합니다.
      driver = webdriver.PhantomJS()

      # 화면 크기를 설정합니다.
      driver.set_window_size(800, 600)

      # 로그인하고 이동한 뒤 주문 이력을 가져옵니다.
      sign_in(driver)
      navigate(driver)
      goods = scrape_history(driver)
      # 출력합니다.
      print(goods)

  def sign_in(driver):
      """
      로그인합니다
      """
      print('Navigating...', file=sys.stderr)
      print('Waiting for sign in page loaded...', file=sys.stderr)
      time.sleep(2)

      # 입력 양식을 입력하고 전송합니다.
      driver.get('https://nid.naver.com/nidlogin.login')
      e = driver.find_element_by_id('id')
      e.clear()
      e.send_keys(NAVER_ID)
      e = driver.find_element_by_id('pw')
      e.clear()
      e.send_keys(NAVER_PASSWORD)
      form = driver.find_element_by_css_selector("input.btn_global[type=submit]")
      form.submit()

  def navigate(driver):
      """
      적절한 페이지로 이동한 뒤 
      """
      print('Navigating...', file=sys.stderr)
      driver.get("https://order.pay.naver.com/home?tabMenu=SHOPPING")
      print('Waiting for contents to be loaded...', file=sys.stderr)
      time.sleep(2)

      # 페이지를 아래로 스크롤합니다.
      # 사실 현재 예제에서는 필요 없지만 활용 예를 위해 넣어봤습니다.
      # 스크롤을 해서 데이터를 가져오는 페이지의 경우 활용할 수 있습니다.
      driver.execute_script('scroll(0, document.body.scrollHeight)')
      wait = WebDriverWait(driver, 10)

      # [더보기] 버튼을 클릭할 수 있는 상태가 될 때까지 대기하고 클릭합니다.
      # 두 번 클릭해서 과거의 정보까지 들고옵니다.
      driver.save_screenshot('note-1.png')
      button = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '#_moreButton a')))
      button.click()
      button = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '#_moreButton a')))
      button.click()
      # 2초 대기합니다.
      print('Waiting for contents to be loaded...', file=sys.stderr)
      time.sleep(2)

  def scrape_history(driver):
      """
      페이지에서 주문 이력을 추출합니다.
      """
      goods = []
      for info in driver.find_elements_by_css_selector('.p_info'):
          # 요소를 추출합니다.
          link_element = info.find_element_by_css_selector('a')
          title_element = info.find_element_by_css_selector('span')
          date_element = info.find_element_by_css_selector('.date')
          price_element = info.find_element_by_css_selector('em')
          # 텍스트를 추출합니다.
          goods.append({
              'url': link_element.get_attribute('.a'),
              'title': title_element.text,
              'description': date_element.text + " - " + price_element.text + "원"
          })
      return goods

  if __name__ == '__main__':
      main()
  </code>
  </pre>
  
* RSS 피드 생성하기
  * 설치
  <pre>
  <code>
  $ pip install feedgenerator
  </code>
  </pre>
  
  * 네이버페이 주문 이력으로 피드 생성하기
  <pre>
  import sys
  import time

  from selenium import webdriver
  from selenium.webdriver.common.by import By
  from selenium.webdriver.support import expected_conditions as EC
  from selenium.webdriver.support.ui import WebDriverWait
  import feedgenerator

  # 인증 정보를 환경변수에서 추출합니다.
  NAVER_ID = os.environ['NAVER_ID']
  NAVER_PASSWORD = os.environ['NAVER_PASSWORD']

  def main():
      """
      메인 처리
      """
      # PhantomJS의 WebDriver 객체를 생성합니다.
      driver = webdriver.PhantomJS()

      # 화면 크기를 설정합니다.
      driver.set_window_size(800, 600)

      # 로그인하고 이동한 뒤 주문 이력을 가져옵니다.
      sign_in(driver)
      navigate(driver)
      goods = scrape_history(driver)

      # RSS 피드로 저장합니다.
      with open('shopping_history.rss', 'w') as f:
          save_as_feed(f, goods)

  def sign_in(driver):
      """
      로그인합니다
      """
      print('Navigating...', file=sys.stderr)
      print('Waiting for sign in page loaded...', file=sys.stderr)
      time.sleep(2)

      # 입력 양식을 입력하고 전송합니다.
      driver.get('https://nid.naver.com/nidlogin.login')
      e = driver.find_element_by_id('id')
      e.clear()
      e.send_keys(NAVER_ID)
      e = driver.find_element_by_id('pw')
      e.clear()
      e.send_keys(NAVER_PASSWORD)
      form = driver.find_element_by_css_selector("input.btn_global[type=submit]")
      form.submit()

  def navigate(driver):
      """
      적절한 페이지로 이동한 뒤 
      """
      print('Navigating...', file=sys.stderr)
      driver.get("https://order.pay.naver.com/home?tabMenu=SHOPPING")
      print('Waiting for contents to be loaded...', file=sys.stderr)
      time.sleep(2)
      # 페이지를 아래로 스크롤합니다.
      # 사실 현재 예제에서는 필요 없지만 활용 예를 위해 넣어봤습니다.
      # 스크롤을 해서 데이터를 가져오는 페이지의 경우 활용할 수 있습니다.
      driver.execute_script('scroll(0, document.body.scrollHeight)')
      wait = WebDriverWait(driver, 10)

      # [더보기] 버튼을 클릭할 수 있는 상태가 될 때까지 대기하고 클릭합니다.
      # 두 번 클릭해서 과거의 정보까지 들고옵니다.
      driver.save_screenshot('note-1.png')
      button = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '#_moreButton a')))
      button.click()
      button = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '#_moreButton a')))
      button.click()
      # 2초 대기합니다.
      print('Waiting for contents to be loaded...', file=sys.stderr)
      time.sleep(2)

  def scrape_history(driver):
      """
      페이지에서 주문 이력을 추출합니다.
      """
      goods = []
      for info in driver.find_elements_by_css_selector('.p_info'):
          # 요소를 추출합니다.
          link_element = info.find_element_by_css_selector('a')
          title_element = info.find_element_by_css_selector('span')
          date_element = info.find_element_by_css_selector('.date')
          price_element = info.find_element_by_css_selector('em')
          # 텍스트를 추출합니다.
          goods.append({
              'url': link_element.get_attribute('.a'),
              'title': title_element.text,
              'description': date_element.text + " - " + price_element.text + "원"
          })
      return goods

  def save_as_feed(f, posts):
      """
      주문 내역을 피드로 저장합니다.
      """
      # Rss201rev2Feed 객체를 생성합니다.
      feed = feedgenerator.Rss201rev2Feed(
          title='네이버페이 주문 이력',
          link='https://order.pay.naver.com/',
          description='주문 이력')

      # 피드를 추가합니다.
      for post in posts:
          feed.add_item(title=post['title'],
                        link=post['url'],
                        description=post['description'],
                        unique_id=post['url'])

      # 피드를 저장합니다.
      feed.write(f, 'utf-8')

  if __name__ == '__main__':
      main()
  </code>
  </pre>
  
### 5-7. 추출한 데이터 활용하기
* 지도로 시각화 하기
  * 지오코딩으로 위치 정보 추출하기 : API 키는 구글에 로그인한 상태로 사용 신청
  * 한국의 박물관 위치 정보 추출하기
  <pre>
  <code>
  import time
  import sys
  import os
  import json
  import dbm
  from urllib.request import urlopen
  from urllib.parse import urlencode
  from SPARQLWrapper import SPARQLWrapper

  def main():
      features = []  # 박물관 정보 저장을 위한 리스트
      for museum in get_museums():
          # 레이블이 있는 경우에는 레이블, 없는 경우에는 s를 추출합니다.
          label = museum.get('label', museum['s'])
          address = museum['address']
          lng, lat = geocode(address)

          # 값을 출력해 봅니다.
          print(label, address, lng, lat)
          # 위치 정보를 추출하지 못 했을 경우 리스트에 추가하지 않습니다.
          if lng is None:
              continue

          # features에 박물관 정보를 GeoJSON Feature 형식으로 추가합니다.
          features.append({
              'type': 'Feature',
              'geometry': {'type': 'Point', 'coordinates': [lng, lat]},
              'properties': {'label': label, 'address': address},
          })

      # GeoJSON FeatureCollection 형식으로 dict를 생성합니다.
      feature_collection = {
          'type': 'FeatureCollection',
          'features': features,
      }
      # FeatureCollection을 .geojson이라는 확장자의 파일로 저장합니다.
      with open('museums.geojson', 'w') as f:
          json.dump(feature_collection, f)

  def get_museums():
      """
      SPARQL을 사용해 DBpedia에서 박물관 정보 추출하기
      """
      print('Executing SPARQL query...', file=sys.stderr)

      # SPARQL 엔드 포인트를 지정해서 인스턴스를 생성합니다.
      sparql = SPARQLWrapper('http://ko.dbpedia.org/sparql')

      # 한국의 박물관을 추출하는 쿼리입니다.
      sparql.setQuery('''
      SELECT * WHERE {
          ?s rdf:type dbpedia-owl:Museum .
          ?s prop-ko:소재지 ?address .
          OPTIONAL { ?s rdfs:label ?label . }
      } ORDER BY ?s
      ''')

      # 반환 형식을 JSON으로 지정합니다.
      sparql.setReturnFormat('json')

      # query()로 쿼리를 실행한 뒤 convert()로 파싱합니다.
      response = sparql.query().convert()
      print('Got {0} results'.format(len(response['results']['bindings']), file=sys.stderr))
      # 쿼리 결과를 반복 처리합니다.
      for result in response['results']['bindings']:
          # 다루기 쉽게 dict 형태로 변환해서 yield합니다.
          yield {name: binding['value'] for name, binding in result.items()}

  # Google Geolocation API
  GOOGLE_GEOCODER_API_URL = 'https://maps.googleapis.com/maps/api/geocode/json'
  # DBM(파일을 사용한 Key-Value 데이터베이스)로 지오코딩 결과를 캐시합니다.
  # 이 변수는 dict처럼 다룰 수 있습니다.
  geocoding_cache = dbm.open('geocoding.db', 'c')

  def geocode(address):
      """
      매개변수로 지정한 주소를 지오코딩해서 위도와 경도를 반환합니다.
      """
      if address not in geocoding_cache:
          # 주소가 캐시에 존재하지 않는 경우 지오코딩합니다.
          print('Geocoding {0}...'.format(address), file=sys.stderr)
          time.sleep(1)
          url = GOOGLE_GEOCODER_API_URL + '?' + urlencode({
              'key': os.environ['GOOGLE_API_ID'],
              'language': 'ko',
              'address': address,
          })
          response_text = urlopen(url).read()
          # API 응답을 캐시에 저장합니다.
          # 문자열을 키와 값에 넣으면 자동으로 bytes로 변환합니다.
          geocoding_cache[address] = response_text

      # 캐시 내의 API 응답을 dict로 변환합니다.
      # 값은 bytes 자료형이므로 문자열로 변환합니다.
      response = json.loads(geocoding_cache[address].decode('utf-8'))
      try:
          # JSON 형식에서 값을 추출합니다.
          lng = response['results'][0]['geometry']['location']['lng']
          lat = response['results'][0]['geometry']['location']['lat']
          # float 형태로 변환한 뒤 튜플을 반환합니다.
          return (float(lng), float(lat))
      except:
          return (None, None)

  if __name__ == '__main__':
      main()
  </code>
  </pre>
  
  * Google Maps JavaScript API로 지도에 시각화하기 : 지도에 GeoJSON의 내용을 출력하는 HTML
  <pre>
  <code>
  <!DOCTYPE HTML>
  <meta charset="utf-8">
  <title>한국의 박물관</title>
  <style>
  html, body, #map { height: 100%; margin: 0; padding: 0; }
  </style>
  <div id="map"></div>
  <script>
  function initMap() {
      // 지도를 초기화합니다.
      var map = new google.maps.Map(document.getElementById('map'), {
          center: { lat: 35.7, lng: 137.7 },
          zoom: 7
      });
      // InfoWindow 객체를 생성합니다.
      var infowindow = new google.maps.InfoWindow();
      // geojson 파일의 상대 URL을 지정합니다.
      var geojsonUrl = './museums.geojson';
      // geojson 파일을 읽어 들이고 출력합니다.
      map.data.loadGeoJson(geojsonUrl);
      // 마커를 클릭했을 때 실행할 이벤트를 등록합니다.
      map.data.addListener('click', function(e) {
          // 생성하고 박물관 이름(labe)을 추가합니다.
          var h2 = document.createElement('h2');
          h2.textContent = e.feature.getProperty('label');
          // div 요소를 생성하고, h2 요소와 박물관 주소(address)를 추가합니다.
          var div = document.createElement('div');
          div.appendChild(h2);
          div.appendChild(document.createTextNode('주소: ' + e.feature.getProperty('address')));
          // InfoWindow에 출력할 내용으로 div 요소를 지정합니다.
          infowindow.setContent(div);
          // 출력 위치로 마커의 위치를 지정합니다.
          infowindow.setPosition(e.feature.getGeometry().get());
          // 지정한 지점에서 38픽셀 위에 출력하게 합니다.
          infowindow.setOptions({pixelOffset: new google.maps.Size(0, -38)});
          // InfoWindow를 출력합니다.
          infowindow.open(map);
      });
  }
  </script>
  <!-- Google Maps JavaScript API 스크립트를 읽어 들입니다. 완료했을 때 initMap() 함수를 호출합니다. -->
  <script async defer src="https://maps.googleapis.com/maps/api/js?callback=initMap"></script>
  </code>
  </pre>
  
* BigQuery로 해석하기
  * 트위터 데이터를 BigQuery로 임포트하기
  <pre>
  <code>
  # 설치
  $ pip install bigquery-python
  
  # import_from_stream_api_to_bigquery.py
  import os
  import sys
  from datetime import timezone
  import tweepy
  import bigquery

  # 트위터 인증 정보를 읽어 들입니다.
  CONSUMER_KEY = os.environ['CONSUMER_KEY']
  CONSUMER_SECRET = os.environ['CONSUMER_SECRET']
  ACCESS_TOKEN = os.environ['ACCESS_TOKEN']
  ACCESS_TOKEN_SECRET = os.environ['ACCESS_TOKEN_SECRET']
  auth = tweepy.OAuthHandler(CONSUMER_KEY, CONSUMER_SECRET)
  auth.set_access_token(ACCESS_TOKEN, ACCESS_TOKEN_SECRET)

  # BigQuery 인증 정보(credentials.json)을 지정해 BigQuery 클라이언트를 생성합니다.
  # 명시적으로 readonly=False를 지정하지 않으면 쓰기 작업을 할 수 없습니다.
  client = bigquery.get_client(json_key_file='credentials.json', readonly=False)

  # BigQuery 데이터 세트 이름
  DATASET_NAME = 'twitter'

  # BigQuery 테이블 이름
  TABLE_NAME = 'tweets'

  # 테이블이 존재하지 않으면 생성합니다.
  if not client.check_table(DATASET_NAME, TABLE_NAME):
      print('Creating table {0}.{1}'.format(DATASET_NAME, TABLE_NAME), file=sys.stderr)
      # create_table()의 3번째 매개변수로 스키마를 지정합니다.
      client.create_table(DATASET_NAME, TABLE_NAME, [
          {'name': 'id',          'type': 'string',    'description': '트윗 ID'},
          {'name': 'lang',        'type': 'string',    'description': '트윗 언어'},
          {'name': 'screen_name', 'type': 'string',    'description': '사용자 이름'},
          {'name': 'text',        'type': 'string',    'description': '트윗 문장'},
          {'name': 'created_at',  'type': 'timestamp', 'description': '트윗 날짜'},
      ])

  class MyStreamListener(tweepy.streaming.StreamListener):
      """
      Streaming API로 추출한 트윗을 처리하기 위한 클래스
      """
      status_list = []
      num_imported = 0
      def on_status(self, status):
          """
          트윗을 추출할 때 호출되는 메서드입니다.
          매개변수: 트윗을 나타내는 Status 객체
          """
          # Status 객체를 status_list에 추가합니다.
          self.status_list.append(status)
          if len(self.status_list) >= 500:
              # status_list에 500개의 데이터가 모이면 BigQuery에 임포트합니다.
              if not push_to_bigquery(self.status_list):
                  # 임포트에 실패하면 False가 반환되므로 오류를 출력하고 종료합니다.
                  print('Failed to send to bigquery', file=sys.stderr)
                  return False
              # num_imported를 추가한 뒤 status_list를 비웁니다.
              self.num_imported += len(self.status_list)
              self.status_list = []
              print('Imported {0} rows'.format(self.num_imported), file=sys.stderr)
              # 요금이 많이 나오지 않게 5000개를 임포트했으면 종료합니다.
              # 계속 임포트하고 싶다면 다음 두 줄을 주석 처리해 주세요.
              if self.num_imported >= 5000:
                  return False

      def push_to_bigquery(status_list):
          """
          트윗 리스트를 BigQuery에 임포트하는 메서드입니다.
          """
          # Tweepy의 Status 객체 리스트를 dict 리스트로 변환합니다.
          rows = []
          for status in status_list:
              rows.append({
                  'id': status.id_str,
                  'lang': status.lang,
                  'screen_name': status.author.screen_name,
                  'text': status.text,
                  # datetime 객체를 UTC POSIX 타임스탬프로 변환합니다.
                  'created_at': status.created_at.replace(tzinfo=timezone.utc).timestamp(),
              })
          # dict 리스트를 BigQuery에 임포트합니다.
          # 매개변수는 순서대로
          # <데이터 세트 이름>, <테이블 이름>, <데이터 리스트>, <데이터를 식별할 필드 이름>입니다.
          # insert_id_key는 데이터가 중복되지 않게 만들려고 사용했습니다.
          return client.push_rows(DATASET_NAME, TABLE_NAME, rows, insert_id_key='id')

  # Stream API로 읽어 들이기 시작합니다.
  print('Collecting tweets...', file=sys.stderr)
  stream = tweepy.Stream(auth, MyStreamListener())

  # 공개된 트윗을 샘플링한 스트림을 받습니다.
  # 언어를 지정하지 않았으므로 모든 언어의 트윗을 추출할 수 있습니다.
  stream.sample()
  </code>
  </pre>
