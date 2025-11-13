# 다나와 옵션/리뷰 통합 크롤러 (crawlerV2)

Playwright(Python) 기반으로 다나와 카테고리 페이지의 상품 정보, 옵션 정보, 리뷰 정보를 자동 수집해 CSV로 저장하는 크롤러입니다.
최신 버전에서는 옵션 존재 여부에 따른 자동 분기 처리, 상세정보/리뷰 통합 크롤링, 정확한 상품 선택자 적용, 페이지 상태 복구 로직 안정화가 포함되어 있습니다.

## 사전 준비

- Python 3.10 이상 권장 (Windows 10/11 테스트 완료)
- pip 사용 가능 환경
- Playwright가 브라우저(Chromium)를 자동 설치하므로 별도 설치 불필요

## 설치 방법

```
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python -m playwright install
```

## 실행 예시

```
python merged_crawler_final_v2.py ^
  --category-url "https://prod.danawa.com/list/?cate=16249098" ^
  --output result.csv ^
  --items-per-page 20 ^
  --max-total-items 100 ^
  --delay-ms 800 ^
  --headless
```

## 주요 옵션

- --category-url : 크롤링 대상 다나와 카테고리 URL (필수)
- --output : 결과 CSV 파일명
- --items-per-page : 페이지당 최대 크롤링 상품 수
- --max-total-items : 전체 크롤링 상품 제한 수
- --pages : 최대 페이지 수
- --delay-ms : 요청 대기 시간(ms). 권장: 800~1500
- --headless : 브라우저 창 없이 실행

## 옵션 및 리뷰 크롤링 구조

### 옵션이 있는 경우
1. 상세 페이지에서 옵션 리스트(li.list-item) 추출
2. 옵션별 상세 페이지 접속
3. 상세정보(#bookmark_product_information) 크롤링
4. 리뷰(#bookmark_cm_opinion) 크롤링
5. 리뷰 여러 개면 여러 행 생성
6. 옵션 첫 번째 행만 상세정보 포함, 리뷰 행은 상세정보 공백 처리

### 옵션이 없는 경우
1. 기본 상세정보 1행 저장
2. 리뷰 페이지 이동 후 리뷰 크롤링
3. 리뷰 여러 행 생성
4. 리뷰 행은 상세정보 공백 처리

## 페이징 처리

movePage(N) 호출 기반 이동 + 리스트 페이지 복구 로직 적용.
정확한 첫 상품 선택자를 사용해 오판 방지:

```
li.prod_item div.prod_info a.prod_link
```

## CSV 출력 구조

| BASE_URL | 현재URL | 상품명 | URL | 옵션스펙 | 상세정보 | 리뷰별점 | 리뷰내용 | 리뷰작성일 |

상세정보는 기본/옵션 첫 행에만 포함되며, 리뷰 행은 공백("") 처리됨.

## 문제 해결

### 1. PermissionError: 'result.csv'
- 파일이 열려 있음 → CSV 파일을 닫고 재실행
- 경로 쓰기 권한 문제 → 다른 경로 지정 (--output "./output.csv")

### 2. 페이지 복구 무한 루프
- 원인: 잘못된 선택자로 '인기 순위'를 상품명으로 인식
- 해결: 올바른 선택자 적용 (v2에 패치됨)

### 3. 리뷰 행 상세정보 중복 문제
- 리뷰 행의 상세정보를 ""로 처리하여 해결 (v2 적용)

### 4. 최대 아이템 수 메시지가 두 번 출력
- break가 product loop만 종료함 → return 사용 권장

## 사용 팁

- 안정성 우선: delay-ms 800~1500 추천
- headless 제거 시 동작 확인 가능
- 개발 단계에서는 max-total-items로 크롤링량 조절

## 주의사항

- 사이트 이용약관과 로봇 정책을 준수해야 합니다.
- 크롤링으로 인한 법적 책임은 사용자에게 있습니다.
