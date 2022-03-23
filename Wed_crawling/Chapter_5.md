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
  
  
  
