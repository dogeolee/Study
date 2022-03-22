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

* wget 옵션

|옵션|설명|옵션|설명|
|---|---|---|---|
|-V, --version|wget 버전 출력|-h, --help|도움말 출력|
|-q, --quiet|진행 싱황을 출력하지 않음|-O <file>, --output-document=<file>|file에 저장|
|-c, --continue|이전 상태에서 이어서 파일 다운|-r, --recursive|링크를 돌며 재귀적으로 다운|
|-l depth, --level=<depth>|재귀적 다운 시 순회 깊이 제한|-w <seconds>, --wait=<seconds>|재귀적 다운 시 다운 간격 지정|
|-np, --no-parent|재귀적 다운 시 부모 디렉토리 크롤링 제외|-I <list>, --include <list>|재귀적 다운로드 시 list에 포함된 디렉터리만 돔|
|-N, --timestamping|파일이 변경되었을 때만 다운|-m, --mirror <list>|미러링 전용 옵션 활성화|

* 실제 사이트 크롤링하기\
-l 옵션으로 링크를 한 번만 더 타고 들어가게 만들고, -w 옵션으로 다운로드 간격을 1초로 지정, -no-parent 옵션으로 부모 디렉터리를 크롤링하지 않으며 --restrict-file-names=nocontrol 로 URL에 한국어 등이 포함돼 있을 경우 한국어 파일명으로 저장하게 한다.
<pre>
<code>
$ wget -r --no-parent -w 1 -l 1 --restrict-file-names=nocontrol https://www.hanbit.co.kr/
$ tree www.hanbit.co.kr/
</code>
</pre>
 
### 1-3. 유닉스 명령어로 스크레이핑하기
* 텍스트 저리와 관련된 유닉스 명령어
  * cat 명령어 : 매개변수로 전달된 파일을 출력
  * gerp 명령어 : 일부 줄만 추출할 때 사용
  * cut 명령어 : 텍스트 일부를 제거할 때 사용
  * sed 명령어 : 특정 조건에 맞는 줄을 치환하거나 제거할 때 사용

* 정규 표현식에서 사용할 수 있는 주요 메타 문자
 
|메타 문자|설명|
|---|---|
|.|임의의 문자 하나를 나타냄|
|[]|[] 내부의 문자 중에 하나와 매치|
|[] 내부의 -|-로 문자의 범위를 나타낼 수 있음|
|[] 내부의 ^|^를 앞에 붙이면 부정을 나타냄|
|^|줄의 시작을 나타냄|
|$|줄의 끝을 나타냄|
|*|직전의 패턴을 0번이상 반복|
|+|직전의 패턴을 1번 이상 반복|
|?|직전의 패턴을 0번 또는 1번 반복|
|{n}|직전의 패턴을 지정한 n만큼 반복|
|()|()로 감싼 패턴을 그룹으로 묶음|
|\||\|로 구분된 패턴을 모두 매치함|
 
### 1-4. 페이지 하나 출력하기
* 줄에서 원하는 문자열 위치 추출하기
  * sed 명령어에 s/.\*(<추출하고 싶은 위치의 정규 표현식>).*/\1/ 을 매개변수로 지정
  * sed 명령어에 s/<제거하고 싶은 정규 표현식>//g 를 지정해 정규표현식에 해당하는 부분을 제거해버린 뒤 남는 부분 사용
  * cut 명령어로 특정 문자를 문자열로 자른 후 n번째 데이터를 추출
  * awk 명령어로 공백을 구분한 뒤 n번째 데이터를 추출
 
 
 
