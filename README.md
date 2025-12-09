# Common FastAPI

FastAPI 프로젝트용 공통 모듈 라이브러리

## 포함 모듈

- **ai**: AI 클라이언트
- **restful**: 요청/응답 모델
- **shared**: 상수, 로거, 유틸리티, 임베딩

## 설치

### 개발 모드 (권장)
```bash
pip install -e c:\Src\Git\common_fastapi
```

### 일반 설치
```bash
pip install c:\Src\Git\common_fastapi
```

## 사용 예시

```python
from common_fastapi.shared.constant import Const
from common_fastapi.shared.logger import logger
from common_fastapi.ai.llm_openai import LLMClient
from common_fastapi.restful.resp import rsObj, rsError

# 상수 사용
print(Const.CODE_OK)

# 로거 사용
logger.info("애플리케이션 시작")

# LLM 클라이언트
llm = LLMClient()
response = llm.chat([{"role": "user", "content": "안녕하세요"}])

# 응답 객체
return rsObj({"result": "success"})
```

## 버전

- 0.1.0: 초기 버전
