
# 기본
* 홈페이지: https://www.owasp.org/index.php/OWASP_Dependency_Check

* 해당 홈페이지에서 커맨드라인을 받아서 설치 함.
```
VERSION="3.0.2"
wget http://dl.bintray.com/jeremy-long/owasp/dependency-check-${VERSION}-release.zip
unzip dependency-check-${VERSION}-release.zip
mv dependency-check /usr/local/
ln -s /usr/local/dependency-check/bin/dependency-check.sh /usr/local/bin/dependency-check.sh
```


# 설치
* 미묘하게 젠킨스에서 동작에 이슈 있음. 특히 슬레이브 노드가 따로 놀경우에는 플러그인으로 동작하지 않을수 있음.


# 구동
## 깔아서 쓰는 경우
```
# 업데이트만 하는 경우
dependency-check.sh --updateonly -d /opt/owasp/

# 업데이트된 데이터베이스를 사용해서 현재위치에서 스캐닝. 프로젝트 이름을 꼭 지정해야 함.
# 주의: 파이썬, 루비 등을 스캐닝 할때에는 --enableExperimental 옵션을 꼭 붙여 준다.(노드는 오탐이 많아서 제거 되었다고 함)
dependency-check.sh -f ALL -s /opt/test/ --project "ASDF" -d /opt/owasp/  -n --enableExperimental
```

## 도커를 통해서 쓰는 경우

* 원래 도커 버전은 https://hub.docker.com/r/owasp/dependency-check/ 에 있는데, 현재(2017년 11월 28일 기준) 리포트 생성시에 이슈가 생김. 이에 따라 https://hub.docker.com/r/ziozzang/owasp-dependency-check/ 에 새로 빌드 해서 올림(컨테이너 내부에서 루트 권한으로 동작 하도록 수정)


* Update
```
#!/bin/bash
DBPATH=/opt/owasp
mkdir -p ${DBPATH}
chmod 777 ${DBPATH}
docker pull ziozzang/owasp-dependency-check
docker run --rm \
    --volume ${DBPATH}:/usr/share/dependency-check/data \
    ziozzang/owasp-dependency-check --updateonly
```

* Scanning

```
#!/bin/bash
DBPATH=/opt/owasp
REPORT=/opt/report
rm -rf ${REPORT}
mkdir -p ${REPORT}

CWD=`pwd`

docker pull ziozzang/owasp-dependency-check
docker run --rm   \
 -v ${DBPATH}:/usr/share/dependency-check/data \
 -v ${CWD}:/src \
 -v ${REPORT}:/report/ \
 ziozzang/owasp-dependency-check \
 -n --enableExperimental -o /report/ -f ALL --scan /src --project "ASDF"
```

## 테스트 예제
* tomcat-embed-core-8.0.33.jar 이 경우 CVE 이슈가 꽤 많음.
```
docker run --rm \
  --volume ${DBPATH}:/usr/share/dependency-check/data \
  --volume `pwd`:/src \
  --volume `pwd`:/report \
  owasp/dependency-check \
  -f ALL --scan /src/tomcat-embed-core-8.0.33.jar --project "ASDF" -n
```

## 젠킨스 연동
* 플러그인 설치 필요함.
* 빌드 후 처리 부분에
```
# Publish OWASP Dependency-Check results
#  >> Dependency-Check results 에 리포트 파일을 넣어 줌 "**/dependency-check-report.xml"
```
