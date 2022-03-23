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
