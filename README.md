# Inventory — 외식업 재고·매출 운영 자동화 플랫폼

> 입고·재고·주문·매출 분석·리포트를 하나의 흐름으로 처리하는 외식업 운영 자동화 플랫폼

💻 **Frontend Repository**: [Inventory-FE](https://github.com/yerincho94/Inventory-FE)

<br/>

## 프로젝트 소개

외식업 매장의 운영을 하나의 시스템에서 처리할 수 있도록 만든 재고·매출 관리 플랫폼입니다.
로그인부터 기준 정보 관리, 입고, 재고 관리, 주문, 매출 분석, 리포트까지 매장 운영의 전 과정을 한 흐름 안에서 다루며, 알림과 챗봇이 필요한 정보를 빠르게 확인할 수 있도록 지원합니다.

- **개발 기간**: 2026.02.02 ~ 2026.03.23
- **팀 구성**: 백엔드 4명 (프로그래머스 데브코스 최종 프로젝트)

<br/>

## 기술 스택

### Backend
![Java](https://img.shields.io/badge/Java%2017-007396?style=flat-square&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white)
![Spring Data JPA](https://img.shields.io/badge/Spring%20Data%20JPA-6DB33F?style=flat-square&logo=spring&logoColor=white)
![QueryDSL](https://img.shields.io/badge/QueryDSL-0769AD?style=flat-square)
![Spring Security](https://img.shields.io/badge/Spring%20Security-6DB33F?style=flat-square&logo=springsecurity&logoColor=white)

### Database & Search
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![Elasticsearch](https://img.shields.io/badge/Elasticsearch%208.x-005571?style=flat-square&logo=elasticsearch&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white)

### Infra & Tools
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=flat-square&logo=nginx&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat-square&logo=amazonaws&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=flat-square&logo=jenkins&logoColor=white)
![Swagger](https://img.shields.io/badge/Swagger-85EA2D?style=flat-square&logo=swagger&logoColor=black)

<br/>

## 주요 기능

| 도메인 | 기능 |
|---|---|
| 기준 정보 | 매장·식자재·메뉴·거래처(Vendor) 관리 |
| 입고 / 재고 | OCR 기반 입고 처리, 재고 차감, 폐기 관리 |
| **주문** | 주문 등록 및 재고 연동 차감 |
| **매출 분석** | **Elasticsearch 집계 기반 대시보드 — 피크타임 히트맵·메뉴 랭킹·매출 추이·전월 대비 성장률(MTD)** |
| **리포트** | **월간/기간 리포트 자동 생성(스케줄러) 및 PDF 다운로드** |
| **발주** | **거래처 기반 발주 관리** |
| 알림 / 챗봇 | 운영 정보 알림 및 챗봇 안내 |

<br/>

## 담당 역할 (조예린)

> 본 프로젝트에서 **Elasticsearch 기반 매출 분석 · 리포트 · 발주** 영역을 담당했습니다.

- **ES 집계 매출 분석 대시보드** — 요일×시간대 피크타임 히트맵, 메뉴 랭킹, 매출 추이, 전월 대비 성장률(MTD)을 Elasticsearch 집계로 구현, **0.5초 내 응답** 달성
- **매출 상세 집계** — 환불 요약·메뉴별 매출을 `terms` / `nested aggregation`으로 구현, 조회 시 **N+1 제거**
- **월간/기간 리포트** — 매월 1일 자동 실행 스케줄러로 알림 발행 + PDF 생성, QueryDSL `fetch join`으로 N+1 해결
- **실시간 매출 대시보드(FE 연동)** — 30초 폴링 기반 실시간 매출, 시간대별 추이 차트
- **발주** — 거래처(Vendor) 기반 발주 기능
- **주문** - 테이블 QR 스캔 → 메뉴 선택 → 주문 → 재고 자동 차감 구현, 중복 결제 방지(Idempotency Key), 재고 부족 시 예외 발생 → 자동 롤백

<br/>

## 시스템 아키텍처

<p align="center">
  <img width="5434" height="2179" alt="최종 소프트웨어 아키텍처" src="https://github.com/user-attachments/assets/5ce192a6-dfab-4b16-8167-d0f2836ef429" />
</p>

<br/>

## ERD

<p align="center">
  <img width="5030" height="3222" alt="인벤토리(Inventory)-ERD" src="https://github.com/user-attachments/assets/d4ee55ee-c5b5-4dbd-9afc-7c3c519fe9ca" />
  [ERDCloud 바로가기](https://www.erdcloud.com/d/eoKCCWehxiZJTKp7J)
</p>

<br/>

## 주요 트러블슈팅 — 타임존 불일치로 인한 매출 집계 왜곡

| 구분 | 내용 |
|---|---|
| **문제** | 야간(밤 9시 이후) 주문이 다음 날짜로 잘못 집계되어 매출 통계가 왜곡됨 |
| **원인** | PostgreSQL은 UTC, Elasticsearch는 KST(9시간 차) 기준으로 데이터를 처리 → 저장소 간 시간 기준 불일치 |
| **해결** | 데이터 일부만 보지 않고 두 저장소의 시간 처리 경로를 추적, 쿼리 전달 단계에서 **UTC로 명시 변환**해 기준 통일 |
| **결과** | 날짜별 매출 집계 정확성 확보. 겉으로 드러난 증상보다 데이터가 흐르는 경로 전체를 점검하는 것이 해결의 출발점임을 학습 |

<br/>

## 프로젝트 구조

```
src/main/java/com/inventory
├── domain          # 도메인 엔티티 (Ingredient, Vendor, Order, Sales ...)
├── repository      # JPA / QueryDSL / Elasticsearch Repository
├── service         # 비즈니스 로직 (SalesAnalyticsService, ReportPdfService ...)
├── controller      # REST API
├── scheduler       # 월간 리포트 스케줄러
└── global          # 공통 설정, 예외 처리, 보안
```

<br/>

## 로컬 실행 방법

## Local Run (Docker Compose + Nginx)

### 사전 준비

- Docker Desktop 실행 중
- FE/BE 레포가 같은 부모 폴더 아래에 위치
```
Workspace/
├── AIBE4_FinalProject_Team1_BE
└── AIBE4_FinalProject_Team1_FE
```

---

### Run
```bash
docker compose up -d --build
```

빌드 완료 후 아래 URL로 접속할 수 있습니다.

| Service | URL |
|--|-----|
| Web App | http://localhost |
| Swagger UI | http://localhost/swagger-ui/index.html |
| OpenAPI JSON | http://localhost/v3/api-docs |

---

### Stop

`Ctrl + C`로 로그 출력을 종료한 뒤 아래 명령어로 컨테이너를 중지합니다.
```bash
docker compose down
```

<br/>

## 팀원

| 이름 | 역할 |
|---|---|
| 김소명 (팀장) | `인프라 / 매장·초대 입고 / 정규화 / 알림 / 챗봇` |
| 서동환 | `OCR / 입고 / 폐기 Elasticsearch / 재고 분석` |
| 서영광 | `OAuth2 / JWT / 주문 / 차감 / 알림 / 챗봇` |
| **조예린** | **발주 / Elasticsearch 매출 분석 / 주문 / 리포트** |

<br/>

## 협업 컨벤션

### 커밋 메시지 유형

| 유형 | 의미 |
|------|------|
| `feat` | 새로운 기능 추가 |
| `fix` | 버그 수정 |
| `docs` | 문서 수정 |
| `style` | 코드 formatting, 세미콜론 누락 등 |
| `refactor` | 코드 리팩토링 |
| `test` | 테스트 코드 추가 |
| `chore` | 패키지 매니저 수정, 기타 수정 |
| `design` | CSS 등 UI 디자인 변경 |
| `comment` | 주석 추가 및 변경 |
| `rename` | 파일/폴더명 수정 또는 이동 |
| `remove` | 파일 삭제 |
| `!breaking change` | 커다란 API 변경 |
| `!hotfix` | 급한 버그 수정 |
| `assets` | 에셋 파일 추가 |

### 커밋 메시지 형식

**제목:**
```
type : 커밋메시지
```

**내용:**
```markdown
### 작업 내용
- 작업 내용 1
- 작업 내용 2
- 작업 내용 3
```
