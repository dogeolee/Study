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
* Cron 설정
* 오류 통지

### 7-3. 크롤링과 스크레이핑 분리하기
* 메시지 큐 RQ 사용 방법
* 메시지 큐로 연동하기
* 메시지 큐 운용하기

### 7-4. 크롤링 성능 향상과 비동기 처리
* 멀티 스레드와 멀티 프로세스
* 비동기 I/O를 사용해 효율적으로 크롤링하기

### 7-5. 클라우드 활용하기
* 클라우드의 장점
* AWS SDK 사용하기
* 클라우드 스토리지 사용하기
