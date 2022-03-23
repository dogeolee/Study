# Chapter 4. 크롤러를 사용할 때 기억해야 하는 것
### 4-1. 크롤러 분류하기
크롤러는 대상 웹사이트에 따라 다양한 성질을 가질 수 있으며 다음과 같은 세 가지 기준에 따라 분류함
* 상태를 가지고 있는지
    * 상태를 가지는 스테이트풀(Statefull) 크롤러
    * 상태를 가지지 않는 스테이트리스(Stateless) 크롤러
* 자바스크립트를 실행할 수 있는지
    * 자바스크립트를 실행할 수 있는 크롤러
    * 자바스크립트를 실행할 수 없는 크롤러
* 불특정 다수의 사이트를 대상으로 하는지
    * 특정 웹사이트만을 대상으로 하는 크롤러
    * 불특정 다수의 웹사잉트를 대상으로 하는 크롤러

### 4-2. 크롤러를 만들 때 주의해야 하는 것
* robots.txt로 크롤러에게 지시하기
<pre>
<code>
# 모든 페이지의 크롤링을 허가하지 않는 robots.txt
User-agent : *
Disallow : /

# 모든 페이지의 크롤링을 허가하는 robots.txt
User-agent : *
Disallow : 

# 특정한 크롤러를 허가하지 않는 robots.txt
User-agent : *
Disallow : /old/
Disallow : /tmp/

User-agent : annoying-bot
Disallow : /

# /articles/ 아래만 크롤링을 허가하는 robots.txt
User-agent : *
Allow : /articles/
Disallow : /
</code>
</pre>

* robots meta 태그
    * nofollow : 해당 페이지 내부의 링크를 타고 도는 것을 허가하지 않음
    * noarchive : 해당 페이지를 아카이브로 저장하는 것을 허가하지 않음
    * noindex : 해당 페이지를 검색 엔진에 인덱스하는 것을 허가하지 않음

* HTTP 통신 오류 분류
    * 네트워크 레벨의 오류 : DNS 이름 해석 실패 또는 통신 타음아웃처럼 서버와 정상적으로 통신할 수 없는 경우 발생
    * 대표적인 HTTP 상태 코드
    
    |상태코드|설명|
    |---|---|
    |100 Continue|요청이 연결되어 있습니다.|
    |200 OK|요청이 성공했습니다.|
    |301 Moved Permanently|요청한 리소스가 영구적으로 이동했습니다.|
    |302 Found|요청한 리소스가 일시적으로 이동했습니다.|
    |304 Not Modified|요청한 리소스가 변경되지 않았습니다.|
    |400 Bad Request|클라이언트의 요청에 문제가 있으므로 처리할 수 없습니다.|
    |401 Unauthoized|인증되지 않았으므로 처리할 수 없습니다.|
    |403 Forbidden|요청이 허가되지 않습니다.|
    |404 Not Found|요청한 리소스가 존재하지 않습니다.|
    |408 Request Timeout|일정 시간 내에 요청이 완료되지 않았습니다.|
    |500 Internal Server Error|서버 내부에 문제가 발생했습니다.|
    |502 Bad Gateway|게이트웨이 서버가 백엔드 서버로부터 오류를 받았습니다.|
    |503 Service Unavailable|서버가 일시적으로 요청을 처리할 수 없는 상태입니다.|
    |504 Gateway Timeout|게이트웨이 서버에서 백엔드 서버로의 요청이 타임아웃됐습니다.|
    
* 파이썬에서 HTTP 통신 오류 처리하기
    * 상태 코드에 맞는 오류 처리하기
    <pre>
    <code>
    import time

    import requests
    # 일시적인 오류를 나타내는 상태 코드를 지정합니다.
    TEMPORARY_ERROR_CODES = (408, 500, 502, 503, 504)  

    def main():
        """
        메인 처리입니다.
        """
        response = fetch('http://httpbin.org/status/200,404,503')
        if 200 <= response.status_code < 300:
            print('Success!')
        else:
            print('Error!')

    def fetch(url):
        """
        지정한 URL에 요청한 뒤 Response 객체를 반환합니다.
        일시적인 오류가 발생하면 최대 3번 재시도합니다.
        """
        max_retries = 3  # 최대 3번 재시도합니다.
        retries = 0  # 현재 재시도 횟수를 나타내는 변수입니다.
        while True:
            try:
                print('Retrieving {0}...'.format(url))
                response = requests.get(url)
                print('Status: {0}'.format(response.status_code))
                if response.status_code not in TEMPORARY_ERROR_CODES:
                    return response  # 일시적인 오류가 아니라면 response를 반환합니다.
            except requests.exceptions.RequestException as ex:
                # 네트워크 레벨 오류(RequestException)의 경우 재시도합니다.
                print('Exception occured: {0}'.format(ex))
                retries += 1
                if retries >= max_retries:
                    # 재시도 횟수 상한을 넘으면 예외를 발생시켜버립니다.
                    raise Exception('Too many retries.')  
                # 지수 함수적으로 재시도 간격을 증가합니다(**는 제곱 연산자입니다).
                wait = 2**(retries - 1)  
                print('Waiting {0} seconds...'.format(wait))
                time.sleep(wait)  # 대기합니다.

    if __name__ == '__main__':
        main()
    </code>
    </pre>
    
    * retrying을 이용한 재시도 처리
    <pre>
    <code>
    import requests
    from retrying import retry  # pip install retrying
    import time
    # 일시적인 오류를 나타내는 상태 코드를 지정합니다.
    TEMPORARY_ERROR_CODES = (408, 500, 502, 503, 504)  

    def main():
        """
        메인 처리입니다.
        """
        response = fetch('http://httpbin.org/status/200,404,503')
        if 200 <= response.status_code < 300:
            print('Success!')
        else:
            print('Error!')

    # stop_max_attempt_number로 최대 재시도 횟수를 지정합니다.
    # wait_exponential_multiplier로 특정한 시간 만큼 대기하고 재시도하게 합니다. 단위는 밀리초로 입력합니다.
    @retry(stop_max_attempt_number=3, wait_exponential_multiplier=1000)
    def fetch(url):
        """
        지정한 URL에 접근한 뒤 Response 객체를 반환합니다.
        일시적인 오류가 발생할 경우 3번까지 재시도합니다.
        """
        print('Retrieving {0}...'.format(url))
        response = requests.get(url)
        print('Status: {0}'.format(response.status_code))
        if response.status_code not in TEMPORARY_ERROR_CODES:
            # 오류가 없다면 response를 반환합니다.
            return response
        # 오류가 있다면 예외를 발생시킵니다.
        raise Exception('Temporary Error: {0}'.format(response.status_code))

    if __name__ == '__main__':
        main()
    </code>
    </pre>
    
### 4-3. 여러 번 사용을 전제로 설계하기
* 변경된 데이터만 추출하기
   * CacheControl을 사용해 캐시 처리하기
   <pre>
   <code>
   import requests
   # pip install CacheControl
   from cachecontrol import CacheControl  

   session = requests.session()
   # session을 래핑한 cached_session 만들기
   cached_session = CacheControl(session)

   # 첫 번째는 캐시돼 있지 않으므로 서버에서 추출한 이후 캐시합니다.
   response = cached_session.get('https://docs.python.org/3/')
   print(response.from_cache)  # False

   # 두 번째는 ETag와 Last-Modified 값을 사용해 업데이트됐는지 확인합니다.
   # 변경사항이 없는 경우에는 콘텐츠를 캐시에서 추출해서 사용하므로 빠른 처리가 가능합니다.
   response = cached_session.get('https://docs.python.org/3/')
   print(response.from_cache)  # True
   </code>
   </pre>
   
### 4-4. 크롤링 대상의 변화에 대응하기
* 변화 감지하기
   * 정규 표현식으로 가격이 제대로 나오는지 확인하기
   <pre>
   <code>
   import re
   value = '3,000'

   # 숫자와 쉼표만을 포함한 정규 표현식에 매치하는지 확인합니다.
   if not re.search(r'^[0-9,]+$', value):
       # 값이 제대로 돼 있지 않다면 예외를 발생시킵니다.
       raise ValueError('Invalid price')
   </code>
   </pre>
   
   * Voluptuous로 유효성 검사하기
   <pre>
   <code>
   # pip install voluptuous
   from voluptuous import Schema, Match  

   # 다음 4개의 규칙을 가진 스키마를 정의합니다
   schema = Schema({                  # 규칙1: 객체는 dict 자료형
       'name': str,                   # 규칙2：name은 str(문자열) 자료형
       'price': Match(r'^[0-9,]+$'),  # 규칙3：price가 정규 표현식에 맞는지 확인
   }, required=True)                  # 규칙4：dict의 키는 필수

   # Schema 객체는 함수처럼 호출해서 사용합니다.
   # 매개변수에 대상을 넣으면 유효성 검사를 수행합니다.
   schema({
       'name': '포도',
       'price': '3,000',
   })  # 유효성 검사를 통과하므로 아무 문제 없음

   schema({
       'name': None,
       'price': '3,000',
   })  # 유효성 검사를 통과하지 못 하므로, MultipleInvalid 예외가 발생
   </code>
   </pre>
   
* 변화 통지하기
   * 메일 전송하기
   <pre>
   <code>
   import smtplib
   from email.mime.text import MIMEText
   from email.header import Header

   # MIMEText 객체로 메일을 생성합니다.
   msg = MIMEText('메일 본분입니다.')  

   # 제목에 한글이 포함될 경우 Header 객체를 사용합니다.
   msg['Subject'] = Header('메일 제목입니다.', 'utf-8') 
   msg['From'] = 'me@example.com'
   msg['To'] = 'you@example.com'

   # SMTP()의 첫 번째 매개변수에 SMTP 서버의 호스트 이름을 지정합니다.
   with smtplib.SMTP('localhost') as smtp:
       # 메일을 전송합니다.
       smtp.send_message(msg)

   '''
   with smtplib.SMTP_SSL('smtp.gmail.com') as smtp:
       # 구글 계정의 사용자 이름과 비밀번호를 지정해서 로그인합니다.
       # 2단계 인증을 설정한 경우 애플리케이션 비밀번호를 사용해 주세요.
       smtp.login('사용자 이름', '비밀번호')
       # send_message() 메서드로 메일을 전송합니다.
       smtp.send_message(msg)
   '''
   </code>
   </pre>
