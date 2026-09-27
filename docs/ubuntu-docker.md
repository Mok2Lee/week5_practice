# Ubuntu VM의 Docker 설치 확인

교수자가 설치 가능한 환경임을 확인한 경우에만 진행합니다. 이미 Docker가 정상 실행되면 설치를 반복하지 않습니다. 실패하면 오류를 남기고 Windows Docker Desktop과 PowerShell 실습으로 전환합니다.

1. [Docker의 Ubuntu 설치 안내](https://docs.docker.com/engine/install/ubuntu/)에서 현재 Ubuntu 버전의 지원 여부와 기존 Docker 패키지 유무를 확인합니다.
2. 공식 안내의 **Install using the apt repository**에서 키와 저장소를 등록합니다. 아래는 새 실습 VM용 순서입니다.

```bash
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

3. Docker 서비스와 실제 이미지 실행을 확인합니다.

```bash
sudo systemctl start docker
sudo docker version
sudo docker compose version
sudo docker run --rm hello-world
```

**Ubuntu 기본 설치에서는 Docker 명령 앞에 sudo를 붙입니다.** 이 문서는 Ubuntu의 설치·연결 확인용 참고입니다. Windows PowerShell의 폴더·파일 명령은 Ubuntu 터미널에 그대로 입력하지 않습니다. 예: `sudo docker compose up -d --build`. Docker 그룹을 변경하지 않아도 실습할 수 있습니다. Windows PowerShell에서는 sudo를 붙이지 않습니다.

설치가 되어도 이미지 다운로드나 Flask 빌드 중 pip 다운로드가 실패할 수 있습니다. 이 경우 Windows의 C:\lab\week5 폴더에서 같은 저장소로 계속합니다. 학생 PC에서 인증서 검증을 끄는 방법은 사용하지 않습니다.
