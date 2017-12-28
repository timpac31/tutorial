# openssl
> openssl update
1. 버전 확인 
  [root@ ] openssl version

2. 문제가 되는 버전일 경우 업데이트 진행<br/>
  http://www.openssl.org/source/ 에 접속하여 [LATEST]라고 적힌 최신 버전의 소스를 
  다운받아 서버로 복사합니다.<br/>
  [root@ ] wget http://www.openssl.org/source/openssl-1.0.1g.tar.gz 

3. tar 명령어로 압축을 풉니다.<br/>
  [root@ ] tar xfvz openssl-1.0.1g.tar.gz 

4. 기존의 openssldir 경로를 입력하여 컴파일/설치를 합니다.<br/>
  [root@ ] ./config --prefix=/usr/local --openssldir=[기존 openssl경로명]<br/>
  [root@ ] make<br/>
  [root@ ] make install<br/>

5. 설치된 버전 확인 <br/>
  [root@ ] openssl version
