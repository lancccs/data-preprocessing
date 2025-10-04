## 2025년 2학기 기업연계 AI 캡스톤 디자인 프로젝트 - 한성대

2025.10.04 개발 내용


capstone25-t3-kch/
├─ .gitignore
├─ README.md
├─ package.json
├─ yarn.lock
├─ backend/
│  └─ api-server/
│     ├─ .env.example                         #업데이트
│     ├─ .python-version
│     ├─ README.md
│     ├─ main.py                              #업데이트
│     ├─ pyproject.toml
│     ├─ requirements.txt                     #업데이트
│     ├─ routes/                              #생성
│     │  └─ policies.py
│     └─ data/                                #생성
│        └─ zip_prefix_regions.csv            # 3자리 우편번호 → 지역 매핑
│     └─ jobs/                                #생성
│        ├─ __init__.py
│        └─ ontong/
│           ├─ __init__.py
│           ├─ fetch_and_clean.py             # 원천 수집/클린
│           ├─ preprocess.py                  # 전처리 파이프라인
│           ├─ quality.py                     # 품질 점검
│           ├─ storage_pg.py                  # PostgreSQL 저장 유틸
│           ├─ utils.py
│           └─ zipmap_prefix.py               # 3자리 우편번호 prefix 매핑 로직
├─ frontend/
│ 


1. 프로젝트/레포 세팅,
    - backend/api-server 백엔드 작업 폴더 정리
    -.env 구성 및 requirements.txt 업데이트
2. PostgreSQL 준비 & 연동,
    - 로컬 Postgres 설치·접속(5432) → youth_policy DB 생성
    -테이블 스키마 준비: policy_raw, policy_clean (+ 컬럼 보강: category_auto, region 등)
    -SQL 쿼리로 접속/건수 확인 (\c youth_policy, SELECT COUNT(*) ...)
3. 전처리 파이프라인 구축,
    - 온통청년 API 수집 + 전처리 : python -m jobs.ontong.fetch_and_clean
    -핵심 필드 정규화: 기간(period_start/end 문자열 YYYY-MM-DD), 금액, 대상, 신청방법
    -품질체크 로직(quality.py)로 누락/형식 검사 (FR-14~19 대응)
    - 3-1. 지역 매핑(주소/우편번호→시/군/구)
    -data/zip_prefix_regions.csv 추가(우편번호 앞 3자리 매핑)
    -jobs/ontong/zipmap_prefix.py 구현 → 전처리에서 zipCd 파싱해 region 자동 주입
    -DB 반영 확인(경상남도 등 접두어 매칭 결과 조회)
4. API 서버(FastAPI) 동작 확인,
    - 서버 실행: py -m uvicorn main:app --reload --host 127.0.0.1 --port 8000
    -Swagger UI(/docs)로 확인
    - 4-1. 엔드포인트
    -목록: GET /api/policies (검색/필터: q, region, category, category_auto, limit/offset)
    -상세: GET /api/policies/{plcy_no}
5. 데이터 적재/검증,
    - 수집 원문 policy_raw 적재 확인
    -전처리 결과 policy_clean 적재 및 필드 점검
    -샘플 레코드 상세 조회 성공: /api/policies/20250924005400211793
    
    ------------------------------------------------------
    ------------------------------------------------------
    로컬 환경에서 테스트 하는 방법
    ------------------------------------------------------
    

1. 환경변수 (.env) 생성,
    - 온통청년 api 키 입력할 것
2. requirements.txt 내 패키지 설치,
3. postgreSQL에서 DB 생성,
    
    >> psql -U postgres -h localhost -p 5432
    >> CREATE DATABASE youth_policy;
    
4. backend/api-server 에서 실행 후 DB에서 데이터 적재 확인,
    
    >> python -m jobs.ontong.fetch_and_clean
    
5. API 서버 실행,
    
    >> $env:PYTHONPATH = "$PWD"
    >> py -m uvicorn main:app --reload --host 127.0.0.1 --port 8000
    
6. 브라우저 테스트,
    
    => Swagger:
    
    http://127.0.0.1:8000/docs
    
    - 6-1. 목록: GET /api/policies
    => 파라미터 없이 실행 시 최근 데이터 나옴
    => 또는 검색 예시:
    region=부산광역시
    limit=5
    - 6-2. 상세: GET /api/policies/{plcy_no}
    => 검색 예시:
    plcy_no=20250924005400211793
