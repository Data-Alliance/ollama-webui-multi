# ollama-custom

ollama/ollama:latest 기반 커스텀 이미지.
gcube 워크로드 환경변수로 주입한 모델명을 컨테이너 기동 시 자동으로 설치한다.

---

## 베이스 이미지

```
ollama/ollama:latest
```

---

## 추가된 내용

| 구분 | 내용 |
|------|------|
| 추가 패키지 | `curl` — 헬스체크 대기 루프에서 서버 Ready 확인용 |
| ENTRYPOINT | `/entrypoint.sh` — 모델 자동 pull 처리 |

---

## entrypoint.sh 동작 순서

```
1. ollama serve 백그라운드 기동
2. curl로 :11434 헬스체크 — Ready 될 때까지 2초 간격 대기 (최대 60초)
3. OLLAMA_MODEL 환경변수 값을 공백 기준으로 분리해서 순서대로 ollama pull
4. 모든 모델 준비 완료 → 서버 foreground 유지
```

> OLLAMA_MODEL이 비어있으면 모델 pull 없이 서버만 기동됨

---

## 환경변수

| KEY | 기본값 | 설명 |
|-----|--------|------|
| `OLLAMA_HOST` | `0.0.0.0` | 이미지 내 기본값. 모든 인터페이스 수신 (gcube 내부 통신 허용) |
| `OLLAMA_MODEL` | `""` | gcube 환경변수에서 주입. 설치할 모델명 |

**OLLAMA_MODEL 입력 예시**

```
# 단일 모델
deepseek-r1:8b

# 복수 모델 (공백 구분)
deepseek-r1:8b nomic-embed-text
```

---

## 포트

| 포트 | 용도 |
|------|------|
| `11434` | Ollama API (gcube 내부 통신 전용, 외부 노출 불필요) |

---

## GPU 호환

| GPU 세대 | Compute Capability | 최소 CUDA | 최소 드라이버 |
|----------|--------------------|-----------|---------------|
| RTX 40 시리즈 (Ada Lovelace) | 8.9 | 12.0 | 525+ |
| RTX 50 시리즈 (Blackwell) | 12.0 | 12.8 | 570+ |

ollama:latest 이미지가 런타임에 GPU를 감지하여 cuda_v12 / cuda_v13 백엔드를 자동 선택하므로 단일 이미지로 RTX 40/50 모두 동작한다.

---

## 시연용 추천 모델

| 모델 | VRAM | 특징 |
|------|------|------|
| `deepseek-r1:8b` | 6GB | `<think>` 블록으로 추론 과정 실시간 노출 |
| `qwen2.5:14b` | 10GB | 한국어 응답 품질 오픈소스 최상위 |
| `llama3.3:70b` | 40GB+ | 오픈소스 최고 수준 (H200 기준 권장) |

---

## 사용 방법

### 1. GitHub Actions로 이미지 빌드

`ollama/` 하위 파일을 수정 후 main 브랜치에 push하면 자동으로 빌드되어 ghcr.io에 올라간다.

```
git push origin main
→ GitHub Actions 자동 실행
→ ghcr.io/{username}/ollama-custom:latest 생성
```

### 2. gcube 워크로드 컨테이너 설정

| 항목 | 값 |
|------|-----|
| 이미지 | `ghcr.io/{username}/ollama-custom:latest` |
| 컨테이너 포트 | `11434` |
| 컨테이너 명령 | **없음** |

환경변수:

| KEY | VALUE |
|-----|-------|
| `OLLAMA_HOST` | `0.0.0.0` |
| `OLLAMA_MODEL` | `deepseek-r1:8b` |

개인저장소: **없음**

### 3. 모델 변경 방법

gcube 워크로드 환경변수 탭에서 `OLLAMA_MODEL` 값만 바꾸고 워크로드를 재배포하면 된다.
이미지 재빌드 없이 모델 전환 가능.

```
# 단일 모델로 변경
OLLAMA_MODEL=qwen2.5:14b

# RAG용 임베딩 모델 함께 설치 (공백 구분)
OLLAMA_MODEL=deepseek-r1:8b nomic-embed-text
```

### 4. 배포 후 동작 확인

컨테이너 로그에서 아래 메시지가 나오면 정상:

```
[INFO] Ollama server is ready.
[INFO] Pulling model: deepseek-r1:8b
[INFO] Model pull complete: deepseek-r1:8b
[INFO] All models ready. Keeping server alive...
```

최초 배포 시 모델 다운로드로 수 분 소요. 이후 재배포 시에는 이미 다운받은 모델이 없으면 다시 pull한다.
