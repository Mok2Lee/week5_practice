# week5_practice · Docker 실습

**Windows의 Docker Desktop과 PowerShell에서 진행합니다.** Flask와 nginx를 각각 컨테이너로 실행한 뒤 같은 프로젝트를 Compose로 실행합니다.

|구분|이름·주소|
|---|---|
|실습 폴더|`C:\lab\week5`|
|Flask 이미지 / 컨테이너|`week5_01` / `week5_api01`|
|nginx 이미지 / 컨테이너|`week5_02` / `week5_web01`|
|네트워크|`week5_net01`|
|웹 페이지|`http://localhost:8080`|

## 1. 실행 환경 확인

NAS에서 제공하는 Docker Desktop 설치 파일로 설치하고 실행합니다. 설치 과정에서 WSL 2 또는 재부팅을 요구하면 화면 안내를 따릅니다. **Linux 컨테이너 모드**에서 PowerShell을 열고 실행합니다.

```powershell
docker version
docker compose version
docker run --rm hello-world
```

Client와 Server 버전, Compose 버전, `Hello from Docker!`를 확인합니다. 설치 파일과 컨테이너 이미지 다운로드는 별도 과정입니다. 다운로드 실패 시 표시된 오류를 교수자에게 보여 줍니다.

## 2. 본인 저장소 준비

[교수자 저장소](https://github.com/Mok2Lee/week5_practice)를 **Fork**합니다. 아래 `내계정`을 본인 GitHub 계정명으로 바꿉니다. 저장소 이름은 week5_practice, 컴퓨터 안의 폴더 이름은 week5입니다.

```powershell
New-Item -ItemType Directory -Force C:\lab
Set-Location C:\lab
git clone https://github.com/내계정/week5_practice.git week5
Set-Location .\week5
code .
```

이후 명령은 VS Code의 **PowerShell 터미널**, `C:\lab\week5`에서 실행합니다. 같은 폴더가 이미 있으면 다시 clone하지 말고 기존 실습 폴더를 확인합니다.

Fork 대신 본인의 빈 저장소로 옮길 때는 교수자 저장소를 clone한 뒤 `git remote set-url origin https://github.com/내계정/week5_practice.git`로 연결을 바꿉니다. Push 인증에는 GitHub의 로그인 안내를 따릅니다.

## 3. 프로젝트 구성

```text
api/
  app.py              연결 확인·프로젝트 검색·소개글 분량 검사
  requirements.txt    Flask 버전
  Dockerfile          Flask 이미지 제작
nginx/
  Dockerfile          nginx 이미지 제작
  nginx.conf          화면 제공과 API 요청 전달
  html/               index.html, style.css, app.js
compose.starter.yaml  Compose 작성 틀
```

nginx는 HTML 화면을 제공하고 `/api/` 요청을 `week5_api01:5000`의 Flask로 전달합니다. **이미지에 파일을 복사하므로 소스를 수정한 뒤에는 해당 이미지를 다시 빌드**합니다.

## 4. Docker 명령으로 실행

### 이미지 제작

```powershell
docker build -t week5_01 ./api
docker build -t week5_02 ./nginx
docker images
```

Flask 이미지와 nginx 이미지가 모두 만들어졌는지 확인합니다. Flask 설치는 Dockerfile의 빌드 과정에서 수행하므로 Windows에 Flask를 따로 설치하지 않습니다.

### 네트워크와 컨테이너 실행

```powershell
docker network create week5_net01
docker run -d --name week5_api01 --network week5_net01 week5_01
docker run -d --name week5_web01 --network week5_net01 -p 8080:80 week5_02
docker ps
curl.exe -i http://localhost:8080/api/health
```

**HTTP 200**과 `{"status":"ok"}`를 확인합니다. 실행 직후 응답이 없으면 잠시 후 다시 실행합니다. 컨테이너 이름이나 8080번 포트가 이미 사용 중이면 기존 자원을 먼저 확인하며, 다른 프로젝트의 자원은 삭제하지 않습니다.

### 기능 확인

브라우저에서 `http://localhost:8080`을 엽니다.

|기능|입력|확인 결과|
|---|---|---|
|연결 확인|`/api/health`|`status: ok`|
|프로젝트 검색|분야 웹, 검색어 예약|교내 공간 예약 1개|
|소개글 분량 검사|짧은 글 / 100~300자 글|글자 수·단어 수·권장 분량 충족 여부|

```powershell
docker logs --tail 8 week5_api01
docker logs --tail 8 week5_web01
```

검색은 `GET /api/projects`, 분량 검사는 `POST /api/analyze`로 요청합니다. 두 컨테이너의 로그에서 같은 요청의 200 응답을 확인합니다.

## 5. 코드 수정과 컨테이너 교체

`api/app.py`에 예시 프로젝트를 추가합니다. **파일만 수정했을 때 기존 컨테이너의 결과가 그대로**인지 확인한 뒤 실행합니다.

```powershell
docker build -t week5_01 ./api
docker stop week5_api01
docker rm week5_api01
docker run -d --name week5_api01 --network week5_net01 week5_01
docker restart week5_web01
curl.exe -i http://localhost:8080/api/health
```

nginx를 재시작해 새 API 컨테이너 주소를 다시 찾습니다. 검색 결과에 추가한 프로젝트가 표시되는지 확인합니다.

HTML·CSS를 수정했다면 웹 이미지를 다시 빌드하고 웹 컨테이너를 교체합니다.

```powershell
docker build -t week5_02 ./nginx
docker stop week5_web01
docker rm week5_web01
docker run -d --name week5_web01 --network week5_net01 -p 8080:80 week5_02
```

## 6. 같은 구성을 Compose로 실행

수동 실행과 Compose가 **같은 컨테이너·네트워크 이름**을 사용합니다. 먼저 이번 실습에서 만든 수동 자원을 정리합니다.

```powershell
docker stop week5_web01 week5_api01
docker rm week5_web01 week5_api01
docker network rm week5_net01
Copy-Item compose.starter.yaml compose.yaml
```

compose.yaml의 TODO를 완성하고 아래 설정과 비교합니다.

```yaml
name: week5
services:
  api:
    build: ./api
    image: week5_01
    container_name: week5_api01
  nginx:
    build: ./nginx
    image: week5_02
    container_name: week5_web01
    ports:
      - "8080:80"
    depends_on:
      - api
networks:
  default:
    name: week5_net01
```

`api`와 `nginx`은 Compose의 서비스 이름입니다. `container_name`은 화면에 표시되는 컨테이너 이름, `image`는 만들어 사용할 이미지 이름입니다. `networks.default.name`으로 두 서비스가 연결될 네트워크 이름을 정합니다.

```powershell
docker compose config
docker compose up -d --build
docker compose ps
curl.exe -i http://localhost:8080/api/health
docker compose logs --tail 8 api nginx
```

검색·분량 검사 결과가 수동 실행 때와 같은지 확인합니다. 기본 `depends_on`은 시작 순서만 정하므로 Flask가 준비되기 전에는 응답이 잠시 없을 수 있습니다.

## 7. 수정·종료·재실행

API 수정 후:

```powershell
docker compose up -d --build api
docker compose restart nginx
```

HTML·CSS 또는 nginx 설정 수정 후:

```powershell
docker compose up -d --build nginx
```

종료와 재실행:

```powershell
docker compose down
docker compose up -d
docker compose ps
```

실습을 마치면 `docker compose down`으로 종료합니다.

## 오류 확인

|증상|확인할 내용|
|---|---|
|Docker Server 연결 실패|Docker Desktop 실행·엔진 준비 상태|
|이름이 이미 사용 중|`docker ps -a`와 `docker network ls`, 수동 실행 정리 여부|
|8080 포트 사용 중|실습·과제 중 이미 실행 중인 컨테이너|
|nginx 502|API 로그, 같은 네트워크, `week5_api01` 이름, nginx 재시작|
|소스 수정이 반영되지 않음|수정한 api 또는 nginx 이미지 재빌드 여부|
|실습에서 과제로 바꾼 뒤 버튼이 반응하지 않음|Ctrl+Shift+R로 강력 새로고침|

실습용 Flask 개발 서버를 사용합니다. 공개 운영 환경의 배포 구성은 별도로 다룹니다.

Ubuntu VM 설치·연결 확인이 필요한 경우에만 [Ubuntu 참고 안내](docs/ubuntu-docker.md)를 확인합니다. 설치나 다운로드가 실패하면 Windows PowerShell 실습으로 진행합니다.
