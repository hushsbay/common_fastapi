# NCP Ubuntu FastAPI 프로젝트 배포 가이드

## 배포 구조

```
/app/fastapi/           # 공통 FastAPI 환경
├── venv/              # 공통 가상환경 (모든 FastAPI 프로젝트 공유)
└── common_fastapi/    # 공통 모듈

/app/suwon/            # 개별 FastAPI 프로젝트
└── gigchat_fastapi/   # gigchat FastAPI 서버
```

- **포트**: gigchat_fastapi는 8082 포트 사용
- **호출자**: gigchat_nextjs → http://localhost:8082

---

## 1단계: 서버 접속 및 디렉토리 생성

```bash
# NCP Ubuntu 서버 접속
ssh username@your-server-ip

# 디렉토리 생성
sudo mkdir -p /app/fastapi
sudo mkdir -p /app/suwon
sudo chown -R $USER:$USER /app/fastapi
sudo chown -R $USER:$USER /app/suwon
```

---

## 2단계: 공통 가상환경 설정

```bash
# Python3 및 venv 설치 확인
sudo apt update
sudo apt install -y python3 python3-pip python3-venv

# 공통 가상환경 생성
cd /app/fastapi
python3 -m venv venv

# 가상환경 활성화
source /app/fastapi/venv/bin/activate
```

---

## 3단계: common_fastapi 배포

```bash
# 로컬에서 서버로 파일 전송 (Windows PowerShell에서 실행)
# Option 1: rsync (WSL 사용 시)
wsl rsync -avz --exclude='__pycache__' --exclude='.venv' --exclude='*.egg-info' \
  /mnt/c/Src/Git/common_fastapi/ username@server-ip:/app/fastapi/common_fastapi/

# Option 2: scp (직접 전송)
scp -r c:\Src\Git\common_fastapi username@server-ip:/tmp/
# 서버에서 이동
ssh username@server-ip
sudo mv /tmp/common_fastapi /app/fastapi/
sudo chown -R $USER:$USER /app/fastapi/common_fastapi

# 서버에서 계속 작업
cd /app/fastapi
source venv/bin/activate

# common_fastapi editable 모드 설치
pip install -e /app/fastapi/common_fastapi

# 의존성 설치
pip install -r /app/fastapi/common_fastapi/requirements.txt
```

---

## 4단계: gigchat_fastapi 배포

```bash
# 로컬에서 서버로 파일 전송
# Option 1: rsync (WSL 사용 시)
wsl rsync -avz --exclude='__pycache__' --exclude='.venv' --exclude='*.egg-info' \
  /mnt/c/Src/Git/gigchat/gigchat_fastapi/ username@server-ip:/app/suwon/gigchat_fastapi/

# Option 2: scp (직접 전송)
scp -r c:\Src\Git\gigchat\gigchat_fastapi username@server-ip:/tmp/
# 서버에서 이동
ssh username@server-ip
sudo mv /tmp/gigchat_fastapi /app/suwon/
sudo chown -R $USER:$USER /app/suwon/gigchat_fastapi

# 서버에서 계속 작업
cd /app/suwon/gigchat_fastapi
source /app/fastapi/venv/bin/activate  # 공통 venv 사용

# 의존성 설치
pip install -r requirements.txt
```

---

## 5단계: 환경 변수 설정

### common_fastapi 환경 변수

```bash
# common_fastapi 환경 변수 (API 키, DB 정보)
sudo nano /app/fastapi/common_fastapi/.env
```

```env
# OpenAI API
OPENAI_API_KEY=sk-your-api-key

# Database
DB_HOST=your-db-host
DB_PORT=5432
DB_NAME=your-db-name
DB_USER=your-db-user
DB_PASSWORD=your-db-password

# Timezone
DEFAULT_TIMEZONE=Asia/Seoul
```

### gigchat_fastapi 환경 변수

```bash
# gigchat_fastapi 환경 변수 (로그 경로 등)
sudo nano /app/suwon/gigchat_fastapi/.env
```

```env
# Log Path (프로젝트별)
LOG_PATH=/app/suwon/gigchat_fastapi/logs

# Port (옵션)
PORT=8082
```

---

## 6단계: systemd 서비스 등록

### gigchat_fastapi 서비스 생성

```bash
sudo nano /etc/systemd/system/gigchat-fastapi.service
```

```ini
[Unit]
Description=GigChat FastAPI Server
After=network.target

[Service]
Type=simple
User=your-username
WorkingDirectory=/app/suwon/gigchat_fastapi
Environment="PATH=/app/fastapi/venv/bin"
ExecStart=/app/fastapi/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8082 --reload
Restart=always
RestartSec=10

# 로그 설정
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

**주의**: 
- `your-username`을 실제 사용자 이름으로 변경
- `--reload` 옵션은 개발 시에만 사용 (프로덕션에서는 제거)

---

## 7단계: 서비스 시작 및 확인

```bash
# systemd 데몬 리로드
sudo systemctl daemon-reload

# 서비스 활성화 (부팅 시 자동 시작)
sudo systemctl enable gigchat-fastapi

# 서비스 시작
sudo systemctl start gigchat-fastapi

# 서비스 상태 확인
sudo systemctl status gigchat-fastapi

# 로그 확인
sudo journalctl -u gigchat-fastapi -f
```

---

## 8단계: 방화벽 설정

```bash
# UFW 방화벽에서 8082 포트 허용
sudo ufw allow 8082/tcp

# 방화벽 상태 확인
sudo ufw status
```

---

## 9단계: 테스트

```bash
# 서버에서 로컬 테스트
curl http://localhost:8082

# 외부에서 테스트 (서버 IP가 공개된 경우)
curl http://your-server-ip:8082
```

---

## 서비스 관리 명령어

```bash
# 서비스 중지
sudo systemctl stop gigchat-fastapi

# 서비스 재시작
sudo systemctl restart gigchat-fastapi

# 서비스 상태 확인
sudo systemctl status gigchat-fastapi

# 로그 실시간 확인
sudo journalctl -u gigchat-fastapi -f

# 로그 최근 50줄 확인
sudo journalctl -u gigchat-fastapi -n 50
```

---

## 코드 업데이트 시

```bash
# 로컬에서 파일 재전송 (rsync 사용 - 변경된 파일만)
wsl rsync -avz --exclude='__pycache__' --exclude='.venv' --exclude='*.egg-info' \
  /mnt/c/Src/Git/gigchat/gigchat_fastapi/ username@server-ip:/app/suwon/gigchat_fastapi/

# 서버에서 서비스 재시작
ssh username@server-ip
sudo systemctl restart gigchat-fastapi
```

### common_fastapi 업데이트 시

```bash
# 로컬에서 common_fastapi 재전송
wsl rsync -avz --exclude='__pycache__' --exclude='.venv' --exclude='*.egg-info' \
  /mnt/c/Src/Git/common_fastapi/ username@server-ip:/app/fastapi/common_fastapi/

# 서버에서 pip 재설치 (editable 모드이므로 자동 반영되지만 안전하게)
ssh username@server-ip
source /app/fastapi/venv/bin/activate
pip install -e /app/fastapi/common_fastapi

# gigchat_fastapi 재시작
sudo systemctl restart gigchat-fastapi
```

---

## 추가 FastAPI 프로젝트 배포 시

동일한 `/app/fastapi/venv` 공통 가상환경을 사용:

```bash
# 새 프로젝트 배포
scp -r c:\Src\Git\new_project\new_fastapi username@server-ip:/app/suwon/

# 서버에서
cd /app/suwon/new_fastapi
source /app/fastapi/venv/bin/activate  # 공통 venv 재사용
pip install -r requirements.txt

# systemd 서비스 생성 (포트만 변경)
sudo nano /etc/systemd/system/new-fastapi.service
# ExecStart에서 --port 8083 등으로 변경
```

---

## 트러블슈팅

### 1. 포트가 이미 사용 중인 경우

```bash
# 포트 사용 프로세스 확인
sudo lsof -i :8082

# 프로세스 종료
sudo kill -9 [PID]
```

### 2. 서비스가 시작되지 않는 경우

```bash
# 로그 상세 확인
sudo journalctl -u gigchat-fastapi -xe

# 수동으로 uvicorn 실행하여 에러 확인
cd /app/suwon/gigchat_fastapi
source /app/fastapi/venv/bin/activate
uvicorn main:app --host 0.0.0.0 --port 8082
```

### 3. 모듈을 찾을 수 없는 경우

```bash
# 가상환경 활성화 후 패키지 확인
source /app/fastapi/venv/bin/activate
pip list | grep common-fastapi
pip list | grep sentence-transformers

# 없으면 재설치
pip install -e /app/fastapi/common_fastapi
pip install -r /app/suwon/gigchat_fastapi/requirements.txt
```

### 4. DB 연결 오류

```bash
# .env 파일 확인
cat /app/fastapi/common_fastapi/.env

# DB 연결 테스트
source /app/fastapi/venv/bin/activate
python -c "import asyncpg; print('asyncpg OK')"
```

---

## 디렉토리 권한 확인

```bash
# 디렉토리 소유자 확인
ls -la /app/fastapi
ls -la /app/suwon

# 권한 문제 시 수정
sudo chown -R your-username:your-username /app/fastapi
sudo chown -R your-username:your-username /app/suwon
```

---

## 참고: Git을 통한 배포 (선택사항)

서버에 Git을 설치하고 직접 pull하는 방법도 가능:

```bash
# 서버에서 Git 설치
sudo apt install -y git

# 초기 클론
cd /app/fastapi
git clone https://github.com/your-repo/common_fastapi.git

cd /app/suwon
git clone https://github.com/your-repo/gigchat_fastapi.git

# 업데이트 시
cd /app/fastapi/common_fastapi
git pull

cd /app/suwon/gigchat_fastapi
git pull
sudo systemctl restart gigchat-fastapi
```
