---
layout: post
title: "알아두면 좋은 리눅스 명령어"
# author: "DONGWON KIM"
# meta: "Springfield"
categories: "Infra"
comments: true
---

<h2>1. 현재 디렉토리에서 특정 확장자와 문자열을 포함한 파일검색 (find, grep, xargs, sed, sort)</h2><p>특정 디렉토리의 파일들을 검색하기 위해서 find 명령어를 사용합니다.</p><pre class="EnlighterJSRAW" data-enlighter-language="shell"># 현재 디렉토리 하위에 있는 모든 파일들을 출력
find .

# 현재 디렉토리에서 확장자가 log 인 파일들을 출력
find ./*.log</pre><p>grep 명령어를 활용하면 문자열 기반의 강력한 탐색을 할수 있습니다. 다른 명령어와 파이프로 연계하여 많이 활용됩니다. fgrep 명령어를 사용하면 정규표현식을 사용할수 없지만 빠른 탐색 속도를 기대할수 있습니다. 다음과 같이 find와 grep을 연계하여 확장자가 .log 인 파일들을 뽑아 낼수 있습니다.</p><pre class="EnlighterJSRAW" data-enlighter-language="shell"># 현재 디렉토리의 파일들중 확장자가 log 인 파일들을 출력
ls -al | grep .log

# 특정 파일들에 존재하는 임의의 문자열을 탐색해 출력
grep hello hoho.log hehe.log

# 다음과 같이 정규식을 활용하는것도 가능하다.
find . | grep "\.log$"</pre><p>이제 다음으로 할일은 뽑아낸 파일들중에 특정 문자열을 포함하는 파일만 뽑아 내는것이다. grep 명령어의 arguments에 파일들을 마지막 인자로 전달하면 되지만 이를 어떻게 구현할것인가?  xargs 명령어를 입력하면 표준입력으로 들어온 인자들을 command line argument로 주는것이 가능하다. 다음 명령어를 통해 특정 문자열이 들어있는 녀석들을 고를수 있다.</p><pre class="EnlighterJSRAW" data-enlighter-language="shell">find . | grep "\.log$" | xargs grep hello

# 다음 명령어과 같은 출력을 발생시킨다.
# grep hello 1.log 2.log 3.log 4.log 5.log</pre><pre class="EnlighterJSRAW" data-enlighter-language="shell"># 실행 결과 예시
# 각 파일마다 매칭된 문자열을 붙여서 표현해준다.

./1.log:hello
./2.log:hello
./3.log:hello
./4.log:hello
./5.log:hello
./5.log:hello
./5.log:hello hoho</pre><p>grep 명령어를 사용하면 특정 파일들에 임의 문자열이 있는지 체크할수 있지만 모든 검색결과를 반환하므로 우리가 원하는 출력과 맡게 다듬을 필요가 있습니다. sed 명령어를 사용하면 표준입력을 포맷팅(특정 문자 삭제, 대체 등)하여 출력할수 있습니다. 위의 출력에서 파일명만 sed 를 활용하여 뽑아낸 명령어입니다.</p><pre class="EnlighterJSRAW" data-enlighter-language="shell">find . | grep "\.log$" | xargs grep hello | sed 's/:.*//'</pre><pre class="EnlighterJSRAW" data-enlighter-language="shell"># 출력결과
./1.log
./2.log
./3.log
./4.log
./5.log
./5.log
./5.log</pre><p>파일이름만 추출하는데 성공했지만 여전히 중복된 파일들이 보인다. sort 명령어를 활용하면 표준입력으로 들어온 문자열들을 정렬할수 있고 -u 옵션을 사용하면 중복되는 문자열을 제거할수 있다. -u옵션을 활용하여 중복되는 파일을 제거하면 우리가 원하는 목적을 드디어 달성할수 있다.</p><pre class="EnlighterJSRAW" data-enlighter-language="shell">find . | grep "\.log$" | xargs grep hello | sed 's/:.*//' | sort -u</pre><pre class="EnlighterJSRAW" data-enlighter-language="shell"># 출력결과
./1.log
./2.log
./3.log
./4.log
./5.log</pre><h2>2. 최근에 수정된 파일 찾기 (find, xargs, sort)</h2><p>find 명령어의 -mmin 옵션을 활용하면 최근에 어떤 파일들이 수정되었는지 확인할수 있습니다.</p><pre class="EnlighterJSRAW" data-enlighter-language="shell"># 루트 하위 기준으로 60분 이내에 수정된 파일들을 찾는다.
find / -mmin -60

# 다음 명령어로 파일로도 저장할수 있다.
find / -mmin -60 -type -f &gt; resent_files.txt</pre><p>최근 수정된 파일들의 크기를 알고싶을때 ls 명령어와 파이프로 연계하면 됩니다. sort 명령어와 함께 파이프로 연계하면 파일들을 크기순으로 정렬할수 있습니다.</p><pre class="EnlighterJSRAW" data-enlighter-language="shell"># -n 옵션으로 정수기반의 정렬
# -r 옵션으로 DESC로 정렬
# -k 5 옵션으로 다섯번째 키인 파일의 용량으로 정렬
find / -mmin -10 | xargs ls -al | sort -nrk 5</pre><p>단순하게 수정된 파일을 찾는것이 아니라, 디스크 용량 부족 문제가 생겼을때 어떤 파일이 최근에 얼마나 커졌는지를 감시할때 유용하게 사용될수 있습니다.</p><h2>3. 로그 파일 내용 확인하기 (cat, sed, tail, head)</h2><p>cat 명령어를 사용하면 파일의 내용을 쉘에 출력할수 있습니다.</p><pre class="EnlighterJSRAW" data-enlighter-language="generic">cat access.log</pre><p>하지만 보통 로그 파일의 내용은 n GB를 넘어가기 때문에 무작정 cat을 사용했다가 후회할수도 있습니다. sed 명령어를 활용하면 특정 range를 지정하여 로그를 출력할수 있습니다.</p><pre class="EnlighterJSRAW" data-enlighter-language="generic"># 1번째줄부터 10번째 줄까지 출력
sed -n 1,10p access.log</pre><p>만약 첫번째부터 혹은 마지막부터 특정 수만큼의 줄이 필요하다면 head와 tail을 고려할수 있습니다. 특히 tail 명령어의 경우 서버 정상 동작 확인 및 로그 확인용으로 많이 사용합니다.</p><pre class="EnlighterJSRAW" data-enlighter-language="generic"># 처음부터 10번째 줄까지 출력
head access.log

# 처음부터 3번째 줄까지 출력
head access.log -n 3

# 마지막부터 10번째 줄까지 출력
tail access.log

# 마지막부터 3번째 줄까지 출력
tail access.log -n 3

# 마지막부터 3번째 줄까지 출력후, 파일이 갱신될때마다 계속 출력
tail access.log -fn 3</pre><h2>4. interactive한 명령어를 non-interactive 하도록 만들기 (echo)</h2><p>echo 명령어는 argument로 전달된 문자열을 표준출력으로 내보내는 기능을 합니다. 특정 쉘스크립트나 바이너리를 실행시킨후 키보드를 이용하여 인자를 입력하는 대신에 echo를 사용하여 인자를 전달할수 있습니다.</p><pre class="EnlighterJSRAW" data-enlighter-language="generic"># 바이너리 실행후 표준입력으로 '1 2 3 4 5' 를 전달한것과 같은 효과이다.
echo '1 2 3 4 5' | java helloworld</pre><p>키보드를 통해 전달해야하는 인자를 echo나 파이프, 리다이렉션으로 대체하면 어떤 장점이 있을까요? echo나 파이프, 리다이렉션를 활용하면 제품을 빌드하는 과정에서 키보드를 통해 수동으로 인자를 전달해야하는 번거로움이 필요없기 때문에 인프라의 자동화에 큰 도움이 됩니다.</p><h2>5. su와 sudo</h2><p>su 명령어는 로그아웃 없이 다른 사용자의 로그인 기능을 제공합니다. 파라미터가 없으면 root 계정으로 로그인합니다. 로그인 하려는 계정의 패스워드가 필요하며 전환 하려는 계정의 환경변수를 사용할수 있습니다.</p><p>sudo(substitute user do) 로그아웃 없이, 다른 사용자의 권한으로 명령을 실행 가능하게하는 리눅스 명령어입니다. 파라미터가 없는 sudo는 sudo -u root와 동일합니다. (root 권한으로 실행한다) 현재 로그인된 계정의 패스워드가 필요하며 /etc/sudoers 파일에 지정된 사용자만 sudo 명령을 사용할 수 있습니다.</p><h2>6. 파일의 무결성 체크하기 (diff, awk)</h2><p>우리는 서버를 구축하면서 다양한 유틸리티를 설치하게 됩니다. 하지만 유틸리티를 배포하는 사이트가 해커들에게 공격당하여 설치파일이 변조되고 이를 모르고 다운로드하여 설치하게 되면 치명적인 보안사고를 불러일으킵니다. 그래서 몇몇 유틸리티 배포 사이트는 파일에 대한 checksum 을제공하여 사용자가 해당 파일의 변조여부를 확인할수 있도록 하고 있습니다. diff 명령어는 파일들의 내용을 비교하는 기능을 제공하는데, checksum과 함께 해당파일이 변조되지 않았는지 체크할수 있습니다.</p><p>awk 명령어는 파일이나 표준입력으로부터 레코드를 선택하고 그대로 반환하거나 조작 데이터화 할수 있는 유틸리티입니다. 아래의 명령어에서 md5 명령어로 부터 얻은 출력에서 checksum인 4번째 단어를 뽑아 내기 위해서 사용했습니다.</p><pre class="EnlighterJSRAW" data-enlighter-language="shell"># macos 에서 작동
# &lt;(COMMAND) 구문으로 파일대신에 표준출력을 바로 전달 가능

diff &lt;(md5 Downloads/jdk-8u181-linux-x64.rpm | awk '{print $4}') \
  &lt;(echo 'e7c0593a310b83b4ca69ea22f850c71f') \
  &amp;&amp; echo 'correct file'</pre><h2>7. 참고자료</h2><p>https://linux.die.net/man/</p><p>https://askubuntu.com/questions/620936/difference-between-su-and-sudo-su</p><p>https://superuser.com/questions/1059781/what-exactly-is-in-bash-and-in-zsh</p><p>기술평론사 편집부 저 김성재 역, <b>『</b>인프라 엔지니어의 교과서-네트워크 관리편<b>』</b>, 길벗(2016.11.30)</p>
