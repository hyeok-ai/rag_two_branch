# 학습 노트: 개발 환경 · LLM · RAG

> 필기한 내용을 주제별로 재구성한 정리본. 모델명·API 필드 등은 필기 당시(2026년 9월) 기준이며, 바뀔 수 있으므로 공식 문서로 재확인할 것.

## 목차

1. [개발 환경](#1-개발-환경)
   - 1.1 rsync로 프로젝트를 다른 레포로 옮기기
   - 1.2 macOS 환경변수로 API 키 관리
   - 1.3 uv로 Python 버전과 가상환경 관리
   - 1.4 PyPI와 패키지 배포 구조
2. [Python 예외 처리: except 안에서 또 예외가 날 때](#2-python-예외-처리-except-안에서-또-예외가-날-때)
3. [AI 모델 기본 개념](#3-ai-모델-기본-개념)
   - 3.1 용어 정리
   - 3.2 모델의 공개 수준
   - 3.3 Fully open 모델: OLMo, Pythia, LLM360
   - 3.4 성능과 공개성의 관계
   - 3.5 모델 바깥의 시스템: 하네스/스캐폴딩
4. [LLM의 입력과 출력](#4-llm의-입력과-출력)
   - 4.1 가장 밑바닥: 토큰 → logits
   - 4.2 세 종류의 "파라미터" 구분
   - 4.3 상용 API 비교 (OpenAI · Claude · Gemini · HF)
   - 4.4 Tool calling / Structured output / 멀티모달
   - 4.5 LangChain에서의 temperature와 GPT-5 계열
   - 4.6 문서 보는 법
5. [RAG](#5-rag)
   - 5.1 전체 구조
   - 5.2 문서 로딩: WebBaseLoader + SoupStrainer
   - 5.3 청킹: chunk_size / chunk_overlap
   - 5.4 str vs Document: split_text vs split_documents
   - 5.5 임베딩과 벡터스토어
   - 5.6 similarity_search vs retriever.invoke
   - 5.7 BM25 vs Dense, Hybrid 검색
   - 5.8 Lexical / Sparse / Dense 개념 정리
   - 5.9 프로덕션 검색 구성과 평가

---

## 1. 개발 환경

### 1.1 rsync로 프로젝트를 다른 레포로 옮기기

```bash
rsync -av \
  --exclude='.git' \
  --exclude='.venv' \
  --exclude='.env' \
  rag-12/ my-other-repo/rag-12/
```

| 옵션 | 의미 |
|---|---|
| `-a` | 폴더 구조·파일 속성을 유지하며 재귀 복사 |
| `-v` | 복사되는 파일 목록 출력 |
| `-n` | dry-run. 실제 복사 없이 결과만 확인 |
| `--exclude` | 복사에서 제외할 항목 |

- 먼저 `-avn`으로 확인 → 문제없으면 `n`을 빼고 실행.
- **원본 경로 끝의 `/`에 의미가 있다.** `rag-12/`는 "rag-12 폴더 *내부 내용*"을 목적지에 복사한다.

옮긴 뒤 새 레포에서:

```bash
cd ~/projects/my-other-repo
git status
git add rag-12
git commit -m "Add rag-12 project"
git push
```

가상환경은 옮기지 않고 새 위치에서 다시 만든다. `pyproject.toml`과 `uv.lock` 기준으로 재생성된다.

```bash
cd rag-12
uv sync
```

### 1.2 macOS 환경변수로 API 키 관리

**현재 터미널 세션에서만 사용**

```bash
export OPENAI_API_KEY="실제_API_키"
```

**터미널을 새로 열어도 유지되게 하기** (macOS 기본 셸은 zsh — `echo $SHELL` → `/bin/zsh`)

```bash
nano ~/.zshrc
# 맨 아래에 추가
export OPENAI_API_KEY="실제_API_키"

source ~/.zshrc   # 적용
```

**키 값을 노출하지 않고 등록 여부만 확인**

```bash
if [[ -n "$OPENAI_API_KEY" ]]; then
  echo "OPENAI_API_KEY is set"
else
  echo "OPENAI_API_KEY is not set"
fi
```

**Python에서 읽기**

```python
import os

api_key = os.environ["OPENAI_API_KEY"]   # 없으면 KeyError (필수 값)
api_key = os.getenv("OPENAI_API_KEY")    # 없으면 None (선택 값)
```

**`.env` + python-dotenv를 쓰는 프로젝트**

```dotenv
# .env
OPENAI_API_KEY=${OPENAI_API_KEY}
```

```python
from dotenv import load_dotenv
import os

load_dotenv()
api_key = os.environ["OPENAI_API_KEY"]
```

**필요한 환경변수 목록은 `.env.example`로 공유** (값은 비워 둠)

```dotenv
OPENAI_API_KEY=
LANGSMITH_API_KEY=
DATABASE_URL=
```

**프로젝트 구조와 `.gitignore`**

```text
rag-12/
├── .env            ← 커밋 금지
├── .env.example    ← 커밋
├── .gitignore
├── pyproject.toml
├── uv.lock
└── ...
```

```gitignore
.env
.venv/
```

**전체 작업 순서 요약**

```bash
# 1. API 키 등록
nano ~/.zshrc            # export OPENAI_API_KEY="..."
source ~/.zshrc

# 2. 복사 미리보기 → 실제 복사
rsync -avn --exclude='.git' --exclude='.venv' --exclude='.env' rag-12/ my-other-repo/rag-12/
rsync -av  --exclude='.git' --exclude='.venv' --exclude='.env' rag-12/ my-other-repo/rag-12/

# 3. 환경 생성
cd my-other-repo/rag-12
uv sync

# 4. 커밋
cd ..
git status
git add rag-12
git commit -m "Add rag-12 project"
git push
```

→ 코드·`pyproject.toml`·`uv.lock`은 레포로 옮기고, 가상환경과 API 키는 새 위치에서 구성하는 방식.

### 1.3 uv로 Python 버전과 가상환경 관리

uv는 `pip + venv`의 역할에 더해 **Python 인터프리터 자체도 내려받아 관리**한다. 필요한 버전이 없으면 자동 설치한다.

예: Mac에는 Python 3.14가 있지만 특정 실습 폴더의 `.ipynb`는 3.12로 돌리고 싶을 때.

```bash
cd ~/Desktop/python-study
uv python pin 3.12          # .python-version 파일 생성 (프로젝트 버전 기록)
uv venv                     # 없으면 3.12를 내려받아 .venv 생성
uv pip install ipykernel    # Jupyter 커널용
```

(`uv venv --python 3.12`처럼 버전을 직접 지정해도 된다.)

**설치 위치**

| 대상 | 위치 | 확인 명령 |
|---|---|---|
| uv가 관리하는 Python 본체 | `~/.local/share/uv/python/` | `uv python dir` |
| `python3.12` 등 실행 파일 | `~/.local/bin/` | `uv python dir --bin` |
| 프로젝트 가상환경 | `프로젝트/.venv/` | — |

```text
Mac
├── 기존 Python 3.14 (건드리지 않음)
├── ~/.local/share/uv/python/cpython-3.12.x-.../   ← Python 본체
└── python-study/
    ├── .python-version   (3.12)
    ├── .venv/            ← 3.12 기반 프로젝트 전용 가상환경
    └── lesson01.ipynb
```

"Python 설치 → venv 생성"이라는 개념은 그대로이고, Python 설치 단계까지 uv가 해주는 것.

**활성화/비활성화는 기존 venv와 동일**

```bash
source .venv/bin/activate
python --version   # Python 3.12.x
deactivate
```

단, 현재 폴더에 `.venv`가 있으면 `uv pip install ...`은 activate 없이도 그 환경을 사용한다.

**VS Code에서 노트북 커널 선택**

`.ipynb` 열기 → **Select Kernel** → **Python Environments** → `./.venv/bin/python`

```python
import sys
print(sys.version)      # 3.12.x
print(sys.executable)   # .../python-study/.venv/bin/python
```

**기존 방식과 대응**

| pip + venv | uv |
|---|---|
| Python 별도 설치 | `uv python install 3.12` |
| 버전 관리 도구(pyenv 등) 필요 | `uv python pin 3.12` |
| `python3.12 -m venv .venv` | `uv venv --python 3.12` |
| `pip install numpy` | `uv pip install numpy` |
| `source .venv/bin/activate` | 동일 |

학습 순서: 먼저 `uv venv` + `uv pip`로 pip/venv 대체처럼 써 보고, 익숙해지면 `pyproject.toml` / `uv add` / `uv sync` / `uv.lock` 방식으로 넘어간다.

### 1.4 PyPI와 패키지 배포 구조

```text
개발자 ──업로드──▶ PyPI (pypi.org) ──검색/다운로드──▶ pip / uv ──▶ 내 가상환경(.venv)
```

**어디서 받나**

- pip과 uv 모두 기본 인덱스는 **PyPI** (`https://pypi.org/simple`).
- `/simple/<패키지>/`에서 파일 목록(`.whl`, `.tar.gz`), 다운로드 URL, SHA-256 해시를 받아 설치한다.
- 다른 소스도 가능:
  ```bash
  pip install --index-url https://my-company.example/simple my-package   # 사내 인덱스
  uv add git+https://github.com/foo/bar                                  # Git 저장소
  ```

**누가 올리나** — 해당 패키지의 개발자/maintainer. 원칙적으로 **누구나** 올릴 수 있다.

```toml
# pyproject.toml
[project]
name = "my-awesome-package"
version = "0.1.0"
```

```bash
python -m build     # dist/에 .whl, .tar.gz 생성
twine upload dist/* # PyPI 계정 + API 토큰으로 업로드
```

- 이름은 선착순. 이미 있는 이름, 기존 이름과 헷갈릴 만큼 비슷한 이름은 막힐 수 있다.
- PyPI는 공개 배포용. 사내 전용 패키지는 사설 인덱스를 따로 운영한다.
- 요즘 권장 방식: **Trusted Publishing** — GitHub Actions의 신원을 OIDC로 확인하고 단기(약 15분) 토큰을 발급. 장기 비밀키를 GitHub에 저장할 필요가 없다. 프로젝트가 없어도 Pending Trusted Publisher로 등록해 첫 배포 때 자동 생성 가능.

```text
GitHub repo ─(git tag v1.0.0)─▶ GitHub Actions ─(build)─▶ .whl/.tar.gz ─(OIDC)─▶ PyPI
```

**PyPI의 "검증"은 세 종류로 구분해서 이해**

| 검증 | 확인하는 것 | 확인하지 않는 것 |
|---|---|---|
| 업로드 권한 (토큰, Trusted Publishing) | 이 사람/워크플로가 이 프로젝트에 올릴 권한이 있는가 | 코드가 안전한가, 작성자가 신뢰할 만한가 |
| 다운로드 무결성 (SHA-256) | 받은 파일이 PyPI에 있는 파일과 동일한가 | 파일에 악성 코드가 없는가 |
| 악성 패키지 대응 (신고 → 보안팀 조사 → 삭제) | 사후 대응 | 모든 업로드의 사전 코드 리뷰 |

- 위협 예: **typosquatting** (`requests` 대신 `reqeusts`), dependency confusion, 데이터 탈취. 실제로 악성 패키지가 한동안 올라가 있다가 삭제된 사례가 있다.
- 핵심: `pip install xxx`는 **인터넷의 누군가가 만든 코드를 내 컴퓨터에서 실행 가능하게 가져오는 행위**다.
- 대응: 버전 + lockfile + 해시 고정.
  ```bash
  pip install --require-hashes -r requirements.txt
  ```

**다른 언어도 같은 구조**: 공개 레지스트리 + 패키지 매니저

| 언어 | 레지스트리 | 매니저 |
|---|---|---|
| Python | PyPI | pip / uv |
| JavaScript | npm | npm / pnpm / yarn |
| Rust | crates.io | cargo |
| Java | Maven Central | Maven / Gradle |

---

## 2. Python 예외 처리: except 안에서 또 예외가 날 때

except 블록 안의 작업도 실패할 수 있다는 전제로, **그 작업의 성격에 따라** 처리 방식을 정한다.

**기본형: except 안에 try/except**

```python
try:
    do_something()
except SomeError as e:
    try:
        recover_from_error(e)
    except RecoveryError as recovery_error:
        print("복구 작업까지 실패:", recovery_error)
```

중첩이 깊어지면 함수로 분리한다.

```python
def handle_error(error):
    try:
        send_error_report(error)
    except NetworkError:
        logger.exception("에러 리포트 전송 실패")

try:
    do_something()
except SomeError as e:
    handle_error(e)
```

**except 내부 예외를 다루는 세 가지 방식**

1. **무시해도 되는 보조 작업 → 잡고 로그만 남긴다** (알림, 로그 전송 등)
   ```python
   try:
       process()
   except ProcessError as e:
       try:
           send_notification(e)
       except NotificationError:
           logger.exception("알림 전송 실패")
       # 원래 예외 처리 계속
   ```

2. **심각한 실패 → 위로 올린다**
   ```python
   try:
       process()
   except ProcessError:
       cleanup()
       raise
   ```
   주의: `cleanup()`에서 예외가 나면 최종 예외가 `CleanupError`가 되어 원래 `ProcessError`가 가려질 수 있다 (traceback엔 함께 나오지만 무엇이 본질적 실패인지 헷갈림).

3. **새 예외를 만들되 원인을 연결한다: `raise ... from ...`**
   ```python
   try:
       process()
   except ProcessError as e:
       try:
           recover()
       except RecoveryError as recovery_error:
           raise RuntimeError(f"처리 실패 후 복구까지 실패: {e}") from recovery_error
   ```

**피해야 할 패턴: 예외 삼키기**

```python
# 나쁨 — 에러가 완전히 사라져 디버깅 불가
except Exception:
    try:
        handle_error()
    except Exception:
        pass

# 최소한 로그는 남긴다
except Exception:
    logger.exception("예외 처리 중 추가 예외 발생")
```

**cleanup이 목적이면 `finally`나 context manager**

```python
resource = acquire_resource()
try:
    process(resource)
except ProcessError:
    handle_error()
finally:
    resource.close()

with open("data.txt") as f:   # 파일은 이게 더 좋다
    process(f)
```

**권장 구조**

```python
try:
    main_logic()
except ExpectedError as e:
    logger.exception("main_logic 실패")
    try:
        recovery(e)
    except RecoveryError:
        logger.exception("복구 작업도 실패")
    raise
```

> 핵심: except 안에서는 가능한 단순한 작업만 하고, 그 작업도 실패할 수 있으면 별도 함수/별도 예외 처리로 분리한다. `except Exception: pass`는 특별한 이유가 없으면 피한다.

---

## 3. AI 모델 기본 개념

### 3.1 용어 정리

| 용어 | 언제 쓰나 |
|---|---|
| **생성형 AI** (Generative AI) | 분야 전체를 말할 때 가장 무난한 표현 |
| **LLM** (Large Language Model) | 주로 언어를 다루는 핵심 모델 |
| **멀티모달 모델 / 멀티모달 LLM** | 텍스트 + 이미지 + 음성 등을 함께 다루는 모델 |
| **AI 챗봇 / AI 어시스턴트** | 사람이 실제로 쓰는 서비스 형태 |
| **파운데이션 모델** | 대규모 데이터로 학습되어 여러 작업의 기반이 되는 모델 (기술적 표현) |

- **모델 ≠ 서비스.** ChatGPT는 서비스, 그 안에서 GPT 계열 모델이 동작한다. Claude·Gemini·Qwen·DeepSeek는 브랜드/서비스/모델 패밀리 이름이 섞여 쓰인다.
- ChatGPT, Claude, Gemini, Qwen, DeepSeek를 통틀어 부를 때: **"생성형 AI"** 또는 **"LLM 기반 AI 서비스"**.
- 일상에서는 이미지를 보는 모델도 그냥 "LLM"이라 부르는 경우가 많다. 정확히 구분할 때만 "멀티모달 LLM".

### 3.2 모델의 공개 수준

"오픈소스 모델"이라는 말은 느슨하게 쓰인다. **Open-weight ≠ Open-source**가 핵심.

| 용어 | 뜻 | 가중치 다운로드 | 수정/파인튜닝 | 학습 데이터·코드 공개 |
|---|---|---|---|---|
| 폐쇄형 (Proprietary) | 회사 서버에서만 사용 | ❌ | ❌ | ❌ |
| Open-weight | 학습된 가중치 공개 | ✅ | 대부분 ✅ | 보통 ❌ |
| Open model | 공개 모델의 느슨한 총칭 | 경우에 따라 | 경우에 따라 | 경우에 따라 |
| Source-available | 일부 공개되지만 사용 제한 | 경우에 따라 | 제한적 | 경우에 따라 |
| Open-source AI | 코드·가중치·학습 정보를 충분히 공개, 자유로운 사용/수정/배포 | ✅ | ✅ | ✅ 또는 충분한 정보 |

**OSI Open Source AI Definition 1.0**: 가중치 공개만으로는 부족. 사용·연구·수정·공유의 자유 + 모델 파라미터 + 학습 코드 + 데이터에 대한 충분한 정보(tokenizer, 하이퍼파라미터, 필터링 코드 포함)가 필요하다. 단, 저작권 등으로 원본 데이터를 재배포할 수 없으면, 숙련자가 상당히 동등한 시스템을 만들 수 있을 만큼 출처·선별법·처리법을 제공하는 것도 인정한다.

**비유**

- 가중치 = 공부를 끝낸 AI의 "뇌 상태"
- 학습 코드 = 어떻게 공부시켰는지
- 학습 데이터 = 어떤 교재로 공부했는지
- 모델 구조 = 뇌의 구조

→ Open-weight는 "완성된 뇌를 가져가 돌려도 된다", Open-source AI는 "이 뇌를 어떻게 만들었는지까지 공개한다".

**소프트웨어와의 비교**

```text
소프트웨어:  source code ──compile──▶ binary
LLM:         training data + training code + hyperparameters + seed
             + GPU infra + 수백만 GPU-hours ──▶ weights
```

→ 가중치만 공개 = "컴파일된 바이너리만 공개"에 가깝다.

**주요 모델 분류**

| 회사/기관 | 모델 | 성격 | 라이선스 |
|---|---|---|---|
| OpenAI | ChatGPT의 GPT 계열 | 폐쇄형 | — |
| Anthropic | Claude | 폐쇄형 | — |
| Google | Gemini | 폐쇄형 | — |
| OpenAI | gpt-oss-20b / 120b | Open-weight | Apache 2.0 |
| Alibaba | Qwen3 계열 | Open-weight | Apache 2.0 |
| DeepSeek | DeepSeek-V3 / R1 계열 | Open-weight | MIT |
| Meta | Llama 계열 | Open-weight | Meta 자체 (Llama Community License) |
| Google | Gemma 계열 | Open-weight | Google 자체 (Gemma Terms) |
| Mistral AI | Mistral / Mixtral 등 | 상당수 Open-weight | 모델마다 다름 |
| AI2 | OLMo | Fully open | Apache 2.0 |

모델별 메모:

- **gpt-oss** (2025): OpenAI가 직접 "open-weight"라 부름. 120b는 총 117B 중 토큰당 약 5.1B 활성(MoE), 20b는 총 21B / 활성 3.6B. 추론 구현과 tokenizer 공개. Ollama·vLLM·llama.cpp에서 실행 가능. ChatGPT의 주력 GPT와는 별개.
- **Qwen3**: 0.6B ~ 32B 및 MoE 등 다양한 크기 공개. 오프라인 사내 처리 용도로 많이 검토.
- **DeepSeek**: 서비스도 있지만 모델 자체도 다운로드 가능. R1은 671B 전체 / 약 37B 활성. 구조·RL 과정·추론 코드 등 상당히 많은 기술 정보를 공개하지만, 전체 사전학습 데이터와 실제 학습 코드까지 공개해 재현 가능한 수준은 아님.
- **Llama**: Open-weight 개념을 대중화. 자체 라이선스라 "오픈소스"보다 "Open-weight"가 정확. 데이터는 "공개 + 라이선스 + Meta 서비스 데이터" 수준으로만 설명. 사전학습은 자체 인프라/라이브러리 사용.
- **Gemma vs Gemini**: Gemini = 폐쇄형 주력, Gemma = 다운로드 가능한 Open-weight.

**부르는 법 추천**

1. 파일을 받아 직접 돌릴 수 있다 → **Open-weight 모델**
2. 코드·데이터·학습 방법까지 재현 가능하게 공개 → **Open-source AI / Fully open**
3. 공개 범위를 모르겠다 → **Open model**이라 부르고 라이선스 확인

Qwen·DeepSeek처럼 Apache/MIT 라이선스여도 학습 데이터와 전 과정이 공개되지 않았다면 엄밀히는 Open-weight.

**왜 중요한가: 실행 구조가 다르다**

```text
폐쇄형:      내 PC → 인터넷 → 회사 서버의 모델 → 답변
Open-weight: 내 PC → 내 GPU → 모델 → 답변   (인터넷 없이도 가능)
```

모델 파일을 가지고 있으므로 파인튜닝, 양자화(4bit/8bit), LoRA, 자체 서버 구축이 가능하다. 관련 도구: Ollama, LM Studio, llama.cpp, vLLM.

**공개 정도 스펙트럼**

```text
폐쇄형            Claude / Gemini / 대부분의 GPT         API만 사용
   ▼
Open-weight       Llama / Qwen / DeepSeek / Gemma / gpt-oss   가중치 + 일부 코드
   ▼
Partially Open    가중치 + 학습 recipe + 일부 데이터/정보
   ▼
Fully Open        OLMo / Pythia / LLM360              데이터, 처리 코드, tokenizer, 학습 코드,
                                                       config, checkpoints, logs, eval, post-training
```

```text
공개 정도 →
ChatGPT GPT  ██░░░░░░░░
Claude       ██░░░░░░░░
Gemini       ██░░░░░░░░
Llama        █████░░░░░
gpt-oss      ██████░░░░
DeepSeek     ███████░░░
OLMo         ██████████
```

### 3.3 Fully open 모델: OLMo, Pythia, LLM360

"fully open model", "reproducible LLM", "open training pipeline"이라 불리는 프로젝트들.

**LLM이 만들어지는 흐름** — Fully open 모델은 이 화살표의 상당 부분을 공개한다.

```text
원천 데이터 → 수집·정제 → 중복 제거/필터링 → 토크나이징 → 학습 데이터셋
→ 모델 구조 + 하이퍼파라미터 → 분산 학습 → checkpoint 1, 2, ... → Base 모델
→ SFT / DPO / RL 후학습 → Chat/Instruct 모델
```

**재현에 필요한 것들**

| 필요한 것 | 왜 | OLMo |
|---|---|---|
| architecture | 어떤 Transformer인지 | ✅ |
| tokenizer | 문자열을 어떻게 토큰으로 자르는지 | ✅ |
| training data | 무엇을 보고 배웠는지 | ✅ |
| 전처리/중복 제거 | 필터링·정제 방법 | ✅ |
| 데이터 혼합 비율 | 각 데이터가 얼마나 들어갔는지 | ✅ |
| 데이터 순서 | 어떤 순서로 학습했는지 | 상당 부분 |
| training code | 실제 PyTorch 학습 코드 | ✅ |
| 하이퍼파라미터·optimizer 설정 | LR, batch size, AdamW 설정 | ✅ |
| 중간 checkpoints | 학습 중 모델 상태 | ✅ |
| optimizer state | 학습을 이어가기 위한 상태 | 경우에 따라 |
| training logs | loss 변화 | ✅ |
| evaluation / post-training 코드 | 성능 측정, SFT/RL | ✅ |

**OLMo (AI2)**

- 목표: 완성 모델만이 아니라 **만들어지는 전 과정**("complete model flow")을 공개.
- 공개 범위: 학습 데이터(Dolma 3), 데이터 가공 코드, 가중치, 학습 코드, 로그/메트릭, 중간 체크포인트, 추론·평가·파인튜닝 코드.
- 도구: OlmoCore(학습 프레임워크), 데이터 처리 도구, Open Instruct(후학습), OLMES(평가).
- 규모 감: OLMo 3 7B 1단계 사전학습 = 5.93조 토큰, H100 512장. 32B는 1단계에 H100 1,024장.
- 비유: "우리가 이 모델을 만든 실험실 노트와 재료와 장비 사용법을 같이 공개합니다."

**Pythia (EleutherAI)**

- "학습되면서 모델이 어떻게 변하는가" 연구용. 모델·데이터·코드 모두 공개.
- 각 모델당 **154개 체크포인트**, pre-tokenized 데이터와 dataloader 재구성 스크립트 제공.
- 연구 질문 예: 언제부터 사실을 암기하는가? 문법 능력은 언제 생기는가? 특정 뉴런 기능은 언제 형성되는가?

**LLM360 Amber**

- 전체 중간 체크포인트, 학습 데이터셋, 데이터 준비 코드(RedPajama, RefinedWeb, StarCoderData 처리 등), 학습 코드, config 공개.

OSI 검증 과정에서 OLMo, Pythia, LLM360 Amber·CrystalCoder, Google T5가 Open Source AI 조건 충족 사례로 제시됨.

| 모델 | 특징 |
|---|---|
| OLMo | 가장 체계적인 fully open LLM 생태계 |
| Pythia | 학습 과정 연구·재현성에 특화 |
| LLM360 Amber | 데이터→학습→체크포인트 공개 연구 프로젝트 |

**"완전히 똑같이 재현"은 별개 문제**

- 코드·데이터·설정을 다 가져도 **비트 단위로 동일한 가중치**가 나온다는 보장은 없다: GPU 비결정성, CUDA/cuDNN 버전, GPU 종류, 분산학습 연산 순서, floating-point rounding.
- ML에서 reproducible = "같은 방법으로 돌리면 **비슷한 loss curve와 비슷한 성능**을 얻을 수 있다".
- 현실적 장벽은 비용. `git clone olmo && python train.py`로 RTX 4090 한 장에서 7B를 만들 수는 없다. 개인은 Pythia/OLMo의 작은 모델을 축소 데이터로 학습해 보는 것이 현실적.

### 3.4 성능과 공개성의 관계

**Fully open 모델은 쓸 만한가?**

- **Pythia / Amber**: 지금 기준으로는 LLM 과학 실험 장비. 일상용 아님.
- **OLMo 3**: 연구용이면서 실사용 가능. 32B Think는 꽤 강함.
- **Llama / Qwen / DeepSeek / gpt-oss**: 최고 성능 경쟁에 더 초점.
- **최신 ChatGPT / Claude / Gemini**: reasoning, coding, agent, 멀티모달, 제품 완성도 포함 시 최상단.

AI2 동일 평가 기준 OLMo 3 32B Think vs Qwen 3 32B:

| 벤치마크 | OLMo 3 32B Think | Qwen 3 32B |
|---|---|---|
| MATH | 96.1 | 95.4 |
| AIME 2025 | 72.5 | 70.9 |
| HumanEval+ | 91.4 | 91.2 |
| LiveCodeBench v3 | 83.5 | 90.2 |
| GPQA | 58.1 | 67.3 |
| MMLU | 85.4 | 88.8 |
| BBH | 89.8 | 90.6 |

→ 수학은 이기기도 하고, 코딩·과학 추론·일반 지식은 다소 뒤짐. "완전 공개라서 장난감"은 아니다.

주의: 상용 서비스(ChatGPT 등)는 더 큰 모델, 장문 컨텍스트, 멀티모달, 도구, 검색, 에이전트 최적화를 **하나의 시스템**으로 묶은 것이라 단일 32B 체크포인트와 직접 비교하면 불공평하다.

**유명 모델은 왜 전부 공개하지 않나 — 공개의 층위**

1. **API만 공개** (GPT, Claude, Gemini): 가중치·데이터·학습 코드·체크포인트 ❌, 논문/시스템 카드 ✅. 데이터 출처는 유형 수준(공개 웹, 제3자, 라벨러, 옵트인 사용자, 내부 생성 데이터 등)으로만 설명.
2. **Open-weight** (Llama, DeepSeek, gpt-oss): "완성된 뇌는 주지만, 어떻게 태어나게 했는지는 전부 공개하지 않는다."
   - OLMo = 실험 자체를 공개
   - DeepSeek = 논문 + 완성 모델 + 상당한 구현 정보 공개

> 역설: **"가장 성능 좋은 공개 모델"과 "가장 공개된 모델"은 같은 말이 아니다.** DeepSeek는 성능↔공개성 절충이 공격적이고, OLMo는 과학적 재현성 쪽으로 밀어붙였다.

### 3.5 모델 바깥의 시스템: 하네스/스캐폴딩

모델을 감싸서 실제로 쓸 수 있게 만드는 부분의 이름 (공식 용어는 없음):

- **스캐폴딩(scaffolding)**, **하네스(harness)** — 가장 흔함. 에이전트 쪽에선 "agent harness"
- **오케스트레이션 레이어**, **래퍼(wrapper)**, **애플리케이션 레이어**
- 학계: **컴파운드 AI 시스템(compound AI system)** — 여러 구성요소를 합친 전체

바깥 부분의 구성 요소:

- **챗 템플릿**: 대화를 "사용자:/어시스턴트:" 특수 토큰 형식으로 변환
- **시스템 프롬프트**: 역할과 규칙
- **대화 기록·컨텍스트 관리**: 모델은 기억이 없어 매 턴마다 전체 대화를 다시 읽는다
- **도구 호출(tool use)**: 웹 검색, 코드 실행 등
- **RAG**: 외부 문서 검색 후 주입
- **메모리**: 대화 간 기억
- **안전 필터·분류기**
- **샘플링/디코딩 설정** (temperature 등)
- **서빙 인프라**, **UI**

주의: **"대화하듯 답하는 성격"은 바깥이 아니라 모델 안에 있다.** 사전학습만 한 LLM은 문장을 이어 쓰는 모델이고, 인스트럭션 튜닝·RLHF 같은 **사후 학습(post-training)** 을 거치며 가중치 자체가 어시스턴트처럼 바뀐다.

> 챗봇 = 사후 학습된 모델 + 그걸 감싸는 하네스

---

## 4. LLM의 입력과 출력

### 4.1 가장 밑바닥: 토큰 → logits

"LLM은 문장을 입력받아 문장을 출력한다"는 입문 수준에선 충분하지만, 정확히는:

> **LLM은 지금까지 주어진 토큰 시퀀스를 조건으로 다음 토큰의 확률분포를 계산하는 신경망이다.**

```text
"대한민국의 수도는"
   ↓ tokenizer
[24871, 9134, 392, 12588]          ← input_ids (token ID)
   ↓ Transformer
logits = [V개의 실수]               ← V = vocabulary size (예: 100,000)
   ↓ softmax
다음 토큰 확률분포
```

$$
P(\text{token}_{next} = i) = \frac{e^{z_i}}{\sum_j e^{z_j}}
$$

| 다음 토큰 | 확률 |
|---|---|
| 서울 | 0.91 |
| 부산 | 0.018 |
| 대한민국 | 0.013 |
| 세종 | 0.009 |
| 입니다 | 0.003 |

"서울"을 고르면 이어 붙여 다시 모델을 실행 → "입니다" → "." … 를 반복(**autoregressive generation**).

**Transformer 수준의 입출력**

| 구분 | 이름 | shape | 의미 |
|---|---|---|---|
| 입력 | `input_ids` | `[batch, seq]` | 토큰 ID |
| 입력 | `attention_mask` | `[batch, seq]` | 어느 토큰을 실제 입력으로 볼지 |
| 입력 | `position_ids` | `[batch, seq]` | 위치 정보 (구현에 따라) |
| 입력 | `past_key_values` | Tensor 묶음 | KV cache: 이전 계산 재사용 |
| 출력 | `logits` | `[batch, seq, vocab]` | 각 위치의 다음 토큰 후보 점수 (softmax 전) |
| 출력 | `past_key_values` | Tensor 묶음 | 다음 생성 단계에서 재사용 |
| 선택 출력 | `hidden_states` | 여러 Tensor | 각 layer 내부 표현 |
| 선택 출력 | `attentions` | 여러 Tensor | attention weight |

```python
input_ids.shape               # (1, 5)
logits.shape                  # (1, 5, 128000)
next_token_logits = logits[:, -1, :]   # (1, 128000) — 생성에선 마지막 위치만 중요
```

**전체 생성 흐름 (API 서버 내부)**

```text
텍스트 → Tokenizer → token IDs → Transformer → logits → sampling/decoding
→ token IDs → Tokenizer decode → 텍스트
```

API를 쓸 때 `input_ids`가 안 보이는 이유: 서버가 이 과정을 전부 대신 해주기 때문.

**멀티모달 입력**

```text
이미지 → vision encoder → image embeddings/tokens ─┐
                                                   ├─▶ Transformer
텍스트 → text tokens ──────────────────────────────┘
```

통합 토큰 시퀀스 방식일 수도, cross-attention 방식일 수도 있다(모델마다 다름). 오디오·영상도 각각 임베딩으로 변환되어 컨텍스트에 들어간다. 출력도 텍스트·오디오·이미지·tool call·JSON 등이 가능.

→ 현대 생성형 AI를 추상화하면:

$$
F(\text{multimodal context},\ \text{instructions},\ \text{tools},\ \text{generation config}) \rightarrow \text{output events}
$$

### 4.2 세 종류의 "파라미터" 구분

같은 "파라미터"라는 말이 서로 다른 계층을 가리켜서 헷갈린다.

| 용어 | 예 | 의미 |
|---|---|---|
| **모델 파라미터** | weights ($W_Q, W_K, W_V, W_O$ 등) | 학습된 신경망 숫자. "몇 B 모델"의 B |
| **모델 입력** | `input_ids`, `attention_mask` | 실제 추론에 들어가는 데이터 |
| **생성(디코딩) 파라미터** | `temperature`, `top_p`, `top_k`, `max_new_tokens`, `stop`, `seed`, penalty, `num_beams`, `do_sample` | logits가 나온 **뒤** 토큰 선택 방식을 제어 |

temperature는 Transformer의 입력이 아니다. logits 후처리에서 쓰인다:

$$
\text{softmax}(\text{logits} / T)
$$

- $T$ 작음 → 최고 후보가 압도적 (결정적)
- $T$ 큼 → 분포가 평평해짐 (다양함)

```text
model.forward(input_ids, attention_mask, past_key_values, ...)
        ↓
      logits
        ↓
generation algorithm  ← temperature / top_p / top_k / repetition penalty ...
        ↓
   다음 token 선택
```

상용 API는 이 둘을 **한 요청 객체에 섞어서** 보여주기 때문에 헷갈린다.

**Structured output도 결국 토큰 생성이다.** `{`, `"`, `name`, … 을 차례로 생성하되, decoding 단계에서 JSON schema에 맞지 않는 토큰을 막아 유효한 JSON을 만든다.

### 4.3 상용 API 비교 (OpenAI · Claude · Gemini · HF)

앞의 셋은 **서비스 API I/O**, Llama/HF는 **모델 자체 I/O**까지 볼 수 있다.

**입력 파라미터**

| 목적 | OpenAI Responses | Claude Messages | Gemini generateContent | Llama / HF `forward()` |
|---|---|---|---|---|
| 모델 선택 | `model` (필수) | `model` (필수) | `model` (필수, path) | 모델 객체를 미리 로드 |
| 실제 입력 | `input`: string \| array | `messages` (필수) | `contents[]` (필수) | `input_ids` |
| 시스템 지시 | `instructions` | `system` | `systemInstruction` | chat template 단계 |
| 이미지/파일 | `input[].content[]` | `messages[].content[]` | `contents[].parts[]` | 일반 Llama는 text token |
| 출력 길이 | `max_output_tokens` | `max_tokens` (필수) | `generationConfig.maxOutputTokens` | `generate()`의 `max_new_tokens` |
| 도구 정의 | `tools` | `tools` | `tools` | 없음 |
| 도구 선택 | `tool_choice` | `tool_choice` | `toolConfig` | 없음 |
| 추론 | `reasoning` | `thinking`, `output_config.effort` | `generationConfig.thinkingConfig` | 모델/구현 의존 |
| 구조화 출력 | `text.format` | `output_config.format` | `generationConfig.responseFormat` | 직접 처리 |
| temperature | 모델에 따라 | 최신 모델에서 deprecated | `temperature` | `generate()`에서 |
| top_p / top_k | 모델에 따라 | 최신 모델에서 deprecated | `topP` / `topK` | `generate()`에서 |
| stop | API/모델별 | `stop_sequences` | `stopSequences` | stopping criteria |
| seed | 모델/API별 | — | `seed` | 설정 가능 |
| streaming | `stream` | `stream` | `streamGenerateContent` | 직접 loop |
| 이전 대화 | `previous_response_id`, `conversation` | messages를 다시 전달 (stateless) | `contents[]`에 history | 직접 구성 |
| 안전 설정 | 플랫폼 내부 | 플랫폼 정책 | `safetySettings[]` | 직접 구현 |
| 캐시 | prompt cache 관련 | cache control | `cachedContent` | KV cache 직접 관리 |
| KV cache / attention mask / position / embedding 직접 입력 / labels | ❌ (서버 내부) | ❌ | ❌ | ✅ |

**"실제 내용"의 이름만 다르다**

```text
OpenAI   input[].content[]
Claude   messages[].content[]
Gemini   contents[].parts[]
```

예시 요청:

```json
// OpenAI Responses
{
  "model": "...",
  "instructions": "너는 프로그래밍 강사다.",
  "input": [
    {"role": "user", "content": [
      {"type": "input_text", "text": "이 이미지가 무엇인지 설명해줘."},
      {"type": "input_image", "image_url": "..."}
    ]}
  ]
}
```

```json
// Claude Messages
{
  "model": "...",
  "max_tokens": 2000,
  "system": "너는 프로그래밍 강사다.",
  "messages": [{"role": "user", "content": "Python generator를 설명해줘."}]
}
```

```json
// Gemini generateContent
{
  "contents": [{"role": "user", "parts": [{"text": "양자역학을 설명해줘."}]}],
  "systemInstruction": {"parts": [{"text": "너는 과학 선생님이다."}]},
  "generationConfig": {"temperature": 0.7, "maxOutputTokens": 1000}
}
```

**Reasoning / Thinking**

| 회사 | 파라미터 | 비고 |
|---|---|---|
| OpenAI | `reasoning.effort` | usage에 `reasoning_tokens` |
| Claude | `thinking` (예: `{"type": "adaptive"}`), `output_config.effort` | usage에 thinking token 통계 |
| Gemini | `thinkingConfig`: `includeThoughts`, `thinkingBudget`, `thinkingLevel`(MINIMAL/LOW/MEDIUM/HIGH) | 최신 모델은 `thinkingLevel` 권장 |
| Llama | 공통 API 없음 | 모델/프레임워크별 |

**API 설계의 흐름 변화**: `temperature / top_p / top_k` 중심 → reasoning 모델에서는 `effort = low / medium / high / ...` 중심.

- Claude: Opus 4.7 이후 일부 모델에서 temperature/top_p/top_k deprecated. 최신 SDK에서 제거, 미지원 모델에 non-default 값을 넣으면 오류 가능.
  ```python
  client.messages.create(
      model="...",
      max_tokens=16000,
      output_config={"effort": "high"},
      messages=[{"role": "user", "content": "..."}],
  )
  ```
- Gemini: 구조화 출력은 `responseSchema`/`responseJsonSchema` → `responseFormat` 방향으로 이동 중.

**출력 구조**

| 개념 | OpenAI | Claude | Gemini | Llama/HF |
|---|---|---|---|---|
| 요청 ID | `id` | `id` | `responseId` | — |
| 모델 | `model` | `model` | `modelVersion` | `model.config` |
| 생성 내용 | `output[]` | `content[]` | `candidates[]` | `logits` |
| 텍스트 | `output_text` (편의 속성) | `content[type=text].text` | `candidates[].content.parts[].text` | `tokenizer.decode` 필요 |
| tool call | `output[type=function_call]` | `content[type=tool_use]` | function call part | 직접 구현 |
| 종료 이유 | `status` / incomplete details | `stop_reason` | `finishReason` | EOS / stopping criteria |
| 토큰 사용량 | `usage.input_tokens/output_tokens` | `usage.input_tokens/output_tokens` | `usageMetadata.promptTokenCount/candidatesTokenCount/thoughtsTokenCount` | 직접 계산 |
| logprob | 일부 | 제한적 | `logprobsResult` | logits 직접 |
| safety | 플랫폼 | stop/safety | `safetyRatings` | — |
| hidden state / attention / KV cache | ❌ | ❌ | ❌ | ✅ |

```text
OpenAI Response            Claude Message               Gemini Response
├─ id                      ├─ id                        ├─ candidates[]
├─ model                   ├─ type: "message"           │  ├─ content
├─ status                  ├─ role: "assistant"         │  ├─ finishReason
├─ output[]                ├─ model                     │  ├─ safetyRatings
│  ├─ message              ├─ content[]                 │  ├─ citationMetadata
│  │  └─ output_text       │  ├─ text                   │  └─ groundingMetadata
│  └─ function_call        │  └─ tool_use               ├─ promptFeedback
└─ usage                   ├─ stop_reason               ├─ usageMetadata
                           └─ usage                     ├─ modelVersion
                                                        └─ responseId
```

**Llama / Hugging Face: 모델 수준 I/O**

```python
inputs = tokenizer("대한민국의 수도는", return_tensors="pt")
# {"input_ids": tensor(...), "attention_mask": tensor(...)}

outputs = model(**inputs)
logits = outputs.logits          # (batch, seq, vocab_size)

# 텍스트 생성은 generation layer로
model.generate(**inputs, max_new_tokens=100, do_sample=True, temperature=0.7, top_p=0.9)
```

`LlamaForCausalLM.forward()` 주요 인자:

| 인자 | shape | 역할 |
|---|---|---|
| `input_ids` | `[B, S]` | 토큰 ID |
| `attention_mask` | `[B, S]` | attention 대상 |
| `position_ids` | `[B, S]` | 위치 |
| `past_key_values` | Cache | 이전 K/V 재사용 |
| `inputs_embeds` | `[B, S, H]` | `input_ids` 대신 임베딩 직접 입력 |
| `labels` | — | 학습 시 LM loss 계산 |
| `use_cache` | bool | KV cache 반환 여부 |
| `logits_to_keep` | int/Tensor | 어느 위치의 logits를 계산할지 |

출력 `CausalLMOutputWithPast`: `loss`, `logits`, `past_key_values`, `hidden_states`, `attentions`.

**요약 그림**

```text
[OpenAI / Claude / Gemini]
{사용자 입력, 시스템 지시, history, 이미지/파일, tools,
 generation config, reasoning config, output schema}
        ↓ Provider
{text, tool call, structured output, usage, finish reason, metadata}

[Llama / raw model]
input_ids, attention_mask, position_ids, KV cache
        ↓ Transformer
logits, KV cache, hidden states, attention
        ↓ temperature / top-p / top-k → sampling → token IDs → decode → 문장
```

### 4.4 Tool calling / Structured output / 멀티모달

**Tool calling**: 모델 출력이 답변 문장이 아니라 함수 호출일 수 있다.

```json
{
  "name": "get_weather",
  "description": "특정 도시의 날씨를 가져온다.",
  "parameters": {
    "type": "object",
    "properties": {"city": {"type": "string"}},
    "required": ["city"]
  }
}
```

```text
User "서울 날씨?"
  ↓ LLM
tool call: get_weather(city="서울")      ← 모델은 호출을 "요청"할 뿐 실행하지 않는다
  ↓ 프로그램이 함수 실행
tool result: {"temp": 23, "condition": "맑음"}
  ↓ LLM에 다시 입력
"현재 서울은 23°C이고 맑습니다."
```

→ AI Agent가 동작하는 핵심 원리. RAG, Agent, MCP도 결국 "LLM 컨텍스트에 무엇을 추가 입력하고, 출력을 누가 어떻게 해석/실행하느냐"의 문제.

**전체 시스템 한 장**

```text
입력: Text / Image / Audio / File / Tool result / History
  ↓
Prompt/Chat layer (system, developer, user, assistant history, tool definitions)
  ↓
Tokenizer / Encoders (text→tokens, image/audio→embeddings)
  ↓
Transformer (token/embedding → logits)
  ↓
Generation algorithm (temperature, top-p/k, stop, max tokens)
  ↓ 다음 token 선택 → 반복
최종 token sequence
  ↓
Output: Text / JSON / Tool call / Image·Audio / Citations / Usage·finish reason
```

> 세 층을 구분: **LLM 자체 → logits** / **generation layer → token** / **API → text·tool call·JSON 객체**

### 4.5 LangChain에서의 temperature와 GPT-5 계열

`temperature` 파라미터가 LangChain에서 사라진 게 아니라, **GPT-5 계열에서 모델별로 사용이 제한**된 것.

```python
# 기존 모델: 그대로 가능
llm = ChatOpenAI(model="gpt-4.1-mini", temperature=0)
```

LangChain `ChatOpenAI`는 GPT-5 계열(gpt-5-chat 제외)에 대해 **temperature=1 또는 미설정만 허용**하도록 검사한다.

| 목적 | GPT-4.1 계열 | GPT-5.6 계열 |
|---|---|---|
| 랜덤성 | `temperature` | 일반적으로 직접 조절 안 함 |
| 추론량 | 거의 없음 | `reasoning_effort` (none/low/medium/high/xhigh/max) |
| 답변 길이/상세도 | 프롬프트 | `verbosity` |
| 결정적 출력 | `temperature=0` | 프롬프트 + 구조화 출력 |

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="gpt-5.6-luna",      # 공식 ID. (gpt-6-luna 아님)
    reasoning_effort="low",
    verbosity="low",
)
structured_llm = llm.with_structured_output(MySchema)
```

- `temperature=0`을 썼던 목적(같은 질문 → 같은 결과; RAG, 추출, 분류, SQL/JSON 생성)은 **출력 스키마와 프롬프트로 강하게 제한**하는 방식으로 대체.
- `model_name=`은 동작하는 별칭이지만 최신 예제는 `model=`을 쓴다.

### 4.6 문서 보는 법

| 알고 싶은 것 | 볼 문서 |
|---|---|
| API가 어떤 필드를 받는가 | API Reference |
| 모델이 text/image/audio 중 무엇을 지원하는가, context window, max output | Models / Model Card |
| 모델 내부에서 Tensor를 뭘 받고 내는가 | 모델 구현 / Hugging Face docs |

| 계열 | API 파라미터 | 모델별 기능 |
|---|---|---|
| OpenAI | Responses API Reference | OpenAI Models (모델 클릭 → 입출력 modality, context, reasoning, tools) |
| Anthropic | Messages API Reference (+ Tool Use 문서) | Claude Models |
| Google | Gemini API Reference — `generateContent` (Interactions API는 agentic 용도로 권장) | Gemini Models |
| Mistral | Chat API Reference, Chat Completion Guide | Mistral Models |
| Cohere | v2 Chat API Reference | Cohere Models |
| Llama/Qwen/DeepSeek | HF Transformers → Models → `LlamaForCausalLM` → `forward` / Model Outputs 문서 | 각 model card |

**3계층으로 보기**

```text
① Model Card        이 모델이 뭘 할 수 있지? (modality, context)
② API Reference     어떤 JSON을 보내고 받지? (model, messages, tools, usage ...)
③ Model 구현        Transformer에 뭐가 들어가지? (input_ids → logits)
```

①② = 상용 API 사용자에게, ③ = 오픈웨이트 모델을 직접 돌리거나 원리를 공부할 때 중요. 공부 순서는 **① API Request/Response → ② Generation 설정 → ③ Raw Transformer I/O**.

---

## 5. RAG

### 5.1 전체 구조

```text
[Indexing]
파일 → str → Document(+metadata) → Text Splitter → chunks → Embedding Model → vectors → Vector Store(FAISS)

[Retrieval + Generation]
질문 → Embedding Model → query vector → similarity search → 관련 Document → Prompt에 삽입 → LLM → 답변
```

| 구성요소 | 역할 |
|---|---|
| Text Splitter | 문서를 적당히 자른다 (벡터를 만들지 않음) |
| Embedding | 텍스트를 숫자 벡터로 바꾼다 |
| FAISS 등 | 벡터를 저장하고 가까운 벡터를 찾는다 (임베딩을 만들지 않음) |
| Retriever | 문서를 찾는다 |
| LLM | 찾은 문서를 보고 답을 만든다 |

### 5.2 문서 로딩: WebBaseLoader + SoupStrainer

웹페이지 HTML 전체가 아니라 **원하는 `<div>`만 골라서 파싱**하는 설정.

```python
import bs4
from langchain_community.document_loaders import WebBaseLoader

loader = WebBaseLoader(
    web_paths=("https://n.news.naver.com/...",),
    bs_kwargs=dict(
        parse_only=bs4.SoupStrainer(
            "div",
            attrs={"class": ["newsct_article _article_body", "media_end_head_title"]},
        )
    ),
)
docs = loader.load()
print(docs[0].page_content)   # 실제로 어떤 텍스트가 들어왔는지 확인
```

- `SoupStrainer`: BeautifulSoup에게 "이 조건에 맞는 HTML만 관심 가져"라고 지정하는 필터.
- `"div"`: div 태그만, `attrs={"class": [...]}`: 그 중 해당 클래스만.
  - `media_end_head_title` → 뉴스 제목
  - `newsct_article _article_body` → 뉴스 본문. **클래스 하나가 아니라** `newsct_article`, `_article_body` 두 클래스가 붙은 상태.
- `bs_kwargs`는 내부적으로 `BeautifulSoup(html, "html.parser", parse_only=...)`에 전달된다.

```text
네이버 뉴스 페이지 → HTML 전체 다운로드 → SoupStrainer로 제목+본문만 파싱 → LangChain Document
```

안 쓰면 메뉴, 버튼, 광고, 댓글 영역 텍스트까지 Document에 섞인다.

### 5.3 청킹: chunk_size / chunk_overlap

```python
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)
```

- 기본 설정(`length_function=len`)에서는 **토큰이 아니라 문자 수** 기준. 500 = 대략 500자.
- overlap = 앞 청크 마지막 약 50자를 다음 청크에도 넣음 → 경계에서 의미가 끊기는 문제 완화.

```text
청크 1: RAG는 외부 문서에서 관련 정보를 검색한 뒤 ...
청크 2: 외부 문서에서 관련 정보를 검색한 뒤 그 정보를 LLM의 프롬프트에 넣어...
        ↑ overlap이 없으면 "그 정보"가 무엇인지 애매해진다
```

| 기준 | chunk가 작을 때 | chunk가 클 때 |
|---|---|---|
| 검색 정확도 | 특정 문장을 정확히 찾기 쉬움 | 관련 없는 내용도 딸려옴 |
| 문맥 보존 | 끊기기 쉬움 | 잘 보존 |
| 결과 정보량 | 적음 | 많음 |

| 문서 유형 | 시작값 예시 |
|---|---|
| FAQ / 규정 / 매뉴얼 (한 문단 = 한 정보) | size 300~500, overlap 30~100 |
| 논문 / 보고서 (문맥 중요) | size 800~1500, overlap 100~300 |

- overlap은 chunk의 **10~20%** 에서 시작하는 경우가 많다. 500/50 = 10%로 전형적인 예제값 (노트북에 근거는 없고 무난한 시작값).
- 기준은 절대값이 아니라 **한 chunk가 하나의 의미 단위를 충분히 담는가**. 질문의 답이 한 청크 안에 들어갈 정도여야 한다.
- 실무: 500/50, 1000/100, 1500/200 등으로 바꿔 가며 실제 질문에 원하는 청크가 검색되는지 확인해 결정.

> **chunk_size는 LLM이 읽을 수 있는 최대 길이가 아니라, Retriever가 "검색하기 좋은 의미 단위"가 되도록 정하는 값이다.**

### 5.4 str vs Document: split_text vs split_documents

흔한 실수: `text`가 `str`인데 `split_documents()`에 넣음. `split_documents()`는 `Iterable[Document]`를 받는다.

| 문자열 기반 | Document 기반 |
|---|---|
| `split_text(str)` | `split_documents(list[Document])` |
| → `list[str]` | → `list[Document]` |
| `FAISS.from_texts(list[str])` | `FAISS.from_documents(list[Document])` |

```python
# LangChain Document
Document(page_content="안녕하세요...", metadata={"source": "...", "page": 1})
```

**방법 1: 문자열 그대로**

```python
import torch
from langchain_openai import OpenAIEmbeddings
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_community.vectorstores import FAISS
from langchain_text_splitters import RecursiveCharacterTextSplitter

# 1. 텍스트 읽기 ([:500]은 테스트용 — 전체를 쓰려면 f.read())
with open("data/chain-of-density.txt", encoding="utf-8") as f:
    text = f.read()[:500]

# 2. 임베딩 선택 (OpenAI 실패 시 로컬 모델로 대체)
try:
    embeddings = OpenAIEmbeddings()
    embeddings.embed_query("test")          # 실제 API 호출 가능 여부 확인
    print("OpenAI 임베딩 사용")
except Exception as e:
    print(f"OpenAI 임베딩 사용 불가: {e}")
    device = "mps" if torch.backends.mps.is_available() else "cpu"
    embeddings = HuggingFaceEmbeddings(
        model_name="BAAI/bge-m3",
        model_kwargs={"device": device},
        encode_kwargs={"normalize_embeddings": True},
    )
    print(f"HuggingFace 임베딩으로 대체 ({device})")

# 3. 문자열 splitting
text_splitter = RecursiveCharacterTextSplitter(chunk_size=100, chunk_overlap=10)
splits = text_splitter.split_text(text)
print(f"총 chunk 개수: {len(splits)}")

# 4. FAISS
vectorstore = FAISS.from_texts(texts=splits, embedding=embeddings)

# 5. 검색
for i, doc in enumerate(vectorstore.similarity_search("이 문서의 주요 내용은 무엇인가?", k=3), 1):
    print(f"\n--- 결과 {i} ---\n{doc.page_content}")
```

**방법 2: Document로 감싸기 (권장 — 나중에 metadata가 중요해짐)**

```python
from langchain_core.documents import Document

document = Document(page_content=text, metadata={"source": "data/chain-of-density.txt"})
splits = text_splitter.split_documents([document])
vectorstore = FAISS.from_documents(documents=splits, embedding=embeddings)
```

**패키지 설치**

```bash
pip install -U langchain-community langchain-openai langchain-huggingface \
    langchain-text-splitters sentence-transformers faiss-cpu
```

- `RecursiveCharacterTextSplitter` → `langchain_text_splitters` (별도 패키지)
- `OpenAIEmbeddings` → `langchain_openai` (`OPENAI_API_KEY` 필요)
- `HuggingFaceEmbeddings` → `langchain_huggingface` (`sentence-transformers` 필요)

### 5.5 임베딩과 벡터스토어

```text
[저장]  Document → page_content → embeddings.embed_documents() → [0.013, -0.382, ...] → FAISS
[검색]  질문 → embeddings.embed_query() → [0.018, -0.401, ...] → FAISS에서 가까운 벡터 → Document
```

- LangChain 임베딩 인터페이스의 핵심 메서드: `embed_documents(List[str])`, `embed_query(str)`.
- **BAAI/bge-m3**: 한국어가 포함된 RAG에 합리적인 multilingual 모델. 가중치 약 2.27GB라 첫 실행 시 다운로드 시간/디스크 필요. Mac은 `mps` 사용 가능하나 `torch.backends.mps.is_available()`로 확인.

> ⚠️ **인덱스를 만든 임베딩 모델 = 검색 쿼리에 쓰는 임베딩 모델**이어야 한다.
> OpenAI 임베딩으로 만든 FAISS 인덱스를 BGE-M3로 검색하면 안 된다 — 서로 다른 벡터 공간(좌표계)이다. 모델을 바꾸면 인덱스부터 다시 만든다. fallback 로직을 쓸 때 특히 주의.

### 5.6 similarity_search vs retriever.invoke

```python
# Vector Store에 직접 요청
vectorstore.similarity_search("구글")

# Retriever 인터페이스를 통해 요청
retriever = vectorstore.as_retriever()
retriever.invoke("삼성전자가 자체 개발한 AI 의 이름은?")
```

```text
similarity_search:  질문 → 임베딩 → FAISS 유사 벡터 검색 → Document
retriever.invoke:   질문 → Retriever → FAISS → Document
```

- `as_retriever()`는 새 DB를 만드는 게 아니라 **기존 vectorstore에 검색기 인터페이스를 씌운 것**.
- 기본 설정이면 결과는 거의 같다. *(실험 결과: 실제 반환 결과가 같았음.)*
- 둘 다 **LLM이 답을 만들지 않는다.** 반환값은 `[Document(...), ...]`.

**Retriever를 쓰는 이유**

1. RAG 체인에 연결하기 쉬움
   ```python
   chain = (
       {"context": retriever, "question": RunnablePassthrough()}
       | prompt
       | llm
   )
   ```
2. 검색 옵션 설정
   ```python
   vectorstore.as_retriever(search_kwargs={"k": 3})   # 3개만
   vectorstore.as_retriever(search_type="mmr")        # MMR (Maximal Marginal Relevance)
   ```

| 코드 | 의미 |
|---|---|
| `vectorstore.similarity_search(query)` | Vector DB에 직접 유사도 검색 |
| `vectorstore.as_retriever()` | Vector Store를 Retriever로 변환 |
| `retriever.invoke(query)` | Retriever에게 관련 문서 요청 |

비유: vectorstore = 도서관 책장. `similarity_search`는 직접 책장에서 찾기, `retriever.invoke`는 사서에게 부탁하기.

### 5.7 BM25 vs Dense, Hybrid 검색

엄밀히는 "BM25 vs FAISS"가 아니라 **lexical retrieval(BM25) vs embedding 기반 dense retrieval**. FAISS는 벡터 검색 라이브러리일 뿐.

**예제 데이터가 보여주는 것** (다의어 "배")

```text
난 오늘 많이 먹어서 배가 정말 부르다        → 배 = stomach
떠나는 저 배가 오늘 마지막 배인가요?         → 배 = ship
내가 제일 좋아하는 과일들은 배, 사과...      → 배 = pear
```

- query `"배"` → BM25는 세 문서 모두 "배"가 있어 의미 구분이 어려움.
- query `"먹었더니 배가 불러"` → dense는 의미로 첫 문장을 잘 찾음.
- query `"A-12983"`, `"ERR_AUTH_1032"` → 의미 이해가 필요 없고 exact match가 중요 → BM25가 강함.

| 상황 | BM25 | Dense |
|---|---|---|
| 기준 | 단어 일치 | 의미 유사도 |
| 제품번호/코드/에러 메시지 | 매우 강함 | 불리 |
| 고유명사/제품명 | 강함 | 경우에 따라 약함 |
| 동의어/패러프레이징 | 약함 | 강함 |
| "로그인이 안 돼요" ↔ "인증에 실패했습니다" | 약할 수 있음 | 강함 |
| 자연어 Q&A | 보통 | 강함 |

**EnsembleRetriever**: 여러 retriever 결과를 **weighted Reciprocal Rank Fusion(RRF)** 으로 합친다 → 두 방식의 약점 보완.

**하나만 고른다면: 사용자 query 형태를 먼저 본다**

| 서비스 | 먼저 선택 |
|---|---|
| 사내 문서 Q&A, FAQ 챗봇, 자연어 지식검색, 논문 검색 | Dense |
| 고객지원 RAG | Dense 또는 Hybrid |
| 쇼핑, 법률/규정, 의료 문서 | Hybrid |
| 코드 검색 | BM25 또는 Hybrid |
| 로그, 에러코드, SKU 검색 | BM25 |

```text
자연어 질문이 대부분? ── YES → Dense
        │ NO
exact keyword 중요? ── YES → BM25
        │ NO → Dense
두 종류가 섞여 있다면 → Hybrid (BM25 + Dense) + Reranker
```

실제 사용자는 "출장비 규정 알려줘"(의미 검색)와 "HR-2024-031 문서 찾아줘"(exact)를 섞어서 묻기 때문에 현업에서 Hybrid를 많이 고려한다.

**한국어 BM25 주의**: BM25는 토큰 기반이라 조사(배가/배는/배를, 법인카드를/법인카드는)가 문제. 형태소 분석/적절한 tokenizer가 품질에 큰 영향 (LangChain `BM25Retriever`의 `preprocess_func`, Elasticsearch의 Nori analyzer 등). Dense는 표면형 변화에 상대적으로 덜 민감.

### 5.8 Lexical / Sparse / Dense 개념 정리

셋은 같은 레벨의 개념이 아니다.

- **Lexical**: 단어/토큰 **일치**를 중심으로 검색하는 방식
- **Sparse**: 대부분의 값이 0인 **희소 벡터 표현**을 쓰는 방식
- **Dense**: 대부분의 차원이 값을 가지는 **밀집 임베딩**으로 의미 검색하는 방식

실무에선 Lexical ≈ Sparse로 묶지만 엄밀히 동일하지 않다.

```text
Information Retrieval
├─ Sparse Retrieval
│  ├─ Traditional (Lexical): TF-IDF, BM25
│  └─ Learned Sparse: SPLADE
└─ Dense Retrieval: Embedding + Vector Search
```

| 구분 | Lexical | Sparse | Dense |
|---|---|---|---|
| 핵심 | 단어 일치 | 희소 벡터 기반 | 밀집 벡터 의미 검색 |
| 대표 | BM25, TF-IDF | BM25, SPLADE | OpenAI Embeddings, BGE, E5 |
| 정확한 단어 | 강함 | 강함 | 상대적으로 약함 |
| 동의어/의미 | 약함 | 방식에 따라 개선 | 강함 |

**Sparse 벡터 예시** — vocabulary 100,000개 중 몇 개만 non-zero

```text
"휴가 신청" → [0, 0, 0, 0, 1.8, 0, 2.3, 0, 0, ...]
```

**Learned sparse (SPLADE)** — 관련 토큰까지 확장해서 어느 정도 의미 정보도 담는다

```text
"자동차 수리" → 자동차 2.1, 수리 1.9, 차량 0.8, 정비 1.2
```

**Dense 벡터 예시** — 대부분의 차원에 값이 있다. cosine similarity / dot product / L2 distance로 비교

```text
"휴가 신청 방법" → [0.023, -0.71, 0.38, 0.19, 0.04, ...]
```

**query별 비교** (문서 A: "아이폰 17 Pro 배터리 교체 방법")

| Query | Lexical | Dense |
|---|---|---|
| "아이폰 17 Pro 배터리" | 매우 높음 | 높음 |
| "애플 폰 전지가 너무 빨리 닳아요" | 약함 (단어 겹침 거의 없음) | 강함 (애플 폰≈아이폰, 전지≈배터리) |
| "iPhone17Pro-A3294" | 강함 (exact token) | 의미 없음 |

> **한 문장씩**
> - Lexical = 단어가 같은가?
> - Sparse = 중요한 토큰들의 희소 벡터가 얼마나 겹치는가? (BM25가 대표 사례)
> - Dense = 문장의 의미가 비슷한가?
>
> BM25 = Lexical = Sparse / SPLADE = Sparse + 약간의 semantic / Embedding + FAISS·Qdrant·pgvector = Dense

### 5.9 프로덕션 검색 구성과 평가

**전형적인 구조**

```text
Query
 ├─▶ BM25/Sparse 검색 (top 20~100)
 └─▶ Dense 검색     (top 20~100)
        ↓
   Fusion (RRF)
        ↓
   Reranker (Cross Encoder, Cohere Rerank, BGE Reranker, ColBERT ...)
        ↓
   Top K → LLM
```

reranker는 비싸므로 전체 corpus가 아니라 retrieval로 뽑힌 **작은 후보 집합에만** 적용한다. 단순 `similarity_search(k=4)` RAG와 production RAG의 품질 차이가 여기서 생긴다.

**검색 백엔드**

| 시스템 | 특징 |
|---|---|
| Elasticsearch / OpenSearch | 전통 BM25 + vector/hybrid (OpenSearch는 RRF 결합 지원) |
| Qdrant | Vector DB, dense/sparse/hybrid 강점 |
| Pinecone | 관리형 vector DB, hybrid |
| Weaviate | Vector DB + hybrid |
| Milvus | 대규모 vector search |
| pgvector | PostgreSQL에 vector search 추가 |
| FAISS | 로컬/라이브러리 기반 vector search |
| Azure AI Search | 기업용 keyword/vector/hybrid |
| Vespa | 대규모 search/ranking |

**FAISS는 현업에서 안 쓰나?** — 쓴다. 다만 DB가 아니라 **라이브러리**. PoC에 최적(몇 줄이면 semantic retrieval 완성). 규모가 커지면 CRUD, metadata filtering, replication, sharding, backup, 접근 제어, 모니터링, 온라인 인덱스 업데이트가 필요해져 Qdrant/Pinecone/Milvus/pgvector/Elasticsearch 등으로 옮긴다. `BM25Retriever.from_texts()`도 개념 학습용이고, 대규모 lexical 검색은 Elasticsearch/OpenSearch를 검토.

**선택은 머리가 아니라 평가로**

1. 실제 서비스 query 100~500개 수집
2. 각 query의 정답 문서 라벨링
3. BM25 / Dense / Hybrid를 같은 query set으로 실행
4. 지표 비교: Recall@K, Hit Rate@K, MRR, nDCG, latency

| Retriever | Recall@5 | MRR | latency |
|---|---|---|---|
| BM25 | 0.72 | 0.61 | 20ms |
| Dense | 0.84 | 0.70 | 35ms |
| Hybrid | 0.91 | 0.78 | 48ms |

(예시 수치) → retriever 선택에서는 모델 이름보다 **자체 벤치마크 데이터**가 훨씬 중요하다.

**다음 공부 방향**: Retriever → Reranker → LLM 구조.
