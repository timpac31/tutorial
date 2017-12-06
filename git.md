# 초기화
1. 계정설정
+ $) git config --global user.name "timpac31"
+ $) git config --global user.email "timpac31@gmail.com"

2. 초기화
+ $) git init
+ $) rm -rf .git   //git init 취소

3. 원격저장소 연결
+ $) git remote add [저장소이름] [주소]
+ $) git remote add origin https:github.com/timpac31/MySource.git

# add->commit->push
1. add
+ $) git add [파일 or 디렉토리]
+ ex) git add test.jsp	
+ ex) git add .  			//현재폴더 하위 모든 파일 추가

2. commit
+ $) git commit -m "test commit"

3. push
+ $) git push [저장소명] [브랜치명]
+ ex) git push origin master  
+ $) git push origin +master   //강제푸쉬	      

4. pull
+ ex) git pull origin master

# branch
1. 생성
+ ex) git branch [브랜치명]
2. 변경
+ ex) git checkout [브랜치명]
3. 삭제
+ ex) git branch -m [브랜치명]

# git diff editor 등록(p4merge)
1. p4merge 설치
2. config등록
+ ex) git config --global merge.tool p4merge
+ ex) git config --global mergetoll.p4merge.cmd 'p4merge $BASE $LOCAL $REMOTE $MERGED'
+ ex) git config --global mergetool.p4merge.path 'C:\Program Files\Perforce\p4merge.exe'
3. 실행
+ ex) git difftool
