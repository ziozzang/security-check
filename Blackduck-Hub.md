
# TL;DR
* 블랙덕 허브를 내부 CI등 시스템에 연동 하기.

# 젠킨스 연동
* 쉘 스크립트가 아닌 경우 플러그인을 통해서 설치가 가능함. "blackduck hub" 를 설치 할 것.
 * 관련 플러그인: https://wiki.jenkins.io/display/JENKINS/BlackDuck+Hub+Plugin
* 글로벌 설정 쪽에 서버 접속 정보를 설정 가능 함. 주소는 https://foo.com 같은 방식으로 설정 할 것.
* 프로젝트를 작성 할때에 post-build action 단계 에서 "blackduck hub integration" 를 추가 해주는 방식으로 작업 추가가 가능함.
  * 설정시 어드밴스 옵션을 클릭후, "Generate Black Duck Risk Report" 를 클릭해 줄 것: 이걸 설정해주면 검사 완료를 기다리고 간략한 리포팅이 젠킨스 결과에 임베딩 되게 됨.
  
## 파이프라인 연동

* https://jenkins.io/doc/pipeline/steps/blackduck-hub/ 참고



# 쉘 스크립트 연동
* 블랙덕 허브에 로그인후에 유저 아이콘쪽 Tool섹션에 Hub Scanner 를 설치 해야 한다. 해당 스캐너는 JRE 환경을 요구 한다. headless로 설치 할 것.
  * ex: default-jre-headless or openjdk-9-jre-headless

```
# 아래 설정에서 블랙덕 허브의 주소는 foo.com 이라고 가정 한다.
# 패스워드 선언.
declare -x BD_HUB_PASSWORD="password_here"
# Docker Scanning
./bin/scan.docker.sh \
  --host "foo.com" --port 443 --scheme HTTPS \
  --username "username_here" \
  --project "docker:PRJ:foo" --release "local_images:latest"  --image "centos:7"

# Directory Scanning
./bin/scan.cli.sh \
  --host "foo.com" --port 443 --scheme HTTPS \
  --username "username_here" \
  --project "dir_scan" --release "directory:asdf" /opt/bd/
```

# 주요 이슈
* 릴리즈 이름에 /  문자는 사용이 불가능 함. 대신 : 같은 문자는 사용이 가능함.
