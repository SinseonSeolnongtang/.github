<div align="center">

# 🍽️ Findish

**탐색부터 예약·주문까지, 모든 식당 선택을 한 곳에서 끝내는 개인화·그룹화 AI 다이닝 플랫폼**

🥇 **Best Project Award** · 2026 Hansung University Capstone Design Exhibition

![LangGraph](https://img.shields.io/badge/LangGraph-1C3C3C?style=flat-square&logo=langchain&logoColor=white)
![MCP](https://img.shields.io/badge/MCP-4A90E2?style=flat-square&logo=protocol&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI-412991?style=flat-square&logo=openai&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=black)
![PostgreSQL](https://img.shields.io/badge/pgvector-336791?style=flat-square&logo=postgresql&logoColor=white)

</div>

---

## 📌 Overview

한 끼를 정하기 위해 지도·리뷰·예약·배달 앱을 오가며 정보를 직접 종합해야 하는 현실에서 출발했습니다.
식당 한 곳에 쌓인 수백 개의 리뷰, 모두에게 동일한 별점순 추천, 불투명한 추천 근거, 그리고 흩어진 사용자 여정을
하나의 플랫폼에서 해결하고자 했습니다.

Findish는 **수만 건의 리뷰를 6대 관점으로 정량화한 추천 엔진**과
**자연어 명령을 실제 행동까지 옮기는 대화형 다이닝 에이전트**를 결합하여,
'무엇을·누구와·어떻게 먹을지' 고민하는 순간부터 실제 주문에 이르기까지의 의사결정을 지원합니다.

<br/>

**Goals**

| | |
| --- | --- |
| 🎯 **개인화·그룹화 추천** | 6측면 취향 벡터와 '좋아요' 학습 이력 기반, 개인 및 그룹 최적 추천 |
| 🗺️ **통합 탐색과 정량 비교** | 지도·목록 탐색 + 관점별 정량 비교(레이더 차트·Trade-off) |
| 💬 **대화형 행동 대행** | 탐색·조회부터 예약·장바구니·주문까지 자연어 명령으로 수행 |
| 🔍 **설명 가능한 추천 (XAI)** | 추천 근거를 문장·키워드·6각형 그래프와 페르소나로 제시 |

---

## 🏗️ Architecture

프론트엔드 · 백엔드 · AI 추천 서비스 · 다이닝 에이전트 · 데이터 파이프라인의 **5개 구성요소**로 이루어진 3-Tier 구조입니다.

```
Frontend (React)
     │  REST API
     ▼
Backend (Spring Boot)  ──JWT(HS384)──▶  AI Service (FastAPI + LangGraph)
     │                                          │
     │                                  AI Agent (LangGraph)
     │                                          │
     │                                    MCP Server ──▶ External Tools
     ▼                                          ▼
        PostgreSQL + pgvector  ·  Redis
```

- 백엔드가 발급한 **JWT(HS384)** 를 AI 서버가 검증하는 내부 인증 사용
- AI 서버는 상태를 갖지 않는(**stateless**) 마이크로서비스로 동작
- 에이전트의 모든 외부 도구는 **MCP 서버**를 통해 표준 프로토콜로 노출
- 데이터 계층은 **PostgreSQL(+pgvector)** 중심으로 임베딩 벡터와 도메인 데이터 저장

---

## ⚙️ Core Components

<details open>
<summary><b>데이터 파이프라인 — 비정형 리뷰의 정량화</b></summary>

<br/>

원본 리뷰를 **83,310개 문장**으로 분해·정제하고, 각 문장에 6대 관점(맛·분위기·서비스·가격·시설·웨이팅)과 긍정/부정 감정을 태깅했습니다.

- 빈도 분석으로 **4,841개 핵심 키워드**와 **1,182개 '가게 시그니처'** 도출
- **SigLIP2** 멀티모달 모델로 요약 텍스트와 리뷰 사진을 동일한 768차원 공간에 인코딩해 관점별 대표 이미지 매칭
- 논리적 검색(메뉴·자연어·개인화)은 **1536차원 텍스트 임베딩**, 대표 이미지 추출은 **768차원 멀티모달 공간**에서 처리

</details>

<details open>
<summary><b>6측면 크로스 임베딩 & 실시간 개인화</b></summary>

<br/>

하나의 1536차원 6측면 임베딩을 **'무엇과 무엇을 비교하느냐'에 따라 세 기능으로 재사용**하는 것이 추천 엔진의 핵심입니다.

| 비교 대상 | 기능 |
| --- | --- |
| 식당 ↔ 취향 | AI Pick (맞춤 추천) |
| 자연어 쿼리 ↔ 식당 | 의미 기반 검색 |
| 식당 ↔ 식당 | 비교 분석 |

- '좋아요' 시 **지수이동평균(EMA, α 0.6→0.3)** 으로 취향 벡터 실시간 갱신
- 코사인 유사도로 상위 10개를 추린 뒤 **LLM 리랭킹**으로 최종 Top-3 선별
- 리뷰의 긍정 편향은 **관점별 Z-score 정규화**로 보정

</details>

<details open>
<summary><b>다이닝 에이전트 아키텍처</b></summary>

<br/>

LangGraph 기반 멀티 에이전트로, 오케스트레이터가 요청 의도를 분류해 **Discovery / Action / General** 중 하나로 라우팅합니다.

- **Discovery Agent** — ReAct 방식으로 탐색·메뉴·요약·리뷰·비교 도구를 자유롭게 연쇄 호출 (`tool_calls` 루프)
- **Action Agent** — 예약·주문·장바구니·취소를 `의도 분류 → 수집(collect) → 확인(confirm) → 실행(execute)`의 **HITL 단계**로 처리
- 단계 상태는 **대화 턴을 넘어 지속 유지**되며, 모든 외부 도구는 MCP 서버를 통해 호출
- `finalize` 단계에서 텍스트와 동적 카드 UI가 결합된 응답으로 조립

</details>

---

## ✨ Key Features

| | 기능 | 설명 |
| --- | --- | --- |
| ① | **AI Pick** | 2단계 recall→precision. 코사인 유사도로 후보를 추리고 LLM이 최적 3곳을 선별·이유 생성 |
| ② | **자연어 검색** | 질의를 조건(하드)·의미(소프트)·측면으로 분해하고 의도 유형에 맞춰 검색 라우팅 |
| ③ | **일반 검색** | 이름·메뉴·키워드를 SQL로 매칭하는 고속 검색 (LLM 미사용) |
| ④ | **비교 분석** | 식당 2~3곳의 측면별 긍·부정, 공통점·차이, 6각형 레이더, 유사도 행렬 산출 |
| ⑤ | **개인화 학습** | EMA로 취향 벡터 누적, 긍정 편향은 부정 강조·z-상대화로 보정, 3-레이더 시각화 |
| ⑥ | **미식 코드 (먹BTI)** | 6측면 중 상위 3개를 조합해 취향을 직관적 페르소나 라벨로 표현 |
| ⑦ | **설명 가능한 추천** | 임베딩 유사도를 키워드로 역추적해 사람이 읽는 추천 이유와 근거 키워드 생성 |
| ⑧ | **행동 대행** | 예약·장바구니·주문·취소를 HITL 흐름으로 안전하게 대행, 그룹은 최대 4인 취향 합산 |

---

## 🔌 AI API Endpoints

각 응답은 추천 근거(`reasonSummary`), 6각형 레이더(`aspectRadar`), 미식 코드, 개인화 점수 분해(`breakdown`) 등 **설명 가능한 필드**를 함께 반환합니다.

| Endpoint | Method | Description |
| --- | --- | --- |
| `/ai/v1/ai-pick` | POST | 개인화·그룹 추천 (2단계 recall→precision) |
| `/ai/v1/recommend` | POST | 자연어 검색 (의도 라우팅) |
| `/ai/v1/search/keyword` | POST | 키워드 일반 검색 (고속) |
| `/ai/v1/compare` | POST | 식당 2~3곳 측면 비교 분석 |
| `/ai/v1/preference/like` | POST | '좋아요' 기반 개인화 학습 (EMA) |
| `/ai/v1/preference/me` | GET | 본인 미식 코드·취향 레이더 조회 |

---

## 🚀 Engineering Highlights

**추천 엔진**
- 하나의 6측면 임베딩 공간을 비교 대상만 바꿔 추천·검색·비교 세 기능에 재사용하는 **크로스 임베딩 설계**
- 모든 LLM 단계에 **규칙 기반 결정적 폴백**을 두어 외부 API 장애에도 서비스 연속성 확보
- 기동 시 캐시 적재(warm-up)로 원격 DB 병목 해소 — **AI Pick 응답 58초 → 7초**
- 리뷰의 약 95%가 긍정인 편향 환경에서도 부정 강조·z-상대화로 **식당 간 변별력 확보**

**다이닝 에이전트**
- 오케스트레이터와 도메인 서브그래프를 분리해 관심사 명확화
- 비가역 작업을 '수집→확인→실행' HITL 흐름으로 안전하게 처리
- 모든 외부 도구를 **MCP 표준 프로토콜**로 노출해 에이전트와 백엔드의 독립성·재사용성 확보

**백엔드 · 프론트엔드**
- 도메인별 `controller · service · repository · dto` 계층 구조 일관성, 인증 주체 기반 소유권 검증
- 백엔드 DTO에 없는 필드가 있어도 파싱이 깨지지 않는 **drop-safe 응답 설계**
- LLM 출력까지 React 엘리먼트로 안전 렌더링해 **XSS 차단** (`dangerouslySetInnerHTML` 미사용)
- 단일 `refreshPromise` 공유로 동시 401 상황에서도 토큰 재발급이 한 번만 발생하도록 동시성 제어

---

## 🛠️ Tech Stack

| 구분 | 기술 |
| --- | --- |
| **Frontend** | React · TypeScript · TanStack Query · Axios |
| **Backend** | Spring Boot · Java · JWT(HS384) · PostgreSQL |
| **AI Service** | Python 3.12 · FastAPI · Uvicorn · LangGraph / LangChain |
| **LLM · Embedding** | OpenAI gpt-4o-mini · text-embedding-3-small (1536d) · bge-reranker |
| **Data · Multimodal** | SigLIP2 (768d) · kiwipiepy · 웹 크롤링·전처리 |
| **DB · Persistence** | PostgreSQL + pgvector · SQLAlchemy + psycopg3 |
| **AI Agent** | LangGraph · MCP (Model Context Protocol) · OpenAI |
| **Deploy** | AWS EC2 (Seoul) · systemd · GitHub |

---

## 👥 Team

**신선한신선설농탕** (W-21) · 지도교수 이청용

| 이름 | 역할 | 주요 담당 |
| --- | --- | --- |
| 이석우 | 팀장 · AI | 추천 엔진 총괄 — 6측면 크로스 임베딩, AI Pick, 코사인 필터 + LLM 리랭킹 파이프라인 |
| 김민서 | Frontend | 통합 탐색·선택 모드·AI Pick·에이전트 챗 UI, 토큰 인증 흐름, 동적 카드 렌더링 |
| 양다원 | Backend | Spring Boot REST API, JWT 인증, 예약·주문·장바구니·찜·친구 도메인, 외부 서비스 연동 |
| 홍성환 | AI Agent | LangGraph 멀티 에이전트, MCP 서버, Discovery/Action 에이전트, HITL 트랜잭션 흐름 |
| 유성재 | Data | 리뷰 수집·정제, 6관점·감정 태깅, 텍스트/멀티모달 임베딩, SigLIP2 대표 이미지 매칭 |

---

<div align="center">
<sub>2026 Hansung University Capstone Design</sub>
</div>
