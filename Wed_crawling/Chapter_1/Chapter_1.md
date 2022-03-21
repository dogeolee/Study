# Chaper 1. 크롤링과 스크레이핑이란?

### 1-1 이 책에서 다루는 영역
* 크롤링 : 웹 페이지의 하이퍼링크를 순회하면서 웹 페이지를 다운로드하는 작업
* 스크레이핑 : 다운로드한 웹 페이지에서 필요한 정보를 추출하는 작업
* 개발 환경 : Ubuntu18.04 LTS

### 1-2 wget으로 크롤링하기
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
