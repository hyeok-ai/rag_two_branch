# LangChain-KR 예제 노트북 색인

코딩 중에 필요한 코드 조각을 빠르게 찾기 위한 색인입니다. (노트북 174개)

- **[빠른 찾기](#빠른-찾기-하고-싶은-일--노트북)**: "이걸 하고 싶다" → 어느 노트북을 열면 되는지
- **[노트북별 상세](#노트북별-상세)**: 각 노트북의 주제, 다루는 내용, 핵심 클래스/API, 노트북 안에서 정의한 재사용 가능한 함수·클래스

표기 규칙
- **핵심 API**: 노트북에서 import해 사용하는 주요 LangChain/외부 라이브러리 클래스·함수
- **정의**: 노트북 안에서 직접 `def` / `class`로 작성한 코드 (복사해서 재사용하기 좋은 부분)
- `langchain_teddynote`: 책 저자가 만든 보조 패키지입니다(`pip install langchain-teddynote`). 여기서 가져온 기능은 표준 LangChain에 없으므로, 가져다 쓸 때 의존성을 확인하세요.
- 대부분의 노트북은 시작 부분에서 `load_dotenv()`로 API 키를 불러오고 `langchain_teddynote.logging.langsmith()`로 LangSmith 추적을 켭니다. 아래 목록에서는 생략했습니다.

---

## 빠른 찾기: 하고 싶은 일 → 노트북

| 하고 싶은 일 | 노트북 |
|---|---|
| LLM 호출, 스트리밍 출력, 멀티모달(이미지) 입력 | [01-Basic/02-OpenAI-LLM](01-Basic/02-OpenAI-LLM.ipynb) |
| 프롬프트 → 모델 → 파서 체인(LCEL) 기본 | [01-Basic/03-LCEL](01-Basic/03-LCEL.Ipynb) |
| stream / invoke / batch / 비동기 호출 | [01-Basic/04-LCEL-Advanced](01-Basic/04-LCEL-Advanced.ipynb) |
| RunnablePassthrough / Parallel / Lambda | [01-Basic/05-Runnable](01-Basic/05-Runnable.ipynb), [13-LCEL/01](13-LangChain-Expression-Language/01-RunnablePassthrough.ipynb), [03](13-LangChain-Expression-Language/03-RunnableLambda.ipynb), [05](13-LangChain-Expression-Language/05-RunnableParallel.ipynb) |
| 프롬프트 템플릿, partial 변수, 파일(yaml)에서 로드 | [02-Prompt/01-PromptTemplate](02-Prompt/01-PromptTemplate.ipynb) |
| Few-shot 예제, 유사도 기반 Example Selector | [02-Prompt/02-FewShotTemplates](02-Prompt/02-FewShotTemplates.ipynb) |
| 요약·RAG·평가용 프롬프트 모음 | [02-Prompt/04-Personal-Prompts](02-Prompt/04-Personal-Prompts.ipynb) |
| LLM 출력을 Pydantic 객체 / JSON / 리스트 / Enum 등으로 받기 | [03-OutputParser](#03-outputparser--출력-파서) |
| `with_structured_output()` 사용 | [03-OutputParser/01-Pydantic](03-OutputParser/01-PydanticOuputParser.ipynb), [14-Chains/03](14-Chains/03-Structured-Output-Chain.ipynb) |
| 잘못된 형식의 출력 자동 수정 | [03-OutputParser/08-OutputFixingParser](03-OutputParser/08-OutputFixingParser.ipynb) |
| OpenAI 외 모델(Claude, Gemini, Cohere, Upstage, Perplexity, Together) | [04-Model/01-Chat-Models](04-Model/01-Chat-Models.ipynb), [05-Google](04-Model/05-Google-Generative-AI.ipynb) |
| 로컬 모델(Ollama, HuggingFace, GPT4All) | [04-Model/07~10](#04-model--다양한-llm) |
| LLM 응답 캐싱(InMemory / SQLite) | [04-Model/02-Cache](04-Model/02-Cache.ipynb) |
| 토큰 사용량·비용 확인 | [04-Model/04-TokenUsage](04-Model/04-TokenUsage.ipynb) |
| 체인 직렬화 / 저장 / 불러오기 | [04-Model/03-ModelSerialization](04-Model/03-ModelSerialization.ipynb) |
| 대화 기록(멀티턴) 유지 — 최신 방식 | [05-Memory/10](05-Memory/10-Conversation-With-History.ipynb), [13-LCEL/08-RunnableWithMessageHistory](13-LangChain-Expression-Language/08-RunnableWithMessageHistory.ipynb) |
| 대화 기록을 SQLite / Redis에 영구 저장 | [05-Memory/09-SQLite](05-Memory/09-Memory-using-SQLite.ipynb), [13-LCEL/08](13-LangChain-Expression-Language/08-RunnableWithMessageHistory.ipynb) |
| PDF 로드(PyPDF, PyMuPDF, PDFPlumber 등 비교) | [06-DocumentLoader/01-PDF-Loader](06-DocumentLoader/01-PDF-Loader.ipynb) |
| HWP / Word / PPT / Excel / CSV / JSON / 웹페이지 로드 | [06-DocumentLoader](#06-documentloader--문서-로더) |
| 텍스트 분할(청킹) | [07-TextSplitter](#07-textsplitter--텍스트-분할) |
| 의미 단위 분할(SemanticChunker) | [07-TextSplitter/04](07-TextSplitter/04-SemanticChunker.ipynb) |
| 임베딩 생성 / 유사도 계산 / 임베딩 캐시 | [08-Embeddings](#08-embeddings--임베딩) |
| 벡터DB(Chroma, FAISS, Pinecone) 생성·추가·삭제·저장 | [09-VectorStore](#09-vectorstore--벡터-저장소) |
| MMR, 점수 임계값 검색, top_k | [10-Retriever/01](10-Retriever/01-VectorStoreRetriever.ipynb) |
| BM25 + 벡터 하이브리드(앙상블) 검색, 한국어 형태소 BM25 | [10-Retriever/03](10-Retriever/03-EnsembleRetriever.ipynb), [10](10-Retriever/10-Kiwi-BM25Retriever.ipynb), [11](10-Retriever/11-CC-EnsembleRetriever.ipynb) |
| 검색 결과 재순위화(Reranker) | [11-Reranker](#11-reranker--재순위화) |
| 기본 RAG 파이프라인 전체 코드 | [12-RAG/00-RAG-Basic-PDF](12-RAG/00-RAG-Basic-PDF.ipynb) |
| 대화 기록을 유지하는 RAG | [12-RAG/03](12-RAG/03-Conversation-With-History.ipynb) |
| 이미지·표가 포함된 PDF의 멀티모달 RAG | [12-RAG/10](12-RAG/10-Multi_modal_RAG-GPT-4o.ipynb) |
| 입력에 따라 체인 분기(라우팅) | [13-LCEL/04-Routing](13-LangChain-Expression-Language/04-Routing.ipynb) |
| 실행 시점에 모델·프롬프트 교체(configurable) | [13-LCEL/06-Configure](13-LangChain-Expression-Language/06-Configure.ipynb) |
| API 오류 시 다른 모델로 대체(fallback) | [13-LCEL/11-Fallbacks](13-LangChain-Expression-Language/11-Fallbacks.ipynb) |
| 긴 문서 요약(Stuff, Map-Reduce, Refine, CoD) | [14-Chains/01-Summary](14-Chains/01-Summary.ipynb) |
| 자연어 → SQL | [14-Chains/02-SQL](14-Chains/02-SQL.ipynb), [17-LangGraph/03/09-SQL-Agent](17-LangGraph/03-Use-Cases/09-LangGraph-SQL-Agent.ipynb) |
| Pandas DataFrame / CSV / Excel 분석 에이전트 | [14-Chains/04](14-Chains/04-Structured-Data-Chat.ipynb), [15-Agent/07](15-Agent/07-CSV-Excel-Agent.ipynb) |
| 커스텀 도구(@tool) 정의 / LLM에 도구 바인딩 | [15-Agent/01-Tools](15-Agent/01-Tools.ipynb), [02-Bind-Tools](15-Agent/02-Bind-Tools.ipynb) |
| Tool Calling Agent + AgentExecutor | [15-Agent/03-Agent](15-Agent/03-Agent.ipynb) |
| 웹 검색 + 문서 검색 에이전트(Agentic RAG) | [15-Agent/06](15-Agent/06-Agentic-RAG.ipynb), [17-LangGraph/02/06](17-LangGraph/02-Structures/06-LangGraph-Agentic-RAG.ipynb) |
| 파일 읽기/쓰기 도구, DALL-E 이미지 생성 도구 | [15-Agent/08](15-Agent/08-Agent-Toolkits-File-Management.ipynb), [09](15-Agent/09-Agent-Report-With-Image-Generation.ipynb) |
| RAG 테스트셋 생성 / RAGAS 평가 | [16-Evaluations/01](16-Evaluations/01-Test-Dataset-Generator-RAGAS.ipynb), [02](16-Evaluations/02-Evaluation-Using-RAGAS.ipynb) |
| LangSmith 기반 평가(LLM-as-Judge, 휴리스틱, 커스텀) | [16-Evaluations/04~14](#16-evaluations--평가) |
| LangGraph 기본(State/Node/Edge) 챗봇 | [17-LangGraph/01/02-ChatBot](17-LangGraph/01-Core-Features/02-LangGraph-ChatBot.ipynb) |
| LangGraph 에이전트 + 메모리(체크포인트) | [17-LangGraph/01/03](17-LangGraph/01-Core-Features/03-LangGraph-Agent.ipynb), [04](17-LangGraph/01-Core-Features/04-LangGraph-Agent-With-Memory.ipynb) |
| LangGraph 중단(interrupt) / 사람 개입 / 상태 수정 | [17-LangGraph/01/06~08](17-LangGraph/01-Core-Features/06-LangGraph-Human-In-the-Loop.ipynb) |
| LangGraph 병렬 분기(fan-out/fan-in), 서브그래프 | [17-LangGraph/01/11](17-LangGraph/01-Core-Features/11-LangGraph-Branching.ipynb), [13](17-LangGraph/01-Core-Features/13-LangGraph-Subgraph.ipynb), [14](17-LangGraph/01-Core-Features/14-LangGraph-Subgraph-Transform-State.ipynb) |
| LangGraph 스트리밍 모드(values/updates/messages) | [17-LangGraph/01/05](17-LangGraph/01-Core-Features/05-LangGraph-Streaming-Outputs.ipynb), [15](17-LangGraph/01-Core-Features/15-LangGraph-Streaming-Steps.ipynb) |
| 고급 RAG 그래프(CRAG, Self-RAG, Adaptive RAG) | [17-LangGraph/02/07](17-LangGraph/02-Structures/07-LangGraph-Adaptive-RAG.ipynb), [03/03](17-LangGraph/03-Use-Cases/03-LangGraph-CRAG.ipynb), [03/04](17-LangGraph/03-Use-Cases/04-LangGraph-Self-RAG.ipynb) |
| 멀티 에이전트(협업 / 감독자 / 계층형 팀) | [17-LangGraph/03/06~08](#17-langgraph--03-use-cases-응용-사례) |
| Plan-and-Execute 에이전트 | [17-LangGraph/03/05](17-LangGraph/03-Use-Cases/05-LangGraph-Plan-and-Execute.ipynb) |
| 파인튜닝용 QA 데이터 생성 / Unsloth 파인튜닝 | [18-FineTuning](#18-finetuning--파인튜닝) |
| YouTube 음성 → 텍스트 → QA | [99-Projects/01](99-Projects/01-YouTube-Transcript-Summarize/youtube-qa-bot.ipynb) |

---

## 노트북별 상세

### 01-Basic — 기초

#### [01-OpenAI-APIKey.ipynb](01-Basic/01-OpenAI-APIKey.ipynb)
OpenAI API 키 발급과 `.env` 설정, 설치된 패키지 버전 확인.

#### [02-OpenAI-LLM.ipynb](01-Basic/02-OpenAI-LLM.ipynb)
`ChatOpenAI` 기본 사용법.
- 다루는 내용: AIMessage 응답 구조, LogProb 활성화, 스트리밍 출력, 프롬프트 캐싱, 멀티모달(이미지 인식), System/User 프롬프트 지정
- 핵심 API: `ChatOpenAI`, `get_openai_callback`, `langchain_teddynote`의 `stream_response`, `MultiModal`

#### [03-LCEL.Ipynb](01-Basic/03-LCEL.Ipynb)
LCEL 기본 형태 `prompt | model | output_parser`.
- 다루는 내용: PromptTemplate, 체인 생성, `invoke()`, 출력 파서, 템플릿 교체
- 핵심 API: `PromptTemplate`, `ChatOpenAI`, `StrOutputParser`

#### [04-LCEL-Advanced.ipynb](01-Basic/04-LCEL-Advanced.ipynb)
Runnable 인터페이스의 호출 방식 모음.
- 다루는 내용: `stream`, `invoke`, `batch`, `astream`, `ainvoke`, `abatch`, `RunnableParallel`로 병렬 실행, 배치에서의 병렬 처리
- 핵심 API: `RunnableParallel`

#### [05-Runnable.ipynb](01-Basic/05-Runnable.ipynb)
체인 사이에서 데이터를 넘기는 방법.
- 다루는 내용: `RunnablePassthrough`, `RunnableParallel`, `RunnableLambda`, `itemgetter`
- 정의: `get_today`(오늘 날짜를 프롬프트에 넣기), `length_function`, `multiple_length_function`

---

### 02-Prompt — 프롬프트

#### [01-PromptTemplate.ipynb](02-Prompt/01-PromptTemplate.ipynb)
프롬프트 템플릿 작성법 전반.
- 다루는 내용: `from_template()`, 생성자 방식, `partial_variables`(함수로 날짜 채우기 등), yaml 파일에서 `load_prompt`, `ChatPromptTemplate`, `MessagesPlaceholder`
- 핵심 API: `PromptTemplate`, `ChatPromptTemplate`, `MessagesPlaceholder`, `load_prompt`
- 정의: `get_today`

#### [02-FewShotTemplates.ipynb](02-Prompt/02-FewShotTemplates.ipynb)
Few-shot 프롬프트.
- 다루는 내용: `FewShotPromptTemplate`, Example Selector(유사도, MMR), `FewShotChatMessagePromptTemplate`, 유사도 기반 예제 선택의 한계와 해결
- 핵심 API: `SemanticSimilarityExampleSelector`, `MaxMarginalRelevanceExampleSelector`, `Chroma`, `langchain_teddynote`의 `CustomExampleSelector`

#### [03-LangChain-Hub.ipynb](02-Prompt/03-LangChain-Hub.ipynb)
LangChain Hub에서 프롬프트 가져오기(`hub.pull`), 내 프롬프트 등록(`hub.push`).

#### [04-Personal-Prompts.ipynb](02-Prompt/04-Personal-Prompts.ipynb)
**실전 프롬프트 모음** (복사해서 쓰기 좋음).
- 포함 프롬프트: Stuff 요약, Map 프롬프트, Reduce 프롬프트, Metadata Tagger, Chain of Density 요약(영/한), RAG 문서 프롬프트, 출처 포함 RAG, LLM 평가 프롬프트

#### [05-ChatPromptTemplate.ipynb](02-Prompt/05-ChatPromptTemplate.ipynb)
`ChatPromptTemplate`을 파일(`load_prompt`)에서 불러오는 짧은 예제.

---

### 03-OutputParser — 출력 파서

#### [01-PydanticOuputParser.ipynb](03-OutputParser/01-PydanticOuputParser.ipynb)
LLM 출력을 Pydantic 모델로 파싱(이메일 요약 예제).
- 다루는 내용: `get_format_instructions()`, 파서가 붙은 체인, `with_structured_output()`
- 핵심 API: `PydanticOutputParser`, `BaseModel`, `Field`
- 정의: `EmailSummary`(Pydantic 모델)

#### [02-CommaSeparatedListOutputParser.ipynb](03-OutputParser/02-CommaSeparatedListOutputParser.ipynb)
쉼표로 구분된 리스트 출력 → Python list.

#### [03-StructuredOutputParser.ipynb](03-OutputParser/03-StructuredOutputParser.ipynb)
`ResponseSchema`로 필드를 정의해 dict로 받기(Pydantic 없이 간단히).

#### [04-JsonOutputParser.ipynb](03-OutputParser/04-JsonOutputParser.ipynb)
JSON 출력 파싱(Pydantic 스키마 지정 방식 / 스키마 없는 방식).
- 정의: `Topic`

#### [05-PandasDataFrameOutputParser.ipynb](03-OutputParser/05-PandasDataFrameOutputParser.ipynb)
DataFrame에 대한 질의(열 조회, 평균 등)를 LLM으로 구조화.
- 정의: `format_parser_output`

#### [06-DatetimeOutputParser.ipynb](03-OutputParser/06-DatetimeOutputParser.ipynb)
출력을 `datetime` 객체로 파싱.

#### [07-EnumOutputParser.ipynb](03-OutputParser/07-EnumOutputParser.ipynb)
출력을 정해진 Enum 값 중 하나로 제한(분류 작업에 유용).
- 정의: `Colors`(Enum)

#### [08-OutputFixingParser.ipynb](03-OutputParser/08-OutputFixingParser.ipynb)
파싱 실패 시 LLM이 출력을 고쳐서 다시 파싱.
- 핵심 API: `OutputFixingParser.from_llm()`, `PydanticOutputParser`
- 정의: `Actor`

---

### 04-Model — 다양한 LLM

#### [01-Chat-Models.ipynb](04-Model/01-Chat-Models.ipynb)
여러 LLM 제공사 연결 방법과 특징 설명.
- 다루는 모델: OpenAI, Anthropic Claude, Perplexity, Together AI, Cohere(Command R+, Aya), Upstage Solar, Xionic, LogicKor(한국어 벤치마크 소개)
- 핵심 API: `ChatOpenAI`, `ChatAnthropic`, `ChatPerplexity`(teddynote), `ChatTogether`, `ChatCohere`, `ChatUpstage`
- 정의: `Lotto`(구조화 출력 예제)

#### [02-Cache.ipynb](04-Model/02-Cache.ipynb)
같은 질문의 LLM 응답 캐싱.
- 핵심 API: `set_llm_cache`, `InMemoryCache`, `SQLiteCache`

#### [03-ModelSerialization.ipynb](04-Model/03-ModelSerialization.ipynb)
체인 직렬화: `dumps`/`dumpd`로 JSON 저장, pickle 저장, `load`/`loads`로 복원.

#### [04-TokenUsage.ipynb](04-Model/04-TokenUsage.ipynb)
`get_openai_callback()`으로 토큰 수와 비용 확인.

#### [05-Google-Generative-AI.ipynb](04-Model/05-Google-Generative-AI.ipynb)
Gemini 사용.
- 다루는 내용: API 키, Safety Settings(`HarmCategory`, `HarmBlockThreshold`), batch 실행, 멀티모달
- 핵심 API: `ChatGoogleGenerativeAI`

#### [06-HuggingFace-Endpoint.ipynb](04-Model/06-HuggingFace-Endpoint.ipynb)
HuggingFace Inference API 사용(서버리스 / 전용 엔드포인트).
- 핵심 API: `HuggingFaceEndpoint`, `huggingface_hub.login`

#### [07-HuggingFace-Local.ipynb](04-Model/07-HuggingFace-Local.ipynb)
`HuggingFacePipeline.from_model_id()`로 로컬 모델 실행(짧은 예제).

#### [08-Huggingface-Pipelines.ipynb](04-Model/08-Huggingface-Pipelines.ipynb)
로컬 HF 파이프라인 상세.
- 다루는 내용: 모델 로딩, Gated 모델 사용, 체인 구성, GPU 추론, 배치 GPU 추론
- 핵심 API: `HuggingFacePipeline`, `ChatHuggingFace`, `transformers.pipeline`

#### [09-Ollama.ipynb](04-Model/09-Ollama.ipynb)
Ollama 로컬 모델.
- 다루는 내용: 설치, 모델 다운로드, Modelfile로 커스텀 모델 생성, JSON 출력 형식, 멀티모달(이미지 base64 입력)
- 핵심 API: `ChatOllama`
- 정의: `convert_to_base64`(PIL 이미지→base64), `plt_img_base64`(base64 이미지 표시), `prompt_func`

#### [10-GPT4ALL.ipynb](04-Model/10-GPT4ALL.ipynb)
GPT4All 로컬 모델과 스트리밍 콜백(`StreamingStdOutCallbackHandler`).

#### [11-Gemini-Video.ipynb](04-Model/11-Gemini-Video.ipynb)
`google.generativeai`로 비디오 파일 업로드 후 영상 내용에 대한 질의응답, 업로드 파일 삭제.

---

### 05-Memory — 대화 메모리

> 01~07은 레거시 `langchain.memory` 클래스입니다. 새 코드에는 **10번**이나 **13-LCEL/08**의 `RunnableWithMessageHistory` 방식을 권장합니다.

#### [01-ConversationBufferMemory.ipynb](05-Memory/01-ConversationBufferMemory.ipynb)
전체 대화 저장 메모리, `ConversationChain`에 적용.

#### [02-ConversationBufferWindowMemory.ipynb](05-Memory/02-ConversationBufferWindowMemory.ipynb)
최근 k개 대화만 유지.

#### [03-ConversationTokenBufferMemory.ipynb](05-Memory/03-ConversationTokenBufferMemory.ipynb)
토큰 수 기준으로 대화 유지.

#### [04-ConversationEntityMemory.ipynb](05-Memory/04-ConversationEntityMemory.ipynb)
대화 속 개체(entity) 정보를 추출해 기억.

#### [05-ConversationKnowledgeGraph.ipynb](05-Memory/05-ConversationKnowledgeGraph.ipynb)
대화 내용을 지식 그래프(트리플)로 저장(`ConversationKGMemory`).

#### [06-ConversationSummary.ipynb](05-Memory/06-ConversationSummary.ipynb)
대화 요약 메모리(`ConversationSummaryMemory`), 요약 + 최근 버퍼(`ConversationSummaryBufferMemory`).

#### [07-VectorStoreRetrieverMemory.ipynb](05-Memory/07-VectorStoreRetrieverMemory.ipynb)
대화를 FAISS에 저장하고 관련 있는 과거 대화만 검색.

#### [08-LCEL-add-memory.ipynb](05-Memory/08-LCEL-add-memory.ipynb)
LCEL 체인에 메모리를 수동으로 연결.
- 다루는 내용: `memory.load_memory_variables` + `RunnableLambda` + `itemgetter`, 커스텀 ConversationChain
- 정의: `MyConversationChain`(메모리를 내장한 커스텀 Runnable 클래스)

#### [09-Memory-using-SQLite.ipynb](05-Memory/09-Memory-using-SQLite.ipynb)
SQLite에 대화 기록 영구 저장.
- 다루는 내용: `user_id` + `conversation_id` 두 개의 키로 세션 구분
- 핵심 API: `SQLChatMessageHistory`, `RunnableWithMessageHistory`, `ConfigurableFieldSpec`
- 정의: `get_chat_history`

#### [10-Conversation-With-History.ipynb](05-Memory/10-Conversation-With-History.ipynb)
**멀티턴 체인 기본 템플릿** (session_id별 in-memory 기록).
- 핵심 API: `ChatMessageHistory`, `RunnableWithMessageHistory`, `MessagesPlaceholder`
- 정의: `get_session_history`

---

### 06-DocumentLoader — 문서 로더

#### [00-Document-Loader.ipynb](06-DocumentLoader/00-Document-Loader.ipynb)
`Document` 객체 구조(page_content, metadata)와 로더 메서드: `load()`, `load_and_split()`, `lazy_load()`, `aload()`.

#### [01-PDF-Loader.ipynb](06-DocumentLoader/01-PDF-Loader.ipynb)
PDF 로더 비교(AutoRAG 팀 실험 결과 포함).
- 다루는 로더: `PyPDFLoader`(OCR 옵션 포함), `PyMuPDFLoader`, `UnstructuredPDFLoader`, `PyPDFium2Loader`, `PDFMinerLoader`, `PDFMinerPDFasHTMLLoader`(+BeautifulSoup로 구조 파싱), `PyPDFDirectoryLoader`, `PDFPlumberLoader`
- 정의: `show_metadata`

#### [02-HWP-Loader.ipynb](06-DocumentLoader/02-HWP-Loader.ipynb)
한글(HWP) 파일 로드 — `langchain_teddynote.document_loaders.HWPLoader`.

#### [03-CSV-Loader.ipynb](06-DocumentLoader/03-CSV-Loader.ipynb)
`CSVLoader`(구분자·컬럼 커스터마이징, source_column), `UnstructuredCSVLoader`, pandas `DataFrameLoader`.

#### [04-Excel-Loader.ipynb](06-DocumentLoader/04-Excel-Loader.ipynb)
`UnstructuredExcelLoader`(HTML 모드), pandas로 읽은 뒤 `DataFrameLoader`.

#### [05-Word-Loader.ipynb](06-DocumentLoader/05-Word-Loader.ipynb)
`Docx2txtLoader`, `UnstructuredWordDocumentLoader`(elements 모드).

#### [06-PowerPoint-Loader.ipynb](06-DocumentLoader/06-PowerPoint-Loader.ipynb)
`UnstructuredPowerPointLoader`.

#### [07-WebBase-Loader.ipynb](06-DocumentLoader/07-WebBase-Loader.ipynb)
웹페이지 로드.
- 다루는 내용: `bs4.SoupStrainer`로 특정 태그만 추출, 여러 URL 동시 로드(`aload`, `nest_asyncio`), 프록시 설정(`proxies`)

#### [08-TXT-Loader.ipynb](06-DocumentLoader/08-TXT-Loader.ipynb)
`TextLoader`, 폴더 내 txt 일괄 로드 시 인코딩 자동 감지(`autodetect_encoding`).

#### [09-JSON-Loader.ipynb](06-DocumentLoader/09-JSON-Loader.ipynb)
`JSONLoader`에 `jq_schema`로 필요한 필드만 추출.

#### [10-Arxiv-Loader.ipynb](06-DocumentLoader/10-Arxiv-Loader.ipynb)
arXiv 논문 검색·로드, 요약만 가져오기(`get_summaries_as_docs`), `lazy_load`.

#### [11-Directory-Loader.ipynb](06-DocumentLoader/11-Directory-Loader.ipynb)
`DirectoryLoader`(glob, 진행률 표시, 멀티스레딩, `loader_cls` 변경 — `TextLoader`, `PythonLoader`).

#### [12-UpstageLayoutAnalysisLoader.ipynb](06-DocumentLoader/12-UpstageLayoutAnalysisLoader.ipynb)
Upstage 레이아웃 분석 API로 PDF를 구조화(표·레이아웃 보존).

#### [13-Llamaparser.ipynb](06-DocumentLoader/13-Llamaparser.ipynb)
LlamaParse로 PDF 파싱(마크다운 출력), 멀티모달 모델로 파싱, LangChain Document로 변환.

---

### 07-TextSplitter — 텍스트 분할

#### [01-CharacterTextSplitter.ipynb](07-TextSplitter/01-CharacterTextSplitter.ipynb)
구분자 기준 분할, `chunk_size`, `chunk_overlap`, 메타데이터 부여(`create_documents`).

#### [02-RecursiveCharacterTextSplitter.ipynb](07-TextSplitter/02-RecursiveCharacterTextSplitter.ipynb)
**가장 일반적인 분할기.** 단락→문장→단어 순으로 재귀 분할.

#### [03-TokenTextSplitter.ipynb](07-TextSplitter/03-TokenTextSplitter.ipynb)
토큰 기준 분할 모음.
- 다루는 방식: tiktoken(`from_tiktoken_encoder`), `TokenTextSplitter`, spaCy, SentenceTransformers, NLTK, KoNLPy(Kkma, 한국어), HuggingFace 토크나이저(`from_huggingface_tokenizer`)

#### [04-SemanticChunker.ipynb](07-TextSplitter/04-SemanticChunker.ipynb)
임베딩 유사도로 의미 단위 분할.
- 다루는 내용: breakpoint 방식 — percentile, standard_deviation, interquartile
- 핵심 API: `SemanticChunker`(langchain_experimental)

#### [05-CodeSplitter.ipynb](07-TextSplitter/05-CodeSplitter.ipynb)
프로그래밍 언어 문법에 맞춘 분할: `RecursiveCharacterTextSplitter.from_language(Language.PYTHON)` 등 — Python, JS, TS, Markdown, LaTeX, HTML, Solidity, C#.

#### [06-MarkdownHeaderTextSplitter.ipynb](07-TextSplitter/06-MarkdownHeaderTextSplitter.ipynb)
마크다운 헤더(#, ##, ###) 기준 분할 → 헤더를 메타데이터로 보존, 이후 Recursive 분할과 연결.

#### [07-HTMLHeaderTextSplitter.ipynb](07-TextSplitter/07-HTMLHeaderTextSplitter.ipynb)
HTML 헤더 태그 기준 분할, URL에서 직접 로드 후 다른 분할기와 파이프라인 연결.

#### [08-RecursiveJsonSplitter.ipynb](07-TextSplitter/08-RecursiveJsonSplitter.ipynb)
JSON을 구조를 유지하며 분할(`split_json`, `create_documents`, `split_text`, 리스트→dict 변환 옵션).

---

### 08-Embeddings — 임베딩

#### [01-OpenAIEmbeddings.ipynb](08-Embeddings/01-OpenAIEmbeddings.ipynb)
`embed_query` / `embed_documents`, `dimensions`로 차원 축소, 코사인 유사도 계산.
- 정의: `similarity`

#### [02-CacheBackedEmbeddings.ipynb](08-Embeddings/02-CacheBackedEmbeddings.ipynb)
임베딩 결과 캐싱으로 재계산 방지 — `LocalFileStore`(디스크 영구 저장), `InMemoryByteStore`(메모리).
- 핵심 API: `CacheBackedEmbeddings.from_bytes_store`

#### [03-HuggingFaceEmbeddings.ipynb](08-Embeddings/03-HuggingFaceEmbeddings.ipynb)
오픈소스 임베딩.
- 다루는 내용: `HuggingFaceEndpointEmbeddings`(API), 로컬 `HuggingFaceEmbeddings`(`multilingual-e5-large-instruct`), **BGE-M3** — `FlagEmbedding`의 Dense / Sparse(lexical weight) / Multi-Vector(ColBERT)

#### [04-UpstageEmbeddings.ipynb](08-Embeddings/04-UpstageEmbeddings.ipynb)
Upstage 임베딩(쿼리용 / 문서용 모델 분리), 유사도 계산.

#### [05-OllamaEmbeddings.ipynb](08-Embeddings/05-OllamaEmbeddings.ipynb)
Ollama 로컬 임베딩.

#### [06-llamacpp.ipynb](08-Embeddings/06-llamacpp.ipynb)
llama.cpp(GGUF) 모델로 임베딩.

#### [07-GPT4ALL.ipynb](08-Embeddings/07-GPT4ALL.ipynb)
GPT4All 임베딩.

---

### 09-VectorStore — 벡터 저장소

#### [01-Chroma.ipynb](09-VectorStore/01-Chroma.ipynb)
Chroma 전체 CRUD + 멀티모달.
- 다루는 내용: `from_documents`/`from_texts`, 디스크 저장(`persist_directory`), `similarity_search`(메타데이터 필터, 점수 포함), `add_documents`/`add_texts`(id 지정 업서트), `delete`, `reset_collection`, `as_retriever`
- 멀티모달: `OpenCLIPEmbeddings`로 이미지 임베딩·저장, 텍스트로 이미지 검색
- 정의: `ImageRetriever`

#### [02-FAISS.ipynb](09-VectorStore/02-FAISS.ipynb)
FAISS 전체 사용법.
- 다루는 내용: 빈 인덱스 직접 생성(`faiss.IndexFlatL2` + `InMemoryDocstore`), `from_documents`/`from_texts`, 유사도 검색(필터), `add_documents`/`add_texts`, `delete`, `save_local`/`load_local`, `merge_from`(인덱스 병합), `as_retriever`(mmr, 임계값, k)

#### [03-Pinecone.ipynb](09-VectorStore/03-Pinecone.ipynb)
Pinecone + 한국어 하이브리드 검색(`langchain_teddynote.community.pinecone` 활용).
- 다루는 내용: 한국어 불용어, 문서 전처리, 인덱스 생성, Sparse 인코더(BM25) 생성·저장, 업서트(병렬 포함), 네임스페이스/필터 삭제, `PineconeKiwiHybridRetriever`(dense+sparse, alpha 조절), Reranking

---

### 10-Retriever — 검색기

#### [01-VectorStoreRetriever.ipynb](10-Retriever/01-VectorStoreRetriever.ipynb)
`as_retriever()` 옵션.
- 다루는 내용: MMR(`fetch_k`, `lambda_mult`), `similarity_score_threshold`, top_k, `ConfigurableField`로 실행 시 검색 옵션 변경, 쿼리/문서 임베딩 모델이 다른 경우(Upstage)

#### [02-ContextualCompressionRetriever.ipynb](10-Retriever/02-ContextualCompressionRetriever.ipynb)
검색 결과에서 관련 부분만 추출/필터링.
- 다루는 내용: `LLMChainExtractor`, `LLMChainFilter`, `EmbeddingsFilter`(임계값), `DocumentCompressorPipeline`(분할 + 중복 제거 `EmbeddingsRedundantFilter` + 필터)
- 정의: `pretty_print_docs`

#### [03-EnsembleRetriever.ipynb](10-Retriever/03-EnsembleRetriever.ipynb)
BM25(키워드) + FAISS(의미) 앙상블, 가중치 설정, 런타임 config로 가중치 변경.

#### [04-LongContextReorder.ipynb](10-Retriever/04-LongContextReorder.ipynb)
"Lost in the middle" 대응: 관련도 높은 문서를 앞/뒤로 재배치한 뒤 QA 체인 구성.
- 정의: `format_docs`, `reorder_documents`

#### [05-ParentDocumentRetriever.ipynb](10-Retriever/05-ParentDocumentRetriever.ipynb)
작은 청크로 검색하고 큰 부모 청크/원문을 반환.
- 다루는 내용: 전체 문서 반환, parent/child splitter 크기 조절
- 핵심 API: `ParentDocumentRetriever`, `InMemoryStore`

#### [06-MultiQueryRetriever.ipynb](10-Retriever/06-MultiQueryRetriever.ipynb)
LLM으로 질문을 여러 관점으로 재작성해 검색 후 결과 합치기, 커스텀 프롬프트로 LCEL 구성.

#### [07-MultiVectorRetriever.ipynb](10-Retriever/07-MultiVectorRetriever.ipynb)
문서당 여러 벡터 저장.
- 다루는 내용: 작은 청크 → 원본 반환, **요약본**을 임베딩, **가설 질문(Hypothetical Queries)** 생성 후 임베딩
- 핵심 API: `MultiVectorRetriever`, `InMemoryStore`, `SearchType`, `JsonKeyOutputFunctionsParser`

#### [08-SelfQueryRetriever.ipynb](10-Retriever/08-SelfQueryRetriever.ipynb)
자연어 질문에서 메타데이터 필터를 자동 생성해 검색.
- 다루는 내용: `AttributeInfo`로 메타데이터 스키마 정의, `query_constructor` 체인, `ChromaTranslator`

#### [09-TimeWeightedVectorStoreRetriever.ipynb](10-Retriever/09-TimeWeightedVectorStoreRetriever.ipynb)
최근성 가중 검색, `decay_rate` 높고 낮음 비교, `mock_now`로 가상 시간 테스트.

#### [10-Kiwi-BM25Retriever.ipynb](10-Retriever/10-Kiwi-BM25Retriever.ipynb)
**한국어 BM25 튜닝.** 형태소 분석기로 토큰화한 BM25 비교.
- 다루는 내용: Kiwi, Kkma, Okt 토크나이저 BM25 vs FAISS vs 앙상블 결과 비교
- 정의: `kiwi_tokenize`, `kkma_tokenize`, `okt_tokenize`, `print_search_results`

#### [11-CC-EnsembleRetriever.ipynb](10-Retriever/11-CC-EnsembleRetriever.ipynb)
앙상블 결합 방식 비교: RRF vs Convex Combination(CC) — `langchain_teddynote`의 `EnsembleRetriever`, `KiwiBM25Retriever`.
- 정의: `pretty_print`

---

### 11-Reranker — 재순위화

모두 `ContextualCompressionRetriever`에 reranker를 compressor로 넣는 동일한 패턴입니다. 정의: `pretty_print_docs`

#### [01-Cross-Encoder-Reranker.ipynb](11-Reranker/01-Cross-Encoder-Reranker.ipynb)
HuggingFace Cross-Encoder(`BAAI/bge-reranker` 등) 로컬 reranker, Reranker의 장점·문서 수 설정·트레이드오프 설명.
- 핵심 API: `HuggingFaceCrossEncoder`, `CrossEncoderReranker`

#### [02-Cohere-Reranker.ipynb](11-Reranker/02-Cohere-Reranker.ipynb)
`CohereRerank`(다국어 모델) API.

#### [03-Jina-Reranker.ipynb](11-Reranker/03-Jina-Reranker.ipynb)
`JinaRerank` API.

#### [04-FlashRank-Reranker.ipynb](11-Reranker/04-FlashRank-Reranker.ipynb)
`FlashrankRerank` — 가벼운 로컬 reranker.

---

### 12-RAG — RAG

#### [00-RAG-Basic-PDF.ipynb](12-RAG/00-RAG-Basic-PDF.ipynb)
**RAG 기본 8단계 + 전체 코드 한 셀** (PDF → PyMuPDF → 분할 → 임베딩 → FAISS → retriever → prompt → LLM → 체인). 가장 많이 복사하게 될 템플릿.

#### [01-RAG-Basic-Webloader.ipynb](12-RAG/01-RAG-Basic-Webloader.ipynb)
네이버 뉴스 기사를 `WebBaseLoader`로 로드해 QA 챗봇 구성.

#### [02-RAG-Advanced.ipynb](12-RAG/02-RAG-Advanced.ipynb)
RAG 각 단계별 선택지 총정리.
- 로드: 웹, PDF, CSV, TXT, 폴더, Python 파일
- 분할: Character, Recursive, SemanticChunker
- 임베딩: OpenAI, HuggingFace BGE, FastEmbed
- 벡터스토어: FAISS, Chroma
- 검색기: 유사도, MultiQuery, Ensemble(BM25+FAISS)
- 프롬프트/LLM 생성, RAG 템플릿 실험
- 정의: `format_docs`, `pretty_print`

#### [03-Conversation-With-History.ipynb](12-RAG/03-Conversation-With-History.ipynb)
대화 기록을 유지하는 RAG (`RunnableWithMessageHistory` + PDFPlumber + FAISS).
- 정의: `get_session_history`

#### [04-RAPTOR-Long-Context-RAG-CODE.ipynb](12-RAG/04-RAPTOR-Long-Context-RAG-CODE.ipynb)
RAPTOR(재귀 요약 트리) — LangChain 문서 웹페이지(`RecursiveUrlLoader`) 대상.
- 다루는 내용: UMAP 차원 축소, GMM 클러스터링(BIC로 최적 클러스터 수), 클러스터 요약을 재귀적으로 반복, 요약+원문을 함께 인덱싱
- 정의: `num_tokens_from_string`, `global_cluster_embeddings`, `local_cluster_embeddings`, `get_optimal_clusters`, `GMM_cluster`, `perform_clustering`, `embed`, `embed_cluster_texts`, `fmt_txt`, `embed_cluster_summarize_texts`, `recursive_embed_cluster_summarize`, `format_docs`

#### [05-RAPTOR-Long-Context-RAG-PDF.ipynb](12-RAG/05-RAPTOR-Long-Context-RAG-PDF.ipynb)
04와 동일한 RAPTOR를 PDF 문서에 적용(함수 구성 동일).

#### [08-Web-Summarize-Chain-Of-Density.ipynb](12-RAG/08-Web-Summarize-Chain-Of-Density.ipynb)
웹 문서를 Chain of Density 방식으로 요약하고 반복 단계별 결과를 JSON으로 스트리밍.
- 정의: `StreamCallback`(스트리밍 콜백 핸들러)

#### [10-Multi_modal_RAG-GPT-4o.ipynb](12-RAG/10-Multi_modal_RAG-GPT-4o.ipynb)
**텍스트·표·이미지가 섞인 PDF의 멀티모달 RAG.**
- 다루는 내용: `unstructured.partition_pdf`로 텍스트/표/이미지 추출, 텍스트·표 요약, 이미지 요약(GPT-4o), MultiVectorRetriever에 요약 저장·원본 반환, 검색 결과의 이미지를 LLM에 함께 입력
- 정의: `extract_pdf_elements`, `categorize_elements`, `generate_text_summaries`, `encode_image`, `image_summarize`, `generate_img_summaries`, `create_multi_vector_retriever`, `plt_img_base64`, `looks_like_base64`, `is_image_data`, `resize_base64_image`, `split_image_text_types`, `img_prompt_func`, `multi_modal_rag_chain`

---

### 13-LangChain-Expression-Language — LCEL 심화

#### [01-RunnablePassthrough.ipynb](13-LangChain-Expression-Language/01-RunnablePassthrough.ipynb)
입력 그대로 전달, `.assign()`으로 키 추가, retriever 예제(`{"context": retriever | format_docs, "question": RunnablePassthrough()}`).

#### [02-Inspect-Runnables.ipynb](13-LangChain-Expression-Language/02-Inspect-Runnables.ipynb)
체인 구조 확인: `get_graph()`, `print_ascii()`, `get_prompts()`.

#### [03-RunnableLambda.ipynb](13-LangChain-Expression-Language/03-RunnableLambda.ipynb)
사용자 함수를 체인에 넣기, 인자 여러 개 처리, `RunnableConfig`를 인자로 받기(콜백 전달).
- 정의: `length_function`, `multiple_length_function`, `parse_or_fix`(JSON 파싱 실패 시 LLM으로 고치기)

#### [04-Routing.ipynb](13-LangChain-Expression-Language/04-Routing.ipynb)
입력을 분류해 다른 체인으로 보내기 — 사용자 정의 함수 라우팅(권장), `RunnableBranch`.
- 정의: `route`

#### [05-RunnableParallel.ipynb](13-LangChain-Expression-Language/05-RunnableParallel.ipynb)
입출력 조작, `itemgetter` 단축, 여러 체인 병렬 실행.

#### [06-Configure.ipynb](13-LangChain-Expression-Language/06-Configure.ipynb)
런타임 구성 변경.
- 다루는 내용: `configurable_fields`(model_name 등), `configurable_alternatives`(LLM 교체: OpenAI↔Anthropic, 프롬프트 교체), `HubRunnable`, `with_config`로 설정 저장

#### [07-ChainDecorator.ipynb](13-LangChain-Expression-Language/07-ChainDecorator.ipynb)
`@chain` 데코레이터로 함수를 Runnable로 만들기.
- 정의: `custom_chain`

#### [08-RunnableWithMessageHistory.ipynb](13-LangChain-Expression-Language/08-RunnableWithMessageHistory.ipynb)
**메시지 기록 추가 상세.**
- 다루는 내용: in-memory 기록, `user_id`+`conversation_id` 등 여러 키(`ConfigurableFieldSpec`), Messages 입력/dict 출력 등 입출력 형태별 설정, Redis 영구 저장(`RedisChatMessageHistory`)
- 정의: `get_session_history`, `get_message_history`

#### [09-Custom-Generator.ipynb](13-LangChain-Expression-Language/09-Custom-Generator.ipynb)
스트리밍 출력을 받아 가공하는 제너레이터(쉼표 리스트를 항목별로 yield), 비동기 버전.
- 정의: `split_into_list`, `asplit_into_list`

#### [10-Binding.ipynb](13-LangChain-Expression-Language/10-Binding.ipynb)
`.bind()`로 실행 인자 고정(stop 토큰 등), OpenAI functions / tools 연결.

#### [11-Fallbacks.ipynb](13-LangChain-Expression-Language/11-Fallbacks.ipynb)
`with_fallbacks()` — API 오류(RateLimitError) 시 대체 모델, 특정 예외만 처리, 여러 모델 순차 대체, 체인 단위 fallback.

---

### 14-Chains — 체인

#### [01-Summary.ipynb](14-Chains/01-Summary.ipynb)
**문서 요약 방식 총정리.**
- 다루는 방식: Stuff(`create_stuff_documents_chain`), Map-Reduce, Map-Refine, Chain of Density, Clustering-Map-Refine(KMeans로 대표 청크만 요약, t-SNE 시각화)
- 정의: `map_reduce_chain`, `map_refine_chain`

#### [02-SQL.ipynb](14-Chains/02-SQL.ipynb)
자연어 → SQL.
- 다루는 내용: `create_sql_query_chain`, `QuerySQLDataBaseTool`로 실행, 결과를 LLM이 답변으로 생성, `create_sql_agent`
- 핵심 API: `SQLDatabase.from_uri`

#### [03-Structured-Output-Chain.ipynb](14-Chains/03-Structured-Output-Chain.ipynb)
`with_structured_output()`으로 퀴즈(문제·보기·정답) 생성.
- 정의: `Quiz`

#### [04-Structured-Data-Chat.ipynb](14-Chains/04-Structured-Data-Chat.ipynb)
Pandas DataFrame 에이전트(타이타닉 데이터), 2개 이상 DataFrame 비교 질의.
- 핵심 API: `create_pandas_dataframe_agent`, `PythonAstREPLTool`
- 정의: `StreamCallback`

---

### 15-Agent — 에이전트

#### [01-Tools.ipynb](15-Agent/01-Tools.ipynb)
도구 모음.
- 내장 도구: `PythonREPLTool`(LLM이 짠 코드 실행), `TavilySearchResults`(웹 검색), `DallEAPIWrapper`(이미지 생성)
- 커스텀 도구: `@tool` 데코레이터, 구글 뉴스 검색 도구
- 정의: `print_and_execute`, `add_numbers`, `multiply_numbers`, `search_keyword`

#### [02-Bind-Tools.ipynb](15-Agent/02-Bind-Tools.ipynb)
`llm.bind_tools()`로 도구 호출 → `JsonOutputToolsParser`로 파싱 → 직접 실행, 이후 Agent로 대체.
- 정의: `get_word_length`, `add_function`, `naver_news_crawl`(네이버 뉴스 크롤링), `execute_tool_calls`

#### [03-Agent.ipynb](15-Agent/03-Agent.ipynb)
**Tool Calling Agent 기본 템플릿.**
- 다루는 내용: Agent 프롬프트(`agent_scratchpad`), `create_tool_calling_agent`, `AgentExecutor`(max_iterations 등), 스트리밍으로 중간 단계 확인, 콜백으로 중간 단계 출력, 대화 기록 유지 Agent
- 정의: `search_news`, `python_repl_tool`, `tool_callback`, `observation_callback`, `result_callback`, `get_session_history`

#### [04-Agent-More-LLMs.ipynb](15-Agent/04-Agent-More-LLMs.ipynb)
같은 에이전트를 Claude, Gemini, Ollama 등 여러 LLM으로 실행·비교.
- 정의: `search_news`, `execute_agent`

#### [05-Iter-Human-In-the-Loop.ipynb](15-Agent/05-Iter-Human-In-the-Loop.ipynb)
`AgentExecutor.iter()`로 단계별 실행하며 사용자 승인 받기.
- 정의: `add_function`

#### [06-Agentic-RAG.ipynb](15-Agent/06-Agentic-RAG.ipynb)
웹 검색 도구 + PDF 검색 도구(`create_retriever_tool`)를 가진 에이전트, 대화 기록 유지, 재사용 템플릿.
- 정의: `get_session_history`

#### [07-CSV-Excel-Agent.ipynb](15-Agent/07-CSV-Excel-Agent.ipynb)
CSV/Excel 데이터 분석 에이전트(시각화 포함), 스트리밍 콜백.
- 정의: `tool_callback`, `observation_callback`, `result_callback`, `ask`

#### [08-Agent-Toolkits-File-Management.ipynb](15-Agent/08-Agent-Toolkits-File-Management.ipynb)
`FileManagementToolkit`(파일 읽기·쓰기·목록·복사·삭제)으로 뉴스를 검색해 파일로 저장하는 에이전트.
- 정의: `latest_news`, `get_session_history`

#### [09-Agent-Report-With-Image-Generation.ipynb](15-Agent/09-Agent-Report-With-Image-Generation.ipynb)
웹 검색 + PDF 검색 + DALL-E + 파일 관리 도구로 이미지가 포함된 보고서(마크다운) 작성.
- 정의: `dalle_tool`, `get_session_history`

#### [10-Two-Agent-Debate-With-Tools.ipynb](15-Agent/10-Two-Agent-Debate-With-Tools.ipynb)
도구를 쓰는 두 에이전트의 찬반 토론 시뮬레이션.
- 정의: `DialogueAgent`, `DialogueSimulator`, `DialogueAgentWithTools`, `generate_agent_description`, `generate_system_message`, `select_next_speaker`

#### [12-React-Agent.ipynb](15-Agent/12-React-Agent.ipynb)
LangGraph의 `create_react_agent` + `MemorySaver`로 ReAct 에이전트(웹 검색, 파일 관리, PDF 검색 도구).
- 핵심 API: `create_react_agent`, `MemorySaver`, teddynote `stream_graph`, `visualize_graph`

---

### 16-Evaluations — 평가

> 04~14번은 [16-Evaluations/myrag.py](16-Evaluations/myrag.py)의 `PDFRAG` 클래스를 평가 대상으로 사용합니다.

#### [01-Test-Dataset-Generator-RAGAS.ipynb](16-Evaluations/01-Test-Dataset-Generator-RAGAS.ipynb)
RAGAS로 PDF에서 합성 테스트셋(질문·정답·컨텍스트) 생성, 질문 유형 분포(simple, reasoning, multi_context, conditional) 지정.

#### [02-Evaluation-Using-RAGAS.ipynb](16-Evaluations/02-Evaluation-Using-RAGAS.ipynb)
RAGAS 지표로 RAG 평가: Context Recall, Context Precision, Answer Relevancy, Faithfulness.
- 정의: `convert_to_list`

#### [03-Translate-HF-Upload.ipynb](16-Evaluations/03-Translate-HF-Upload.ipynb)
DeepL로 데이터셋 번역, HuggingFace Hub에 데이터셋 업로드.

#### [04-LangSmith-Dataset.ipynb](16-Evaluations/04-LangSmith-Dataset.ipynb)
LangSmith 평가용 데이터셋 생성(`Client.create_dataset`, `create_examples`).

#### [05-LangSmith-LLM-as-Judge.ipynb](16-Evaluations/05-LangSmith-LLM-as-Judge.ipynb)
LLM 평가자: `qa`, `context_qa`, `cot_qa`, Criteria(간결성 등), `labeled_criteria`, `labeled_score_string`(점수형).
- 정의: `ask_question`, `print_evaluator_prompt`, `context_answer_rag_answer`

#### [06-LangSmith-Embedding-Distance-Evaluation.ipynb](16-Evaluations/06-LangSmith-Embedding-Distance-Evaluation.ipynb)
정답과 답변의 임베딩 거리로 평가(여러 임베딩 모델 비교).

#### [07-LangSmith-Custom-LLM-Evaluation.ipynb](16-Evaluations/07-LangSmith-Custom-LLM-Evaluation.ipynb)
사용자 정의 Evaluator 함수(`Run`, `Example` 입력), 커스텀 LLM-as-Judge.
- 정의: `random_score_evaluator`, `custom_evaluator`, `context_answer_rag_answer`

#### [08-LangSmith-Heuristic-Evaluation.ipynb](16-Evaluations/08-LangSmith-Heuristic-Evaluation.ipynb)
LLM 없는 휴리스틱 평가: ROUGE, BLEU, METEOR, SemScore(한국어 형태소 Kiwi 토크나이저 활용).
- 정의: `rouge_evaluator`, `bleu_evaluator`, `meteor_evaluator`, `semscore_evaluator`

#### [09-LangSmith-Compare-Evaluation.ipynb](16-Evaluations/09-LangSmith-Compare-Evaluation.ipynb)
GPT vs Ollama 모델 실험 결과 비교.
- 정의: `ask_question_with_llm`

#### [10-LangSmith-Summary-Evaluation.ipynb](16-Evaluations/10-LangSmith-Summary-Evaluation.ipynb)
데이터셋 전체를 종합하는 Summary Evaluator(관련성 평가 집계).
- 정의: `relevance_score_summary_evaluator`

#### [11-LangSmith-Groundedness-Evaluation.ipynb](16-Evaluations/11-LangSmith-Groundedness-Evaluation.ipynb)
답변이 컨텍스트에 근거하는지(할루시네이션) 평가 — `UpstageGroundednessCheck`, teddynote `GroundnessChecker`, Summary 평가.

#### [12-LangSmith-Pairwise-Evaluation.ipynb](16-Evaluations/12-LangSmith-Pairwise-Evaluation.ipynb)
두 실험 결과를 LLM이 쌍으로 비교(`evaluate_comparative`).
- 정의: `evaluate_pairwise`

#### [13-LangSmith-Repeat-Evaluation.ipynb](16-Evaluations/13-LangSmith-Repeat-Evaluation.ipynb)
`num_repetitions`로 반복 평가해 편차 확인(GPT / Ollama).

#### [14-LangSmith-Online-Evaluation.ipynb](16-Evaluations/14-LangSmith-Online-Evaluation.ipynb)
LangSmith 온라인 평가(운영 중 실행 추적 자동 평가), 태그 설정.

---

### 17-LangGraph — 01-Core-Features (핵심 기능)

#### [01-LangGraph-Introduction.ipynb](17-LangGraph/01-Core-Features/01-LangGraph-Introduction.ipynb)
LangGraph에 필요한 Python 문법: `TypedDict`, `Annotated`, Pydantic 검증, `add_messages` 리듀서.

#### [02-LangGraph-ChatBot.ipynb](17-LangGraph/01-Core-Features/02-LangGraph-ChatBot.ipynb)
**LangGraph 기본 7단계**: State 정의 → Node 정의 → Graph에 노드 추가 → Edge 추가 → compile → 시각화 → 실행. 전체 코드 포함.
- 정의: `State`, `chatbot`

#### [03-LangGraph-Agent.ipynb](17-LangGraph/01-Core-Features/03-LangGraph-Agent.ipynb)
도구를 쓰는 에이전트 그래프, ToolNode 직접 구현, 조건부 엣지(`add_conditional_edges`).
- 정의: `BasicToolNode`(도구 노드 직접 구현), `route_tools`

#### [04-LangGraph-Agent-With-Memory.ipynb](17-LangGraph/01-Core-Features/04-LangGraph-Agent-With-Memory.ipynb)
`MemorySaver` 체크포인터로 `thread_id`별 대화 기억, `get_state()`로 스냅샷 확인.
- 핵심 API: `ToolNode`, `tools_condition`, `MemorySaver`, `RunnableConfig`

#### [05-LangGraph-Streaming-Outputs.ipynb](17-LangGraph/01-Core-Features/05-LangGraph-Streaming-Outputs.ipynb)
`graph.stream()` 옵션: `output_keys`, `stream_mode`, `interrupt_before` / `interrupt_after`.

#### [06-LangGraph-Human-In-the-Loop.ipynb](17-LangGraph/01-Core-Features/06-LangGraph-Human-In-the-Loop.ipynb)
도구 실행 전에 중단(interrupt)하고 상태를 확인한 뒤 이어서 실행, 상태 이력 조회.

#### [07-LangGraph-Manual-State-Update.ipynb](17-LangGraph/01-Core-Features/07-LangGraph-Manual-State-Update.ipynb)
`update_state()`로 중간 상태 수정(도구 호출 내용 수정, 메시지 교체), 과거 스냅샷에서 수정 후 재실행(Replay).

#### [08-LangGraph-State-Customization.ipynb](17-LangGraph/01-Core-Features/08-LangGraph-State-Customization.ipynb)
State에 커스텀 필드(`ask_human`) 추가, 사람에게 도움을 요청하는 노드 구성.
- 정의: `HumanRequest`, `human_node`, `select_next_node`, `create_response`

#### [09-LangGraph-DeleteMessages.ipynb](17-LangGraph/01-Core-Features/09-LangGraph-DeleteMessages.ipynb)
`RemoveMessage`로 메시지 삭제(수동 / 노드에서 오래된 메시지 자동 삭제).
- 정의: `delete_messages`, `should_continue`, `call_model`

#### [10-LangGraph-ToolNode.ipynb](17-LangGraph/01-Core-Features/10-LangGraph-ToolNode.ipynb)
`ToolNode` 수동 호출, LLM과 결합, 에이전트 그래프 구성(뉴스 검색 + Python 코드 실행 도구).
- 정의: `search_news`, `python_code_interpreter`

#### [11-LangGraph-Branching.ipynb](17-LangGraph/01-Core-Features/11-LangGraph-Branching.ipynb)
병렬 노드 실행: fan-out / fan-in, 예외 처리, 조건부 분기, 신뢰도 기준 결과 정렬.
- 정의: `ReturnNodeValue`, `ParallelReturnNodeValue`, `route_bc_or_cd`, `reduce_fanouts`, `aggregate_fanout_values`

#### [12-LangGraph-Add-Conversation-Summary.ipynb](17-LangGraph/01-Core-Features/12-LangGraph-Add-Conversation-Summary.ipynb)
대화가 길어지면 요약을 생성하고 오래된 메시지를 삭제.
- 정의: `ask_llm`, `should_continue`, `summarize_conversation`, `print_update`

#### [13-LangGraph-Subgraph.ipynb](17-LangGraph/01-Core-Features/13-LangGraph-Subgraph.ipynb)
서브그래프를 노드로 추가 — 스키마 키를 공유하는 경우 / 공유하지 않는 경우(함수로 감싸 변환).

#### [14-LangGraph-Subgraph-Transform-State.ipynb](17-LangGraph/01-Core-Features/14-LangGraph-Subgraph-Transform-State.ipynb)
parent → child → grandchild 3단계 그래프에서 입력·출력 상태 변환.

#### [15-LangGraph-Streaming-Steps.ipynb](17-LangGraph/01-Core-Features/15-LangGraph-Streaming-Steps.ipynb)
스트리밍 모드 상세: `values`, `updates`, `messages`(토큰 단위), 특정 노드만 스트리밍, tag 필터링, 도구 호출 스트리밍, 서브그래프 출력 포함/생략.
- 정의: `create_sns_post`, `create_subgraph`, `format_namespace`, `parse_namespace_info`

---

### 17-LangGraph — 02-Structures (RAG 그래프 구조)

> 02~07번은 [17-LangGraph/02-Structures/rag/pdf.py](17-LangGraph/02-Structures/rag/pdf.py)의 `PDFRetrievalChain`과 [rag/utils.py](17-LangGraph/02-Structures/rag/utils.py)의 `format_docs`를 사용합니다. 번호 순서대로 기능이 하나씩 추가됩니다.

#### [01-LangGraph-Building-Graphs.ipynb](17-LangGraph/02-Structures/01-LangGraph-Building-Graphs.ipynb)
그래프 설계 예시 모음(노드는 빈 함수로 구조만): 검색 → 쿼리 재작성 → GPT/Claude 병렬 실행 → 관련성 체크 → 웹 검색, SQL RAG 구조.

#### [02-LangGraph-Naive-RAG.ipynb](17-LangGraph/02-Structures/02-LangGraph-Naive-RAG.ipynb)
기본 RAG 그래프: 검색 노드 → 답변 노드.
- 정의: `GraphState`, `retrieve_document`, `llm_answer`

#### [03-LangGraph-Add-Groundedness-Check.ipynb](17-LangGraph/02-Structures/03-LangGraph-Add-Groundedness-Check.ipynb)
검색 문서 관련성 체크 노드 추가(관련 없으면 재검색), `GraphRecursionError` 처리.
- 정의: `relevance_check`, `is_relevant`

#### [04-LangGraph-Add-Web-Search.ipynb](17-LangGraph/02-Structures/04-LangGraph-Add-Web-Search.ipynb)
관련 문서가 없으면 Tavily 웹 검색으로 대체.
- 정의: `web_search`

#### [05-LangGraph-Add-Query-Rewrite.ipynb](17-LangGraph/02-Structures/05-LangGraph-Add-Query-Rewrite.ipynb)
질문 재작성 노드 추가.
- 정의: `query_rewrite`

#### [06-LangGraph-Agentic-RAG.ipynb](17-LangGraph/02-Structures/06-LangGraph-Agentic-RAG.ipynb)
에이전트가 검색 도구 사용 여부를 스스로 결정, 문서 평가(yes/no) 후 재작성 또는 답변 생성.
- 정의: `AgentState`, `grade`, `grade_documents`, `agent`, `rewrite`, `generate`

#### [07-LangGraph-Adaptive-RAG.ipynb](17-LangGraph/02-Structures/07-LangGraph-Adaptive-RAG.ipynb)
질문 라우팅(벡터스토어 vs 웹 검색) + 문서 평가 + 할루시네이션 체크 + 답변 적합성 체크 + 쿼리 재작성.
- 정의: `RouteQuery`, `GradeDocuments`, `GradeHallucinations`, `GradeAnswer`(평가용 Pydantic 모델), `route_question`, `decide_to_generate`, `hallucination_check`, `transform_query`, `web_search` 외

---

### 17-LangGraph — 03-Use-Cases (응용 사례)

#### [01-LangGraph-Agent-Simulation.ipynb](17-LangGraph/03-Use-Cases/01-LangGraph-Agent-Simulation.ipynb)
상담사 챗봇과 가상 고객(LLM)의 대화 시뮬레이션(챗봇 테스트용).
- 정의: `call_chatbot`, `create_scenario`, `ai_assistant_node`, `_swap_roles`, `simulated_user_node`, `should_continue`

#### [02-LangGraph-Prompt-Generation.ipynb](17-LangGraph/03-Use-Cases/02-LangGraph-Prompt-Generation.ipynb)
사용자와 대화하며 요구사항을 수집한 뒤 프롬프트를 자동 생성하는 메타 프롬프트 에이전트.
- 정의: `PromptInstructions`, `info_chain`, `prompt_gen_chain`, `get_state`, `add_tool_message`

#### [03-LangGraph-CRAG.ipynb](17-LangGraph/03-Use-Cases/03-LangGraph-CRAG.ipynb)
Corrective RAG: 검색 문서 평가 → 부족하면 쿼리 재작성 + 웹 검색으로 보완.
- 정의: `GradeDocuments`, `retrieve`, `generate`, `grade_documents`, `query_rewrite`, `web_search`, `decide_to_generate`

#### [04-LangGraph-Self-RAG.ipynb](17-LangGraph/03-Use-Cases/04-LangGraph-Self-RAG.ipynb)
Self-RAG: 문서 평가, 할루시네이션 평가, 답변 관련성 평가, 질문 재작성 루프.
- 정의: `GradeDocuments`, `Groundednesss`, `GradeAnswer`, `grade_generation_v_documents_and_question` 외

#### [05-LangGraph-Plan-and-Execute.ipynb](17-LangGraph/03-Use-Cases/05-LangGraph-Plan-and-Execute.ipynb)
계획 수립 → 단계별 실행(ReAct 에이전트) → 재계획 → 최종 보고서.
- 정의: `PlanExecute`, `Plan`, `Response`, `Act`, `plan_step`, `execute_step`, `replan_step`, `should_end`, `generate_final_report`

#### [06-LangGraph-Multi-Agent-Collaboration.ipynb](17-LangGraph/03-Use-Cases/06-LangGraph-Multi-Agent-Collaboration.ipynb)
연구 에이전트(웹 검색) + 차트 생성 에이전트(Python REPL)의 협업 네트워크.
- 정의: `AgentState`, `python_repl_tool`, `make_system_prompt`, `research_node`, `chart_node`, `router`

#### [06-LangGraph-Multi-Agent-Collaboration-llama3.ipynb](17-LangGraph/03-Use-Cases/06-LangGraph-Multi-Agent-Collaboration-llama3.ipynb)
위와 같은 구조를 Together AI의 Llama3로 실행.

#### [07-LangGraph-Multi-Agent-Supervisor.ipynb](17-LangGraph/03-Use-Cases/07-LangGraph-Multi-Agent-Supervisor.ipynb)
감독자(Supervisor) LLM이 다음 작업자 에이전트를 선택하는 구조.
- 정의: `agent_node`, `RouteResponse`, `supervisor_agent`, `get_next`

#### [08-LangGraph-Hierarchial-Agent-Team.ipynb](17-LangGraph/03-Use-Cases/08-LangGraph-Hierarchial-Agent-Team.ipynb)
계층형 팀: 연구 팀(검색·스크래핑) + 문서 작성 팀(개요·작성·편집·차트)을 상위 감독자가 관리.
- 정의: 도구 `scrape_webpages`, `create_outline`, `read_document`, `write_document`, `edit_document` / 유틸 `AgentFactory`, `create_team_supervisor`, `get_last_message`, `join_graph`

#### [09-LangGraph-SQL-Agent.ipynb](17-LangGraph/03-Use-Cases/09-LangGraph-SQL-Agent.ipynb)
SQL 에이전트: 테이블 조회 → 스키마 확인 → 쿼리 생성 → 쿼리 점검 → 실행 → 오류 시 재시도, LangSmith로 평가.
- 정의: `handle_tool_error`, `create_tool_node_with_fallback`(도구 오류 fallback), `db_query_tool`, `model_check_query`, `SubmitFinalAnswer`, `query_gen_node`, `answer_evaluator`

#### [10-LangGraph-Research-Assistant.ipynb](17-LangGraph/03-Use-Cases/10-LangGraph-Research-Assistant.ipynb)
STORM 방식 연구 보조: 분석가 페르소나 생성(Human-in-the-loop) → 분석가별 인터뷰(웹·arXiv 검색) 병렬 실행(`Send`, map-reduce) → 섹션 작성 → 서론/결론 → 최종 보고서.
- 정의: `Analyst`, `Perspectives`, `create_analysts`, `human_feedback`, `generate_question`, `search_web`, `search_arxiv`, `generate_answer`, `save_interview`, `write_section`, `initiate_all_interviews`, `write_report`, `write_introduction`, `write_conclusion`, `finalize_report`

---

### 18-FineTuning — 파인튜닝

#### [01-QA-Pair-GPT.ipynb](18-FineTuning/01-QA-Pair-GPT.ipynb)
PDF에서 GPT로 QA 쌍 생성 → jsonl 저장 → HuggingFace 업로드.
- 정의: `extract_pdf_elements`, `custom_json_parser`

#### [02_Unsloth_llama3_파인튜닝_alpaca.ipynb](18-FineTuning/02_Unsloth_llama3_파인튜닝_alpaca.ipynb)
Unsloth로 Llama3 LoRA 파인튜닝(Colab): 데이터 준비(Alpaca 형식), `SFTTrainer` 학습, 추론, float16/GGUF 변환 후 로컬 저장·HF 업로드.
- 정의: `formatting_prompts_func`, `StopOnToken`

---

### 19-Streamlit / 20-Projects / 22-OpenAI / 99-Projects

#### [19-Streamlit/03-RAG-With-Evaluation/rag-with-ragas.ipynb](19-Streamlit/03-RAG-With-Evaluation/rag-with-ragas.ipynb)
PDF RAG 답변을 RAGAS(answer_relevancy, faithfulness)로 즉석 평가.
- 정의: `RagEvaluator`, `ask`
- 참고: 같은 폴더의 `main.py`는 Streamlit 앱, [19-Streamlit/01-MyProject/pages/](19-Streamlit/01-MyProject/pages/)에 PDF RAG·로컬 RAG·멀티모달·멀티턴·CSV 에이전트·ReAct 에이전트 Streamlit 예제(.py)가 있습니다.

#### [20-Projects/01-ParsingOutput/01-concept.ipynb](20-Projects/01-ParsingOutput/01-concept.ipynb)
이메일 본문을 `PydanticOutputParser`로 구조화 → SERP API로 관련 정보 검색 → 답장 초안 생성.
- 정의: `EmailSummary`

#### [22-OpenAI/01-OpenAI-Assistant-V2.ipynb](22-OpenAI/01-OpenAI-Assistant-V2.ipynb)
OpenAI Assistants API v2(파일 검색)로 RAG: 파일 업로드, Assistant·VectorStore 생성/재사용, 대화 — teddynote `OpenAIAssistant`.

#### [99-Projects/01-YouTube-Transcript-Summarize/youtube-qa-bot.ipynb](99-Projects/01-YouTube-Transcript-Summarize/youtube-qa-bot.ipynb)
YouTube 오디오 다운로드 → Whisper로 텍스트 변환 → FAISS → `RetrievalQA` 질의응답.
- 핵심 API: `YoutubeAudioLoader`, `GenericLoader`, `OpenAIWhisperParser`
- 정의: `StreamCallback`

#### [99-Projects/02-Review-Tagger/Metadata-Tagger.ipynb](99-Projects/02-Review-Tagger/Metadata-Tagger.ipynb)
CSV 리뷰 데이터에 LLM으로 메타데이터(키워드, 디자인, 만족/개선점, 긍·부정 톤, 평점) 자동 태깅 — `create_metadata_tagger`.
- 정의: `Properties`(태깅 스키마)

---

## 참고: 노트북이 import하는 로컬 헬퍼 모듈

| 모듈 | 내용 | 사용하는 곳 |
|---|---|---|
| [16-Evaluations/myrag.py](16-Evaluations/myrag.py) | `PDFRAG` — PDF 로드·분할·FAISS·체인 생성 클래스 | 16-Evaluations 05~14 |
| [17-LangGraph/02-Structures/rag/](17-LangGraph/02-Structures/rag/) (`base.py`, `pdf.py`, `utils.py`) | `PDFRetrievalChain` — RAG 체인 기본 클래스, `format_docs` | 17-LangGraph 02-Structures, 03-Use-Cases, 19-Streamlit/03 |
| [19-Streamlit/01-MyProject/](19-Streamlit/01-MyProject/) (`retriever.py`, `react_agent.py`, `stream_handler.py` 등) | Streamlit 앱용 검색기·에이전트·스트리밍 헬퍼 | Streamlit 예제 |
