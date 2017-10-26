
# 기본
* 홈페이지: https://www.owasp.org/index.php/OWASP_Dependency_Check

* 해당 홈페이지에서 커맨드라인을 받아서 설치 함.


# 설치
* 미묘하게 젠킨스에서 동작에 이슈 있음. 특히 슬레이브 노드가 따로 놀경우에는 플러그인으로 동작하지 않을수 있음.


# 구동
## 깔아서 쓰는 경우
```
# 업데이트만 하는 경우
dependency-check.sh --updateonly -d /tmp/data/dc.h2.db

# 업데이트된 데이터베이스를 사용해서 현재위치에서 스캐닝. 프로젝트 이름을 꼭 지정해야 함.
dependency-check.sh -f ALL -s . --project "ASDF" -n -d /tmp/data/dc.h2.db
```

## 도커를 통해서 쓰는 경우

```
DBPATH="`pwd`/data"

mkdir -p ${DBPATH}
# 사실 권한은 ubuntu:ubuntu 임...
chmod 777 ${DBPATH}

#최신 버전을 받기
docker pull owasp/dependency-check
#업데이트 하기
docker run --rm \
    --volume ${DBPATH}:/usr/share/dependency-check/data \
    owasp/dependency-check --updateonly
# 실제 스캐닝
docker run --rm \
    --volume ${DBPATH}:/usr/share/dependency-check/data \
    --volume `pwd`:/src \
    --volume `pwd`:/report \
    owasp/dependency-check \
    -f ALL --scan /src --project "ASDF" -n
```

## 젠킨스 연동
* 플러그인 설치 필요함.
* 빌드 후 처리 부분에
```
# Publish OWASP Dependency-Check results
#  >> Dependency-Check results 에 리포트 파일을 넣어 줌 "**/dependency-check-report.xml"
```
