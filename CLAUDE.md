# CLAUDE.md

이 문서는 AI 어시스턴트가 이 프로젝트에서 작업할 때 따라야 할 규칙과 컨텍스트를 정의합니다.

## 프로젝트 개요

FastAPI + SQLAlchemy 2.0 (async) + React 19 + Tailwind 4 기반 풀스택 웹 애플리케이션.
**"유지보수성 최우선"** 및 **"모듈화"**를 핵심 가치로 합니다.

## 기술 스택

- **백엔드**: FastAPI 0.109.0, SQLAlchemy 2.0.25 (async), Pydantic v2, Alembic, JWT (python-jose)
- **프론트엔드**: React 19, Vite, Tailwind CSS 4, Zustand, React Router DOM
- **DB**: PostgreSQL (asyncpg, Supabase)
- **Python**: 3.12+

## 프로젝트 구조

```
server/app/
├── core/           # 설정, DB, 미들웨어, 로깅, 의존성
├── shared/base/    # BaseService, BaseRepository, BaseCalculator, BaseFormatter
├── domain/         # 비즈니스 도메인 (도메인별 독립 폴더)
├── api/v1/         # API 엔드포인트 (Router)
└── examples/       # 샘플 도메인 참고용

client/src/
├── core/           # api client, hooks, ui 컴포넌트, layout, store, errors, loading
├── domains/        # 도메인별 기능 (components, pages, api, store, types)
├── App.tsx         # 루트 컴포넌트
└── main.tsx        # 진입점

DOC/                # ARCHITECTURE.md, DEVELOPMENT_GUIDE.md 등 프로젝트 문서
alembic/            # DB 마이그레이션
tests/              # unit/, integration/
```

## 절대 금지 (NEVER DO)

1. **아키텍처**: 레이어드 구조(`Router -> Service -> Repository`) 파괴, 도메인 간 내부 구현 직접 참조
2. **백엔드**: 비즈니스 로직에 절차지향 함수 사용(클래스 기반 필수), 직접 DB 쿼리(Service/Repo 필수), 타입 힌트 누락
3. **프론트**: 직접 `axios` 호출(`apiClient` 사용 필수), 인라인 스타일(`Tailwind` 사용), `any` 타입 사용
4. **데이터베이스**: DB 콘솔/GUI에서 직접 스키마 수정, 마이그레이션 파일(`alembic/versions/`) 수정(Append-only 필수)

## 계층별 책임

- **Router (API)**: HTTP 입출력 처리만 담당. 비즈니스 로직 금지
- **Service**: 도메인 로직 오케스트레이션 및 트랜잭션 관리. `BaseService` 상속
- **Repository**: DB 조회 및 데이터 소스 접근. `BaseRepository` 상속
- **Calculator/Formatter**: 순수 함수(Side-effect Zero) 기반 계산 및 응답 변환
- **도메인 간 통신**: 다른 도메인의 `Service`나 `Repository`를 통해서만 통신

## 코드 스타일

- **Python**: SQLAlchemy 2.0 비동기 패턴, Pydantic v2 `BaseModel` 필수, line-length 100
- **TypeScript**: React 19 패턴, `Zustand` 기반 도메인 상태 관리, `cn()` 유틸을 이용한 조건부 클래스 처리
- **로깅**: 모든 로그에 `request_id` 포함, 민감 정보(PW, 토큰 등) 로깅 절대 금지

## 코드 품질 커맨드

```bash
# 백엔드 포맷팅 및 린트
black server/ --line-length 100
isort server/ --profile black
ruff check server/
mypy server/

# 프론트엔드 린트
cd client && npm run lint

# 테스트
pytest tests/
```

## DB 스키마 변경 워크플로우 (Alembic)

스키마 변경 시 반드시 아래 절차를 따릅니다:
1. 변경 사항을 사용자에게 설명하고 승인 요청
2. `server/app/domain/{domain}/models/` 수정
3. `alembic revision --autogenerate -m "description"`으로 마이그레이션 파일 생성
4. 생성된 마이그레이션 파일 검토 보고 후 `alembic upgrade head` 안내

## 변경 제안 형식

아키텍처나 스키마 변경 시 반드시 다음 형식으로 제안:
- **현재 상황**: 현재 구현 상태
- **제안**: 변경 내용
- **영향 범위**: 영향받는 파일/도메인
- **리스크**: 잠재적 위험 요소

## 문서 동기화

기능 변경 시 관련 `DOC/` 내 가이드를 반드시 업데이트합니다.
상세 구현 예시는 프로젝트 내 기존 코드와 `DOC/` 폴더를 최우선 참고합니다.

## 새 도메인 추가 시

```bash
# 백엔드
mkdir -p server/app/domain/{domain_name}/{models,schemas,repositories,calculators,formatters}

# 프론트엔드
mkdir -p client/src/domains/{domain_name}/{components,pages}
```

기존 `server/app/examples/sample_domain/` 및 `client/src/domains/sample/`을 참고합니다.
