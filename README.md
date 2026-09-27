# week5_practice · Docker 실습

**nginx가 화면을 제공하고 Flask가 API를 처리합니다.** 같은 프로젝트를 Docker 명령으로 실행한 뒤 Compose로 바꿉니다.

| 구분 | 요청 | 확인 내용 |
|---|---|---|
| 기본 확인 | GET /api/health | `{"status":"ok"}` |
| 프로젝트 검색 | GET /api/projects?category=web&q=예약 | 조건에 맞는 프로젝트 |
| 소개글 분량 검사 | POST /api/analyze | 글자 수·단어 수·100~300자 충족 여부 |

## 1. 본인 저장소 준비

GitHub에서 [교수자 저장소](https://github.com/Mok2Lee/week5_practice)를 **Fork**합니다. VM에서는 아래 `내계정`을 바꿔 실행합니다. Codespaces는 본인 Fork에서 열면 파일이 이미 준비되어 있습니다.

```bash
git clone https://github.com/내계정/week5_practice.git
cd week5_practice
```

Fork 대신 본인의 빈 저장소로 옮기려면 교수자 저장소를 clone한 뒤 `git remote set-url origin https://github.com/내계정/week5_practice.git`을 실행합니다. Push에는 GitHub 인증이 필요합니다. Google 비밀번호를 입력하지 않습니다.

## 환경 선택

1. Windows에서는 NAS의 Docker Desktop 설치 파일로 설치하고 실행합니다. WSL 2 설치·업데이트나 재부팅을 요구하면 안내를 따릅니다.
2. `docker version`에서 Client와 Server, `docker compose version`에서 버전을 확인합니다. `docker run --rm hello-world`로 기본 실행을 확인합니다.
3. 교수자 안내에 따라 4장의 Ubuntu VM에서 Docker 설치·이미지 다운로드·빌드를 확인합니다. 막히면 Codespaces로 전환합니다. 인증서 검증을 끄지 않습니다.
4. Codespaces는 본인 저장소의 **Code → Codespaces → Create codespace on main**으로 엽니다. 제공된 devcontainer 설정이 Docker 환경을 준비합니다. 최초 준비에는 시간이 걸립니다.

NAS 설치 파일만으로 이미지와 Python 라이브러리까지 준비되지는 않습니다. Dockerfile의 패키지 설치는 이미지 빌드 중 한 번 실행하며, 학생이 VM의 Python 환경에 Flask를 별도 설치할 필요는 없습니다.

**실습 명령은 Ubuntu VM 또는 Codespaces의 Bash 터미널에서 실행합니다.** Windows에서는 Docker Desktop의 Linux 컨테이너 모드를 사용합니다.

Ubuntu VM 설치와 sudo 사용은 [설치 안내](docs/ubuntu-docker.md)를 확인합니다. 기본 Ubuntu 설치에서는 아래 docker 명령 앞에 sudo를 붙입니다.

## 브라우저 접속

- Windows Docker Desktop: `http://localhost:8080`
- Ubuntu VM: VirtualBox에서 기존 SSH 포트 전달은 유지하고 웹 규칙을 추가합니다. 호스트 IP `127.0.0.1`, 호스트 포트 `8080`, 게스트 포트 `8080`, TCP. Windows에서 `http://localhost:8080`으로 접속합니다.
- Codespaces: **Ports**의 `8080`에서 브라우저 열기를 선택합니다. 학생마다 주소가 다릅니다. 포트 공개 범위는 기본 Private로 유지합니다.

API 확인 주소는 위 웹 주소 뒤에 `/api/health`를 붙입니다. Codespaces 페이지 안의 API 요청도 같은 주소를 사용합니다.

## 2. 파일 역할

```text
api/                  Flask 코드와 이미지 제작 파일
  app.py              health·검색·분량 검사
  requirements.txt    Flask 버전
  Dockerfile          Python 환경과 실행 명령
nginx/
  nginx.conf          정적 파일 제공과 /api/ 전달
  html/               index.html, style.css, app.js
compose.starter.yaml  Compose 작성 틀
```

## 3. Docker 명령으로 실행

**프로젝트 최상위 폴더에서 실행합니다.** 아래 이름이 이미 있으면 이 실습에서 만든 컨테이너인지 먼저 확인하세요.

```bash
docker build -t week5-api:1.0 ./api
docker network create week5-net
docker run -d --name api --network week5-net week5-api:1.0
docker run -d --name week5-nginx --network week5-net -p 8080:80 \
  -v "$(pwd)/nginx/nginx.conf:/etc/nginx/nginx.conf:ro" \
  -v "$(pwd)/nginx/html:/usr/share/nginx/html:ro" nginx:stable-alpine
docker ps
curl -i http://localhost:8080/api/health
```

HTTP 200과 `{"status":"ok"}`를 확인한 뒤 메인페이지에서 검색과 분량 검사를 실행합니다. 검색 예시는 **웹 + 예약**, 분량 검사는 짧은 글과 100~300자 글을 각각 입력합니다.

Windows PowerShell에서 수동 실행할 경우 마지막 docker run 명령을 한 줄로 입력합니다.

```powershell
docker run -d --name week5-nginx --network week5-net -p 8080:80 -v "${PWD}/nginx/nginx.conf:/etc/nginx/nginx.conf:ro" -v "${PWD}/nginx/html:/usr/share/nginx/html:ro" nginx:stable-alpine
curl.exe -i http://localhost:8080/api/health
```

상태와 오류는 두 컨테이너에서 각각 확인합니다.

```bash
docker logs api
docker logs week5-nginx
```

### 코드 수정과 재실행

`api/app.py`의 예시 프로젝트 또는 분량 기준을 수정합니다. 파일 수정만으로 기존 이미지가 바뀌지 않는 점을 먼저 확인합니다.

```bash
docker build -t week5-api:1.0 ./api
docker stop api
docker rm api
docker run -d --name api --network week5-net week5-api:1.0
docker restart week5-nginx
curl -i http://localhost:8080/api/health
```

nginx를 재시작하면 새 API 컨테이너의 주소를 다시 찾습니다. health와 수정된 기능을 확인합니다. HTML·CSS는 파일 연결로 제공하므로 수정 후 브라우저를 새로고침합니다.

## 4. 같은 구성을 Compose로 실행

먼저 이번 실습에서 만든 컨테이너와 네트워크를 정리합니다.

```bash
docker stop week5-nginx api
docker rm week5-nginx api
docker network rm week5-net
cp compose.starter.yaml compose.yaml
```

Windows PowerShell에서는 마지막 줄을 `Copy-Item compose.starter.yaml compose.yaml`로 실행합니다.

compose.yaml의 TODO를 완성합니다. `build`는 `./api`, 공개 포트는 `8080:80`입니다. 완성 형태와 자신의 파일을 비교합니다.

```yaml
services:
  api:
    build: ./api
    image: week5-api:1.0
  nginx:
    image: nginx:stable-alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/html:/usr/share/nginx/html:ro
    depends_on:
      - api
```

```bash
docker compose config
docker compose up -d --build
docker compose ps
curl -i http://localhost:8080/api/health
docker compose logs api nginx
```

처음 실행 직후 응답이 없으면 `docker compose ps`와 로그를 확인하고 API 시작 후 다시 확인합니다. `depends_on`의 기본 형식은 시작 순서를 정하며 API 준비 완료까지 보장하지 않습니다.

**확인:** Docker 명령 방식과 같은 화면, 같은 검색 결과, 같은 분량 검사 결과가 표시되어야 합니다.

API 수정 후에는 다음 순서로 실행합니다.

```bash
docker compose up -d --build api
docker compose restart nginx
```

종료·재실행:

```bash
docker compose down
docker compose up -d
```

## 5. 확인 목록

- [ ] health의 HTTP 200·정상 응답
- [ ] 웹/예약 검색 결과와 결과 없는 검색 확인
- [ ] 소개글의 공백 포함 글자 수와 분량 결과 확인
- [ ] API 코드 수정 후 재빌드 결과 확인
- [ ] Docker 명령과 Compose의 설정 대응 설명
- [ ] 실습 후 `docker compose down`, Codespaces 사용 시 해당 Codespace 정지

## 오류 확인

| 증상 | 확인 |
|---|---|
| Docker Server 연결 실패 | Desktop 실행 여부 또는 Codespaces 준비 상태 |
| 이름이 이미 사용 중 | docker ps -a로 본 실습 컨테이너 확인 후 정리 |
| 8080 포트 사용 중 | 기존 실습 컨테이너 종료 또는 공개 포트 변경 |
| nginx 502 | API 로그, 같은 네트워크, api 이름, nginx 재시작 |
| /api/health가 404 | proxy_pass 끝의 / 유무, Flask 경로 |
| 이미지·pip 다운로드 실패 | 수업 안내에 따라 Codespaces로 전환 |

실습용 Flask 개발 서버를 사용합니다. 공개 운영 환경의 배포 구성은 별도로 다룹니다.

