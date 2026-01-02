# common_fastapi 중앙화된 설정 사용 가이드

## 구조

```
common_fastapi/
├── .env                           # ✅ 공통 환경 변수 (API_KEY, DB_URL)
└── common_fastapi/
    └── shared/
        ├── config.py              # ✅ 공통 환경 변수 로드
        ├── db.py                  # ✅ DB 연결 풀 관리
        ├── logger.py              # ✅ 로거
        └── constant.py            # ✅ 상수

gigchat_fastapi/
├── .env                           # ✅ 프로젝트별 환경 변수 (LOG_PATH 등)
├── main.py                        # common_fastapi 사용
└── route/
    └── chat.py                    # common_fastapi.shared.db 사용
```

## 환경 변수 분리

### common_fastapi/.env (공통 설정)
```bash
# 모든 프로젝트에서 공유하는 설정
OPENAI_API_KEY=sk-xxxxxxxxxxxxx
DB_URL=postgresql://user:password@host/database
```

### gigchat_fastapi/.env (프로젝트별 설정)
```bash
# 프로젝트별 설정
LOG_PATH=c:\logs\gigchat
DEFAULT_TIMEZONE=Asia/Seoul
```

## 사용 방법

### 1. main.py에서 설정
```python
import os
from dotenv import load_dotenv
from common_fastapi.shared.db import init_db_pool, close_db_pool
from common_fastapi.shared.config import validate_env

# ✅ 프로젝트별 .env 로드 (LOG_PATH 등)
load_dotenv()

@asynccontextmanager
async def lifespan(app: FastAPI):
    # ✅ 공통 환경 변수 검증 (API_KEY, DB_URL)
    validate_env()
    
    # ✅ DB 풀 초기화 (common_fastapi/.env 사용)
    pool = await init_db_pool()
    
    # ✅ 프로젝트별 LOG_PATH 사용
    log_path = os.getenv("LOG_PATH", "./logs")
    
    try:
        yield
    finally:
        await close_db_pool()
```

### 2. route/chat.py에서 사용
```python
from common_fastapi.shared.db import get_pool

@router.post("/chat")
async def chat(request: ChatRequest):
    pool = get_pool()  # ✅ 공통 DB 풀 사용
    
    async with pool.acquire() as conn:
        result = await conn.fetch("SELECT * FROM jobs")
    
    return {"result": result}
```

### 3. LLM 사용 (자동으로 OPENAI_API_KEY 사용)
```python
from common_fastapi.ai.llm_openai import LLMClient

# ✅ common_fastapi/.env의 OPENAI_API_KEY 자동 사용
llm = LLMClient()
response = llm.chat([{"role": "user", "content": "안녕하세요"}])
```

## 장점

1. **공통 설정 중앙화**: API 키, DB URL은 한 곳에서만 관리
2. **프로젝트별 설정 유지**: LOG_PATH 등은 각 프로젝트에서 관리
3. **유연성**: 프로젝트마다 다른 로그 경로 사용 가능
4. **보안**: API 키는 common_fastapi/.env 하나만 관리

## 주의사항

- `common_fastapi/.env`: API_KEY, DB_URL만 포함 (Git 제외)
- `프로젝트/.env`: LOG_PATH, TIMEZONE 등 프로젝트별 설정 (Git 제외 가능)
- 새 프로젝트 추가 시: 프로젝트별 `.env` 생성, 공통 설정은 `common_fastapi` 참조
