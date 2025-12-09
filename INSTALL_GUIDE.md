# common_fastapi 설치 및 사용 가이드

## 1단계: 설치 완료 확인

설치가 완료되면 다음과 같이 표시됩니다:
```
Successfully installed common-fastapi-0.1.0
```

## 2단계: gigchat_fastapi에서 import 변경

### 변경 전:
```python
from common.constant import Const
from common.logger import logger
from common.util import format_datetime
from llm.openai import LLMClient
from restful.resp import rsObj, rsError
from restful.rqst import ChatRequest
```

### 변경 후:
```python
from common_fastapi.shared.constant import Const
from common_fastapi.shared.logger import logger
from common_fastapi.shared.util import format_datetime
from common_fastapi.ai.llm_openai import LLMClient
from common_fastapi.restful.resp import rsObj, rsError
from common_fastapi.restful.rqst import ChatRequest
```

## 3단계: 기존 폴더 삭제 (선택)

import를 모두 변경한 후 테스트가 완료되면:
```cmd
cd c:\Src\Git\gigchat\gigchat_fastapi
rmdir /s /q common
rmdir /s /q llm
rmdir /s /q restful
```

## 4단계: 다른 프로젝트에서도 사용

다른 FastAPI 프로젝트에서도 동일하게 설치:
```cmd
cd c:\Src\Git\다른프로젝트
pip install -e c:\Src\Git\common_fastapi
```

## 주의사항

1. **editable 모드 (-e)**: common_fastapi 코드 수정 시 즉시 반영됨
2. **환경변수**: .env 파일에 필요한 변수 설정
   - OPENAI_API_KEY
   - LOG_PATH (선택사항)
   - DEFAULT_TIMEZONE (기본값: Asia/Seoul)
3. **의존성**: requirements.txt에 자동으로 추가되지 않으므로 문서화 필요

## 설치 확인

```python
# 터미널에서 테스트
python -c "from common_fastapi.common.constant import Const; print(Const.CODE_OK)"
# 출력: 0
```
