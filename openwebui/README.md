# openwebui-custom

ghcr.io/open-webui/open-webui:main 기반 커스텀 이미지.
한국어 환경 설정, OCR, PyTorch Blackwell 호환을 빌드 타임에 처리한다.

---

## 베이스 이미지

```
ghcr.io/open-webui/open-webui:main
```

---

## 추가된 내용

| 구분 | 내용 |
|------|------|
| 한국어 로케일 | `ko_KR.UTF-8` 생성 |
| 한국어 폰트 | `fonts-nanum` |
| OCR 엔진 | `tesseract-ocr` + 한국어(`tesseract-ocr-kor`) + 영어(`tesseract-ocr-eng`) 데이터 |
| PyTorch | 2.7+ / CUDA 12.8 — RTX 50 Blackwell (CC 12.0) 호환 |

별도 시작 스크립트 없음 — 베이스 이미지 기본 CMD 그대로 사용.

---

## 데이터 영속성

`DATA_DIR` 을 gcube dropbox-storage 마운트 경로로 직접 지정한다.
OpenWebUI가 처음부터 해당 경로에 읽고 쓰므로 컨테이너가 재시작되어도 데이터가 유지된다.

```
DATA_DIR=/workplace/service_v1/data     ← webui.db, 설정 등
UPLOAD_DIR=/workplace/service_v1/uploads ← 업로드 파일
```

---

## 환경변수

| KEY | 기본값 | 설명 |
|-----|--------|------|
| `OLLAMA_BASE_URL` | `http://localhost:11434` | Ollama 연동 주소 |
| `DEFAULT_MODELS` | `deepseek-r1:8b` | UI 기본 선택 모델 |
| `DATA_DIR` | `/workplace/service_v1/data` | DB 및 설정 저장 경로 |
| `UPLOAD_DIR` | `/workplace/service_v1/uploads` | 업로드 파일 저장 경로 |
| `OLLAMA_REQUEST_TIMEOUT` | `1800` | Ollama 추론 타임아웃(초) |
| `TIMEOUT` | `3600` | 전체 응답 타임아웃(초) |
| `UPLOAD_TIMEOUT` | `3600` | 업로드 타임아웃(초) |
| `RAG_EMBEDDING_ENGINE` | `ollama` | 임베딩 엔진 |
| `RAG_EMBEDDING_MODEL` | `nomic-embed-text` | 임베딩 모델 |
| `RAG_EMBEDDING_CHUNK_SIZE` | `1000` | 청크 크기 |
| `RAG_EMBEDDING_BATCH_SIZE` | `2` | 배치 크기 |
| `ENABLE_OCR` | `true` | OCR 활성화 |
| `OCR_ENGINE` | `tesseract` | OCR 엔진 |
| `OCR_LANG` | `kor+eng` | OCR 언어 |
| `VECTOR_DB` | `chroma` | 벡터 DB |
| `FILE_UPLOAD_MAX_SIZE` | `2147483648` | 최대 업로드 크기 (2GB) |
| `MAX_CONTENT_LENGTH` | `2147483648` | 최대 콘텐츠 크기 (2GB) |
| `ENABLE_CHUNKED_UPLOAD` | `true` | 청크 업로드 활성화 |

---

## 포트

| 포트 | 용도 |
|------|------|
| `8080` | OpenWebUI (gcube 서비스 URL로 외부 노출) |

---

## gcube dropbox-storage 마운트

| 저장소 | 마운트 주소 |
|--------|-------------|
| dropbox-storage | `/workplace/service_v1/data` |
| dropbox-storage | `/workplace/service_v1/uploads` |

---

## 사용 방법

### 1. GitHub Actions로 이미지 빌드

`openwebui/` 하위 파일을 수정 후 main 브랜치에 push하면 자동으로 빌드되어 ghcr.io에 올라간다.

```
git push origin main
→ GitHub Actions 자동 실행
→ ghcr.io/{username}/openwebui-custom:latest 생성
```

### 2. gcube 워크로드 컨테이너 설정

| 항목 | 값 |
|------|-----|
| 이미지 | `ghcr.io/{username}/openwebui-custom:latest` |
| 컨테이너 포트 | `8080` |
| 컨테이너 명령 | **없음** |

환경변수:

| KEY | VALUE |
|-----|-------|
| `OLLAMA_BASE_URL` | `http://localhost:11434` |
| `DEFAULT_MODELS` | `deepseek-r1:8b` |

개인저장소(dropbox-storage) 마운트:

| 저장소 | 마운트 주소 |
|--------|-------------|
| dropbox-storage | `/workplace/service_v1/data` |
| dropbox-storage | `/workplace/service_v1/uploads` |

### 3. 서비스 접속

배포 완료 후 gcube 워크로드의 서비스 URL로 접속하면 OpenWebUI 로그인 화면이 나온다.

- 최초 접속 시 회원가입 → 첫 번째 계정이 자동으로 관리자(Admin) 권한 부여
- 이후 접속은 등록한 계정으로 로그인

### 4. Ollama 모델 연동 확인

로그인 후 채팅 화면 상단 모델 선택란에 `deepseek-r1:8b` 가 표시되면 Ollama와 정상 연동된 것이다.

모델이 표시되지 않는 경우:
- Ollama 컨테이너 로그에서 모델 pull 완료 여부 확인
- 관리자 패널 → Settings → Connections → Ollama API URL이 `http://localhost:11434` 인지 확인

### 5. 데이터 유지 확인

컨테이너를 재시작해도 아래 경로가 dropbox-storage에 마운트되어 있으므로 데이터가 유지된다.

```
/workplace/service_v1/data     ← 대화 기록, 계정 정보, 설정
/workplace/service_v1/uploads  ← 업로드한 파일
```
