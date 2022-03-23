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
