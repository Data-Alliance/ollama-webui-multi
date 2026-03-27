# gcube OpenWebUI + Ollama 멀티컨테이너 워크로드

OpenWebUI와 Ollama를 gcube 플랫폼에서 멀티컨테이너로 배포하는 레포지토리.
git push 한 번으로 커스텀 이미지가 자동 빌드되고, gcube에서 바로 가져다 쓸 수 있다.

---

## 레포지토리 구조

```
.
├── .github/workflows/
│   ├── build-ollama.yml       ← Ollama 이미지 자동 빌드 + ghcr.io 푸시
│   └── build-openwebui.yml    ← OpenWebUI 이미지 자동 빌드 + ghcr.io 푸시
├── ollama/
│   ├── Dockerfile
│   ├── entrypoint.sh
│   └── README.md              ← Ollama 이미지 상세 설명
└── openwebui/
    ├── Dockerfile
    └── README.md              ← OpenWebUI 이미지 상세 설명
```

---

## Step 1 — 레포지토리 코드 push

로컬에서 이 레포지토리를 clone하거나 새 레포를 만들어 파일을 올린다.

```bash
git add .
git commit -m "init"
git push origin main
```

push가 완료되면 GitHub Actions가 자동으로 실행되어 두 이미지를 빌드하고 ghcr.io에 푸시한다.

---

## Step 2 — GitHub Actions 빌드 확인

1. GitHub 레포 상단 **Actions** 탭 클릭
2. `Build Ollama Custom Image`, `Build OpenWebUI Custom Image` 워크플로우가 각각 실행 중인지 확인
3. 두 워크플로우 모두 초록 체크(✓)가 되면 이미지 빌드 완료

빌드가 완료되면 아래 주소로 이미지가 생성된다.

```
ghcr.io/{github계정명}/ollama-custom:latest
ghcr.io/{github계정명}/openwebui-custom:latest
```

---

## Step 3 — GitHub Packages를 Public으로 공개

gcube가 인증 없이 이미지를 pull하려면 패키지가 public이어야 한다.

1. GitHub 레포 메인 페이지 우측 사이드바 **Packages** 클릭
2. `ollama-custom` 패키지 클릭 → 우측 하단 **Package settings** 클릭
3. **Danger Zone** → `Change visibility` → **Public** 선택 후 확인
4. `openwebui-custom` 패키지도 동일하게 반복

---

## Step 4 — gcube 워크로드 생성

gcube 콘솔에서 새 워크로드를 생성한다.
컨테이너를 아래 순서대로 두 개 추가한다.

---

### 컨테이너 1 — OpenWebUI

| 항목 | 값 |
|------|-----|
| 이미지 | `ghcr.io/{github계정명}/openwebui-custom:latest` |
| 컨테이너 포트 | `8080` |
| 컨테이너 명령 | 없음 |

**환경변수:**

| KEY | VALUE |
|-----|-------|
| `OLLAMA_BASE_URL` | `http://localhost:11434` |
| `DEFAULT_MODELS` | `deepseek-r1:8b` |

**개인저장소 마운트:** 없음

---

### 컨테이너 2 — Ollama

| 항목 | 값 |
|------|-----|
| 이미지 | `ghcr.io/{github계정명}/ollama-custom:latest` |
| 컨테이너 포트 | `11434` |
| 컨테이너 명령 | 없음 |

**환경변수:**

| KEY | VALUE |
|-----|-------|
| `OLLAMA_HOST` | `0.0.0.0` |
| `OLLAMA_MODEL` | `deepseek-r1:8b` |

**개인저장소 마운트:** 없음

---

### 목적 노드 / 옵션

| 항목 | 값 |
|------|-----|
| GPU | NVIDIA (RTX 40 또는 RTX 50 시리즈) |
| 최소 CUDA 버전 | `12.0` (RTX 40) / `12.8` (RTX 50 Blackwell) |
| Intro 프록시 사용 | 활성화 |
| L7 Consistent Hash | 활성화 |

---

## Step 5 — 배포 및 동작 확인

워크로드를 배포하면 아래 순서로 진행된다.

```
1. 두 컨테이너 동시 기동
   ├─ OpenWebUI: 서버 시작
   └─ Ollama: ollama serve 실행 → deepseek-r1:8b 모델 자동 다운로드 시작

2. 모델 다운로드 진행 (최초 1회, 약 5~10분 소요)
   └─ 다운로드 중에도 OpenWebUI 화면은 접속 가능

3. 모델 준비 완료 → 채팅 사용 가능
```

Ollama 컨테이너 로그에서 아래 메시지가 나오면 정상:

```
[INFO] Ollama server is ready.
[INFO] Pulling model: deepseek-r1:8b
[INFO] Model pull complete: deepseek-r1:8b
[INFO] All models ready. Keeping server alive...
```

---

## Step 6 — 서비스 접속 및 테스트

1. gcube 워크로드 상세 페이지에서 **서비스 URL** 클릭
2. OpenWebUI 로그인 화면이 나오면 정상
3. **회원가입**으로 계정 생성 (첫 번째 계정이 자동으로 관리자 권한 부여)
4. 로그인 후 채팅 화면 상단 모델 선택란에서 `deepseek-r1:8b` 선택
5. 메시지 입력 → 응답 확인

> 모델 선택란이 비어있으면 Ollama 컨테이너 로그에서 모델 pull 완료 여부를 확인한다.

---

## 모델 변경 방법

gcube 워크로드 환경변수 탭에서 두 군데만 수정하고 재배포하면 된다.

| 컨테이너 | KEY | 변경값 예시 |
|----------|-----|-------------|
| Ollama | `OLLAMA_MODEL` | `qwen2.5:14b` |
| OpenWebUI | `DEFAULT_MODELS` | `qwen2.5:14b` |

이미지 재빌드 없이 모델 전환 가능.

---

## 주의사항

- 이 설정은 dropbox-storage 마운트를 사용하지 않는다.
  컨테이너가 내려가면 대화 기록, 계정 정보, 업로드 파일이 초기화된다.
  **테스트 및 시연 용도로 적합하다.**
- Ollama 포트 11434는 서비스 URL로 노출하지 않는다. (내부 통신 전용)
- OpenWebUI 포트 8080만 서비스 URL로 노출한다.
