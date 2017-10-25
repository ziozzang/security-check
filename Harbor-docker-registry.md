# Harbor설치
* 릴리즈 : https://github.com/vmware/harbor/releases 링크 참고. 오프라인으로 설치 하면 오프라인으로 사용 가능함.

```
# ./install.sh --with-notary --with-clair
```
제공된 패키지로 설치시 루트 권한 필요.

## 설치시 오류 대응

```
hostname = test.jioh.net <- 서빙할 호스트 이름 (특정 이름을 지정 해 줄것)
ui_url_protocol = https <- notary설정시에 https 만 가능.

ssl_cert = /opt/cert/server.crt <- ssl용 키
ssl_cert_key = /opt/cert/server.key

secretkey_path = /data <- 어드민 서버가 안뜬다면 이거 경로 만져 줄 것.
```

## 업그레이드
정규 설치 패키지로 설치 한 경우, 압축을 풀고 install.sh를 실행해주면 별다른 이슈 없이 바로 업그레이드가 됨. (기존 설정 유지 된 채로 업데이트 가능)

만일 설치 환경을 리셋 해야 하는 경우 /data 폴더를 리셋 하면 됨 (완전 초기화) - 백업시 동일.

## 테스트
* 우분투 16.04 기준 테스트 완료.

## 운영 이슈
모든 로그는 /var/log/harbor/ 아래에 저장이 됨. (도커 로그 명령어 사용 불가)


# Docker Security check

vmware harbor는 clair를 통해서 도커 cve체크가 가능함. dmz존에 두고 검사용도로 사용 할수 있음.
 * 관련 문서: https://github.com/vmware/harbor/blob/master/docs/user_guide.md

Clair 데이터베이스는 익스포트 임포트가 가능함
 * 관련 문서: https://github.com/vmware/harbor/blob/master/docs/import_vulnerability_data.md

Security check 는 api가 제공되므로 아래의 api를 사용 하면 됨.(퍼블릭의 경우에는 로그인 파라미터 없이도 요청 가능) severity가 5인경우는 high이므로 주의 요망. 하지만 fixedVersion 이 존재 하지 않는 경우는 업데이트를 할수가 없는 상태임. (이 경우는 정책적으로 무시 하던가 기다리는 방법을 써야 함)

## 요약 시큐리티 정보

아래는 요약 정보를 보여 줌. 이중에 스캔 오버뷰 쪽을 보면 확인이 가능함.

```
# curl https://test.jioh.net/api/repositories/test/gerrit/tags/latest
{
  "digest": "sha256:bf27f30bbb338d1f61d1954812a4fd1bcd2cbd9407c667ab307f52ae56aefa5e",
  "name": "latest",
  "architecture": "amd64",
  "os": "linux",
  "docker_version": "17.06.1-ce",
  "author": "zsx \u003cthinkernel@gmail.com\u003e",
  "created": "2017-10-11T09:59:35.804546459Z",
  "signature": null,
  "scan_overview": {
    "image_digest": "sha256:bf27f30bbb338d1f61d1954812a4fd1bcd2cbd9407c667ab307f52ae56aefa5e",
    "scan_status": "finished",
    "job_id": 10,
    "severity": 5,
    "components": {
      "total": 75,
      "summary": [
        {
          "severity": 1,
          "count": 73
        },
        {
          "severity": 2,
          "count": 1
        },
        {
          "severity": 5,
          "count": 1
        }
      ]
    },
    "details_key": "c44db1a15c6be030901515f5cc3f289c10b864bf0fa7db17931de460e6f71cce",
    "creation_time": "2017-10-25T06:37:30Z",
    "update_time": "2017-10-25T07:00:19Z"
  }
}
```

## 상세 시큐리티 정보
```
# curl https://test.jioh.net/api/repositories/test/gogs/tags/latest/vulnerability/details
[
  {
    "id": "CVE-2017-15650",
    "severity": 2,
    "package": "musl",
    "version": "1.1.15-r7",
    "description": "",
    "link": "https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-15650",
    "fixedVersion": "1.1.15-r8"
  }
]
```

# notary 설정
notary는 도커에서 만든 컨텐츠 사이닝 도구 임. 컨텐츠가 변조 되는지 확인이 가능한 보안 도구로 향후 컨테이너 이미지 사이닝을 하는데 도움이 될수 있을 것임.
* 참고 문서: https://docs.docker.com/engine/security/trust/content_trust/

하버에서 사용 예는 다음의 링크에 notary부분을 참조 한다
* 참고 문서: https://github.com/vmware/harbor/blob/master/docs/user_guide.md
