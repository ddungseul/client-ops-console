# 고객사 운영 콘솔 — 구글 시트 + 앱스 스크립트 + GitHub Pages

고객사 운영 데이터를 모니터링하고, 확인이 필요한 건을 이슈로 감지해 처리하며, 정기 리포트와 안내 메일을 개별·일괄·자동(예약)으로 보내는 운영 콘솔입니다.
어떤 고객사 운영 조직에도 적용할 수 있는 구조이며, **3PL 물류센터 CX팀이 화주사 100곳을 운영하는 사례**로 구현했습니다. 화면과 데이터의 화주사·담당자·고객 정보는 모두 가상 데이터이며 실존 브랜드·인물과 관계없습니다.
원본 데이터는 구글 시트에 두고, 앱스 스크립트가 시트를 읽어 API로 응답하며 Gmail로 메일을 발송합니다. 웹페이지는 GitHub Pages에 공개합니다.

| 업무 흐름 | 콘솔 기능 |
|---|---|
| 모니터링 | 대시보드: 오늘 지표, 확인 필요 항목, 최근 7일 추이, 화주사별 현황 |
| 감지 | 이슈 감지: 출고지연·출고보류·배송실패·재고 부족·사용기한 임박·입고 이슈를 매시간 찾아 이슈로 등록하고 CX팀에 알림 |
| 처리 | 이슈 처리: 담당자·상태·처리 메모·처리 기한·이력 관리 |
| 고객사 안내 | 이슈별 안내 메일, 정기 리포트(개별·일괄·자동 발송), 발송 기록 |

---

## 1. 구성

```
[웹페이지 · GitHub Pages]
      │  조회(GET) · 발송(POST)
      ▼
[앱스 스크립트 웹 앱 = API]  ──메일 발송──▶  [Gmail]
      │  읽기 · 발송 기록 쓰기
      ▼
[구글 시트 = 원본 데이터]
```

| 파일 | 내용 | 올리는 곳 |
|---|---|---|
| `3pl_data.xlsx` | 원본 데이터. 시트 20개(안내 + 데이터 19개) | 구글 드라이브 → 구글 시트로 변환 |
| `Code.gs` | API · 메일 발송 코드 | 구글 시트의 앱스 스크립트 편집기 |
| `index.html` | 운영 콘솔 웹페이지 (10장 참고) | GitHub 저장소 → GitHub Pages |
| `README.md` | 설정 순서와 API 명세(이 문서) | — |

---

## 2. 설정 순서

### 2-1. 구글 시트 만들기

1. 구글 드라이브에서 **새로 만들기** → **Google 스프레드시트**로 빈 시트를 만들고, 파일 이름을 `3PL 원본 데이터`로 바꿉니다.
2. 메뉴 **파일** → **가져오기** → **업로드**에서 `3pl_data.xlsx`를 선택합니다.
3. 가져오기 위치를 **스프레드시트 바꾸기**로 고르고 **데이터 가져오기**를 누릅니다.
4. 하단 시트 탭이 `안내`, `clients`, `contacts` … `mail_logs`까지 20개인지 확인합니다.

.xlsx 파일을 드라이브에서 바로 열면 엑셀 호환 모드로 열려 앱스 스크립트 메뉴를 쓸 수 없습니다. 반드시 위 방법으로 구글 시트 파일에 가져옵니다.

### 2-2. 앱스 스크립트 넣기

1. 구글 시트 메뉴 **확장 프로그램** → **Apps Script**를 엽니다.
2. 기본으로 있는 `Code.gs`의 내용을 모두 지우고, 받은 `Code.gs` 내용을 붙여 넣은 뒤 저장(💾)합니다.
3. 상단 함수 선택 칸에서 `initProperties`를 고르고 **실행**을 누릅니다.
4. 처음 실행하면 권한 승인 창이 열립니다. 본인 계정 선택 → **고급** → **(프로젝트 이름)(으)로 이동(안전하지 않음)** → **허용**을 누릅니다. 본인이 만든 스크립트라서 나오는 경고입니다.
5. 실행 로그에 `API_KEY = key-…` 값이 표시됩니다. 웹페이지에 넣을 값이므로 복사해 둡니다.

### 2-3. 설정 점검과 메일 발송 테스트

1. 함수 선택 칸에서 `checkSetup`을 실행합니다. 로그 마지막 줄이 `설정 정상`이면 됩니다. Gmail 권한을 추가로 묻는 창이 열리면 2-2의 4번과 같이 허용합니다.
2. `sendTestMail`을 실행합니다. 본인 Gmail로 `[테스트] [일일 운영 요약] 루미에르 뷰티 발송 테스트` 메일과 CSV 첨부가 오면 됩니다.
3. 구글 시트의 `mail_logs` 시트에 발송 기록 1행이 추가된 것을 확인합니다.

### 2-4. 웹 앱으로 배포

1. 편집기 오른쪽 위 **배포** → **새 배포**를 누릅니다.
2. 유형 선택(⚙) → **웹 앱**을 고릅니다.
3. **다음 사용자 인증 정보로 실행**: `나`, **액세스 권한이 있는 사용자**: `모든 사용자`로 설정하고 **배포**를 누릅니다.
4. 표시되는 **웹 앱 URL**(`https://script.google.com/macros/s/…/exec`)을 복사해 둡니다.
5. 브라우저 주소창에 아래 주소를 넣어 본문에 `{"data":`로 시작하는 JSON이 나오면 배포가 끝난 것입니다. 주소창의 주소는 `script.googleusercontent.com/macros/echo?…`로 바뀌는데, 앱스 스크립트의 정상 동작입니다.
   ```
   웹앱URL?path=/v1/orders&page_size=1&api_key=복사한API키
   ```

### 2-5. 웹페이지 연결

1. `index.html`을 브라우저로 엽니다. 연결 설정이 저장되지 않은 브라우저는 공개 데모(조회 전용 키)로 연결됩니다(4장 참고).
2. 상단 **연결 설정**에서 웹 앱 URL과 API 키를 넣고 **저장 후 연결**을 누릅니다. "연결되었습니다. 화주사 100곳" 안내가 나오면 연결된 것입니다.
3. **리포트 발송** 화면에서 리포트를 한 건 보내 Gmail로 메일이 오는지 확인합니다.
4. 확인이 끝나면 GitHub 저장소에 `index.html`을 올리고 GitHub Pages를 켭니다. (지난 대시보드와 같은 방법입니다.)

### 2-6. 코드를 수정했을 때

`Code.gs`를 고친 뒤에는 **배포** → **배포 관리** → 연필 아이콘 → 버전 **새 버전** → **배포**를 눌러야 반영됩니다. 이 방법으로 배포하면 웹 앱 URL이 바뀌지 않습니다. **새 배포**를 누르면 URL이 새로 생기니 주의합니다.

구글 시트 값만 고친 경우에는 다시 배포할 필요가 없습니다. 다만 조회 응답이 10분간 캐시되므로, 바로 반영하려면 편집기에서 `clearCache`를 실행합니다.

### 2-7. 자동 발송 · 이슈 감지 켜기

1. 편집기에서 `installScheduler`를 실행합니다. 처음 실행하면 권한 승인 창이 열리며, 2-2의 4번과 같이 허용합니다(시간 트리거 권한이 추가됨).
2. 실행 로그에 "자동 발송 실행기를 설치했습니다."가 나오면 완료입니다. 이후 매시간 `runSchedules`가 실행되어 이슈를 감지하고(7-15), 시각이 된 예약을 발송합니다.
3. 콘솔의 **자동 발송** 화면 위쪽이 "자동 발송 실행기 켜짐"으로 바뀌었는지 확인합니다.
4. 끌 때는 `removeScheduler`를 실행합니다. 예약 목록은 그대로 남습니다.

예약 목록은 `schedules` 시트에, 이슈는 `issues` 시트에 저장됩니다. 두 시트는 처음 사용할 때 자동으로 만들어집니다. `schedules` 시트에는 예시 예약 3건(사용 안 함)이 들어갑니다.

---

## 3. 데이터 기준

| 항목 | 내용 |
|---|---|
| 기준 시각 | 2026-11-16 13:50 (KST) 고정. 언제 조회해도 이 시각 기준의 상태로 조회됨 |
| 데이터 기간 | 2026-10-01 \~ 2026-11-16 주문, 2026-11-27까지 입고예정 |
| 데이터 수정 | 구글 시트 값을 고치면 다음 요청부터 바로 반영됨 (재배포 불필요) |
| 메일 발송 기록 | `mail_logs` 시트에 1건씩 추가됨 |
| 이슈 | `issues` 시트에 저장됨. 감지·처리 시각은 실제 현재 시각 기준 |
| 가상 데이터 | 화주사·담당자·고객 이름과 이메일, 주문·운송장·정산 값은 모두 가상 데이터이며 실존 브랜드·인물과 관계없음. 이메일 도메인은 `example.com`. 택배사·판매채널 이름은 구분값으로만 사용 |

### 센터 운영 규칙

| 항목 | 규칙 |
|---|---|
| 작업일 | 월\~토 (일요일 휴무, 택배 집하·배송 없음) |
| 피킹 웨이브 | 09:30, 11:30, 13:30, 15:00 |
| 출고 마감 | 16:00 패킹 완료분까지 당일 출고 |
| 택배사 집하 | 17:00 |
| 출고 약속일 | 작업일 14:00 이전 주문은 당일, 이후 주문은 다음 작업일 |
| 출고지연 | 출고 약속일에 출고되지 않은 주문 (`is_delayed: true`) |

### 화주사

이커머스 브랜드 100곳(`C001`\~`C100`)입니다. 일반 3PL 센터에서 많이 다루는 품목으로 구성했고, 식품은 다루지 않습니다. 전체 목록은 `clients` 시트 또는 콘솔의 **화주사 · 담당자** 화면에서 확인합니다.

| 항목 | 내용 |
|---|---|
| 품목 | 패션의류, 패션잡화, 신발, 뷰티, 생활화학, 주방용품, 디지털 액세서리, 소형가전, 문구·팬시, 반려동물용품, 스포츠·아웃도어, 유아용품, 리빙·인테리어 (화주사 1곳당 1개 품목) |
| 보관 | 뷰티·디지털 액세서리·소형가전은 항온, 그 외 상온 |
| 택배사 | CJ대한통운, 한진택배, 롯데택배, 우체국택배 |
| SKU | `화주사ID-일련번호` (예: `C001-003`) |
| LOT번호 | `화주사ID` + 입고 LOT 날짜(YYMMDD) (예: `C001260226`) |
| 물량 | 화주사마다 다름. 화주사당 주문 14\~140건(2026-10-01 \~ 2026-11-16) |

담당자 역할과 수신 리포트

| 역할 | 메일 주소 | 수신 리포트 |
|---|---|---|
| CS 담당 | `cs@브랜드.example.com` | 출고지연 현황, 배송실패 현황, 반품 처리 현황 |
| 물류 담당 | `logistics@브랜드.example.com` | 일일 운영 요약, 안전재고 미달 알림, 사용기한 임박 알림, 입고 이슈 알림 |
| 운영 담당 | `ops@브랜드.example.com` | CS 담당과 물류 담당의 수신 리포트 전체 |
| 정산 담당 | `billing@브랜드.example.com` | 월 정산서 |
| 대표 | `ceo@브랜드.example.com` | 일일 운영 요약, 안전재고 미달 알림, 사용기한 임박 알림, 반품 처리 현황, 월 정산서 |

화주사마다 담당자 구성이 다릅니다(CS·물류·정산 / 운영·정산 / 대표 1명). 리포트를 받는 담당자가 없는 화주사도 있으며, 이때는 리포트 발송 화면에서 받는 사람을 직접 입력합니다.

### 상품 관리 기준

| 항목 | 기준 |
|---|---|
| LOT번호 | 모든 재고·입고 품목에 `lot_no` 표시 |
| 사용기한 관리 품목 | 뷰티, 생활화학, 유아용품. 이 품목만 `expiry_date`가 있고, 그 외 품목은 `expiry_date`가 비어 있음 |
| 입고 기준 | 사용기한 관리 품목은 입고일로부터 사용기한 12개월 이상 남은 제품. 미달 시 입고 이슈(`EXPIRY_SHORT`). 그 외 품목은 상품 바코드 부착. 미부착 시 입고 이슈(`BARCODE_MISSING`) |
| 사용기한 임박 | 기준 시각으로부터 사용기한이 180일 미만 남은 재고 (`is_expiry_near: true`) |
| 출고 순서 | 사용기한 관리 품목은 사용기한이 빠른 LOT부터(FEFO), 그 외 품목은 먼저 입고된 LOT부터(FIFO) |
| 반품 재입고 | 택·포장이 유지되고 사용 흔적이 없는 제품만 재입고. 불량·파손 제품은 폐기 |


---

## 4. 인증과 메일 발송 설정

### API 키

| 요청 | API 키 넣는 위치 |
|---|---|
| 조회(GET) | 주소의 조회 조건: `?api_key=API키` |
| 발송(POST) | 요청 본문(JSON)의 `api_key` 필드 |

앱스 스크립트는 요청 헤더를 읽을 수 없어서 헤더(`X-API-Key`) 방식은 쓰지 않습니다.

API 키는 2종류입니다.

| 키 | 스크립트 속성 | 가능한 요청 | 보관 위치 |
|---|---|---|---|
| 전체 키 | `API_KEY` | 조회·발송·처리 모두 | 연결 설정 창에서 입력. 각자의 브라우저에만 저장 |
| 조회 전용 키 | `DEMO_KEY` | 조회(GET)와 일괄 발송의 발송 대상 확인(`dry_run: true`)만. 그 외 POST는 `READ_ONLY` 오류 | `index.html` 코드의 `DEMO` 설정. 포트폴리오 공개용 |

GitHub Pages에 올린 웹페이지는 누구나 소스를 볼 수 있으므로 전체 키는 코드에 넣지 않습니다. 조회 전용 키는 발송·수정이 막혀 있어 코드에 넣어 공개합니다. 연결 설정이 저장되지 않은 브라우저나 주소에 `?demo`를 붙여 연 경우 조회 전용 키로 연결되고, 화면 상단에 '공개 데모 · 조회 전용'이 표시됩니다. 키가 노출되었다면 스크립트 속성의 `API_KEY` 또는 `DEMO_KEY` 값을 바꾸면 됩니다. `DEMO_KEY`를 바꾼 경우 `index.html`의 `DEMO.apiKey`도 같이 바꿉니다.

### 스크립트 속성

편집기 왼쪽 **프로젝트 설정**(⚙) → **스크립트 속성**에서 확인·수정합니다. 수정 후 다시 배포할 필요는 없습니다.

| 속성 | 기본값 | 설명 |
|---|---|---|
| `API_KEY` | `initProperties` 실행 시 자동 생성 | 전체 키. 요청의 `api_key`와 같아야 함 |
| `DEMO_KEY` | `initProperties` 실행 시 자동 생성 | 조회 전용 키. 비우면 조회 전용 키를 쓰지 않음 |
| `TEST_MODE` | `true` | `true`: 모든 메일을 `TEST_RECIPIENT`에게만 발송. `false`: 요청한 받는 사람에게 실제 발송 |
| `TEST_RECIPIENT` | 스크립트 소유자 메일 | 테스트 모드 수신 메일 |
| `SENDER_NAME` | `물류센터 CX팀` | 메일의 보내는 사람 이름. 리포트 본문의 인사말과 서명에도 쓰임 |
| `CX_INBOX` | 스크립트 소유자 메일 | 이슈 감지 알림을 받을 CX팀 메일. 테스트 모드에서는 `TEST_RECIPIENT`로 감 |
| `TEST_BULK_LIMIT` | `3` | 테스트 모드에서 일괄·자동 발송 1회당 실제로 보낼 화주사 수 |
| `CACHE_VERSION` | 없음 | `clearCache` 실행 시 자동으로 바뀜. 직접 고치지 않음 |
| `ISSUE_LAST_SCAN` | 없음 | 마지막 이슈 감지 시각. 감지할 때 자동으로 바뀜. 직접 고치지 않음 |

### 테스트 모드

가상 데이터의 담당자 메일(`…@….example.com`)은 실제로 없는 주소입니다. 포트폴리오로 공개하는 동안에는 `TEST_MODE`를 `true`로 유지합니다.

| 항목 | 테스트 모드 (`true`) | 실제 발송 (`false`) |
|---|---|---|
| 실제 수신자 | `TEST_RECIPIENT` 1명 | 요청한 `to`, `cc` |
| 제목 | 앞에 `[테스트]` 추가 | 요청한 제목 그대로 |
| 본문 | 맨 위에 원래 받는 사람·참조 표시 | 요청한 본문 그대로 |
| 일괄·자동 발송 | 처음 `TEST_BULK_LIMIT`곳(기본 3곳)만 발송, 나머지는 '테스트 생략' | 대상 화주사 전체 발송 |
| 이슈 안내 여러 건 발송 | 처음 `TEST_BULK_LIMIT`건(기본 3건)만 발송, 나머지는 '테스트 생략' | 선택한 이슈 전체 발송 |
| 발송 기록 `mode` | `TEST` | `LIVE` |

### 메일 발송 한도

Gmail 계정마다 하루 메일 발송 한도가 있습니다. 개인 Gmail 계정은 하루 약 100명입니다. 남은 한도는 `checkSetup` 실행 로그에서 확인합니다. 한도를 넘으면 `QUOTA_EXCEEDED` 오류가 반환됩니다.

---

## 5. 공통 규칙

### 요청 주소

```
웹앱URL?path=경로&조회조건&api_key=API키
예) https://script.google.com/macros/s/AKfy…/exec?path=/v1/orders&status=HOLD&api_key=key-…
```

경로는 반드시 `path` 조회 조건으로 보냅니다. 웹 앱 URL 뒤에 경로를 붙이는 방식(`…/exec/v1/orders`)은 브라우저 주소창에서는 되지만, 웹페이지에서 `fetch`로 호출하면 `Failed to fetch` 오류로 차단됩니다.

이 문서의 `GET /v1/orders` 같은 표기는 `path=/v1/orders`로 요청한다는 뜻입니다.

### 응답 형식

목록 조회
```json
{
  "data": [ { ... }, { ... } ],
  "meta": { "page": 1, "page_size": 20, "total": 4817, "total_pages": 241 }
}
```

단건 조회
```json
{ "data": { ... } }
```

오류
```json
{ "error": { "status": 400, "code": "INVALID_PARAM", "message": "date_from는 YYYY-MM-DD 형식이어야 합니다." } }
```

앱스 스크립트는 오류가 나도 HTTP 상태 코드를 항상 200으로 보냅니다. 오류 여부는 응답의 `error` 필드로 판단하고, 상태 코드는 `error.status`로 확인합니다.

### 공통 조회 조건

| 조회 조건 | 설명 | 예시 |
|---|---|---|
| `page` | 페이지 번호. 기본 1 | `page=2` |
| `page_size` | 페이지당 건수. 기본 20, 최대 100 | `page_size=100` |
| `client_id` | 화주사. 쉼표로 여러 개 지정 | `client_id=C001,C003` |
| `status` | 상태 코드. 쉼표로 여러 개 지정 | `status=HOLD,DELIVERY_FAILED` |
| `date_from`, `date_to` | 기간(YYYY-MM-DD, 양 끝 포함). 기준 날짜 필드는 엔드포인트마다 다름 | `date_from=2026-11-01&date_to=2026-11-15` |

- 조회 조건은 `&`로 연결하며 모두 동시에 적용됩니다(AND).
- 지원하지 않는 조회 조건은 무시되고 `meta.ignored_params`에 표시됩니다. 오타 확인에 사용합니다.
- 처음 요청하는 조회는 시트를 읽느라 4\~12초 걸립니다. 같은 경로·조회 조건의 응답은 10분간 캐시되어 두 번째부터는 1\~2초입니다.
- 캐시 때문에 구글 시트를 직접 고친 내용은 최대 10분 뒤에 반영됩니다. 바로 반영하려면 편집기에서 `clearCache`를 실행합니다.
- 메일 발송, 발송 기록(`/v1/mail/…`), 자동 발송 예약(`/v1/schedules…`), 이슈(`/v1/issues…`)는 캐시하지 않습니다.

### 오류 코드

| error.status | error.code | 발생 조건 |
|---|---|---|
| 400 | `INVALID_PARAM` | 조회 조건 형식 오류 |
| 400 | `INVALID_JSON` | 요청 본문이 JSON 형식이 아님 |
| 400 | `VALIDATION_ERROR` | 메일 발송 요청 값 오류. `details`에 항목별 사유 |
| 401 | `UNAUTHORIZED` | API 키 없음 또는 불일치 |
| 403 | `READ_ONLY` | 조회 전용 키로 발송·처리(POST) 요청 |
| 404 | `NOT_FOUND` | 없는 경로 또는 없는 ID |
| 405 | `METHOD_NOT_ALLOWED` | 지원하지 않는 메서드 |
| 429 | `QUOTA_EXCEEDED` | 오늘 메일 발송 한도 초과 |
| 500 | `CONFIG_ERROR` | 스크립트 속성 `API_KEY` 없음 |
| 500 | `SHEET_NOT_FOUND` | 필요한 시트가 없음. 시트 이름 변경 여부 확인 |
| 500 | `INTERNAL_ERROR` | 그 밖의 오류. 편집기 **실행** 메뉴의 실행 기록에서 확인 |

---

## 6. 상태 코드

### 주문 (`/v1/orders`의 `status`)

| 단계 | status | status_label |
|---|---|---|
| 주문 | `RECEIVED` | 주문접수 |
| 주문 | `ALLOCATED` | 재고할당 |
| 피킹 | `PICKING` | 피킹중 |
| 피킹 | `PICKED` | 피킹완료 |
| 패킹 | `PACKING` | 패킹중 |
| 패킹 | `PACKED` | 패킹완료 |
| 출고 | `SHIPPED` | 출고완료 |
| 배송 | `IN_TRANSIT` | 배송중 |
| 배송 | `OUT_FOR_DELIVERY` | 배송출발 |
| 배송 | `DELIVERED` | 배송완료 |
| 예외 | `DELIVERY_FAILED` | 배송실패 |
| 예외 | `HOLD` | 출고보류 (`hold_reason`: 재고부족, 주소확인필요) |
| 예외 | `CANCELLED` | 주문취소 |

### 그 외

| 대상 | status (status_label) |
|---|---|
| 입고 | `SCHEDULED`(입고예정), `ARRIVED`(도착), `INSPECTING`(검수중), `COMPLETED`(입고완료) |
| 피킹 · 패킹 작업 | `WAITING`(대기), `IN_PROGRESS`(작업중), `DONE`(완료) |
| 출고 | `READY`(출고대기), `SHIPPED`(출고완료) |
| 배송 | `SHIPPED`(집하완료), `IN_TRANSIT`(배송중), `OUT_FOR_DELIVERY`(배송출발), `DELIVERED`(배송완료), `DELIVERY_FAILED`(배송실패) |
| 반품 | `REQUESTED`(반품접수), `PICKUP_SCHEDULED`(회수예정), `COLLECTED`(회수완료), `RECEIVED`(센터입고), `INSPECTING`(검수중), `COMPLETED`(처리완료) |
| 반품 처리 결과 | `RESTOCKED`(재입고), `DISPOSED`(폐기) |
| 정산 | `DRAFT`(산정중), `CONFIRMED`(확정), `INVOICED`(청구완료), `PAID`(입금완료) |

---

## 7. 엔드포인트

### 7-1. 기준 정보

| 메서드 · 경로 | 설명 | 조회 조건 |
|---|---|---|
| `GET /v1/clients` | 화주사, 담당자, 담당자별 수신 리포트 | `client_id` |
| `GET /v1/report-types` | 리포트 종류 코드 | 없음 |
| `GET /v1/me` | 요청한 키의 권한. `access`: `full`(전체 키) 또는 `read_only`(조회 전용 키) | 없음 |

리포트 종류 코드

| code | 이름 |
|---|---|
| `daily_summary` | 일일 운영 요약 |
| `outbound_delay` | 출고지연 현황 |
| `delivery_failed` | 배송실패 현황 |
| `stock_alert` | 안전재고 미달 알림 |
| `expiry_alert` | 사용기한 임박 알림 |
| `inbound_issue` | 입고 이슈 알림 |
| `return_status` | 반품 처리 현황 |
| `settlement` | 월 정산서 |

### 7-2. 입고

| 메서드 · 경로 | 설명 |
|---|---|
| `GET /v1/inbounds` | 입고 목록. 입고예정일 최신순 |
| `GET /v1/inbounds/{inbound_id}` | 입고 단건 |

| 조회 조건 | 설명 |
|---|---|
| `client_id`, `status` | 공통 |
| `has_issue` | `true`: 수량불일치·파손·사용기한 미달·바코드 미부착 입고만 |
| `date_from`, `date_to` | 기준: 입고예정일(`expected_date`) |

주요 필드: `expected_date`, `arrived_at`, `completed_at`, `total_expected_qty`, `total_received_qty`, `has_issue`, `issue_type`(`QTY_MISMATCH` 수량불일치, `DAMAGED` 외박스 파손·상품 손상, `EXPIRY_SHORT` 사용기한 기준 미달, `BARCODE_MISSING` 상품 바코드 미부착), `issue_note`, `items[]`(SKU별 LOT번호, 사용기한, 예정·입고·불량 수량)

### 7-3. 보관(재고)

| 메서드 · 경로 | 설명 |
|---|---|
| `GET /v1/inventory` | SKU별 재고. 기준 시각 현재 상태 |

| 조회 조건 | 설명 |
|---|---|
| `client_id` | 공통 |
| `sku` | SKU. 쉼표로 여러 개 지정 |
| `is_low_stock` | `true`: 가용재고가 안전재고보다 적은 SKU만 |
| `is_expiry_near` | `true`: 사용기한이 180일 미만 남은 SKU만 |
| `storage_type` | `상온`, `항온` |
| `q` | 상품명 또는 SKU 일부 검색 |

주요 필드: `location`(로케이션), `qty_on_hand`(실재고), `qty_allocated`(할당), `qty_available`(가용재고 = 실재고 − 할당), `safety_stock`(안전재고), `is_low_stock`, `lot_no`(LOT번호), `expiry_date`(사용기한, 사용기한 관리 품목만), `is_expiry_near`, `last_inbound_at`

`lot_no`와 `expiry_date`는 다음에 출고될 LOT(FEFO·FIFO 기준으로 가장 먼저 출고할 LOT) 기준입니다.

### 7-4. 주문

| 메서드 · 경로 | 설명 |
|---|---|
| `GET /v1/orders` | 주문 목록. 주문일시 최신순 |
| `GET /v1/orders/{order_id}` | 주문 단건. 피킹·패킹·출고·배송·반품 정보와 진행 이력(`timeline`) 포함 |

| 조회 조건 | 설명 |
|---|---|
| `client_id`, `status` | 공통 |
| `channel` | 판매채널: `스마트스토어`, `쿠팡`, `자사몰`, `29CM`, `카카오선물하기` |
| `is_delayed` | `true`: 출고지연 주문만 |
| `has_return` | `true`: 반품이 접수된 주문만 |
| `q` | 주문번호, 고객명, 운송장번호 일부 검색 |
| `date_from`, `date_to` | 기준: 주문일시(`ordered_at`) |

주요 필드: `channel`, `ordered_at`, `customer`(이름·이메일·연락처·주소), `items[]`, `carrier`, `tracking_no`, `promised_ship_date`, `shipped_at`, `delivered_at`, `is_delayed`, `delay_reason`, `hold_reason`, `return_id`

### 7-5. 피킹 · 패킹

| 메서드 · 경로 | 설명 | 조회 조건 |
|---|---|---|
| `GET /v1/picking-tasks` | 피킹 작업. SKU별 로케이션과 피킹 수량 | `client_id`, `status`, 기간(기준: 작업 생성일시) |
| `GET /v1/packing-tasks` | 패킹 작업. 박스 규격과 중량 | `client_id`, `status`, 기간(기준: 작업 생성일시) |

### 7-6. 출고

| 메서드 · 경로 | 설명 |
|---|---|
| `GET /v1/outbounds` | 패킹이 끝나 운송장이 출력된 주문의 출고 정보 |

| 조회 조건 | 설명 |
|---|---|
| `client_id`, `status` | 공통 |
| `carrier` | 택배사 |
| `is_delayed` | `true`: 출고지연만 |
| `date_from`, `date_to` | 기준: 출고 약속일(`promised_ship_date`) |

### 7-7. 배송

| 메서드 · 경로 | 설명 |
|---|---|
| `GET /v1/shipments` | 출고된 주문의 배송 정보와 배송 이력(`events`) |
| `GET /v1/shipments/{tracking_no}` | 운송장번호로 단건 조회 |

| 조회 조건 | 설명 |
|---|---|
| `client_id`, `status` | 공통 |
| `carrier`, `tracking_no` | 택배사, 운송장번호 |
| `has_failed` | `true`: 배송실패 이력이 있는 건만 (재배송 완료 건 포함) |
| `date_from`, `date_to` | 기준: 출고일시(`shipped_at`) |

주요 필드: `delivery_attempts`(배송 시도 횟수), `failed_reason`(수취인 부재, 주소 불명, 공동현관 출입 불가), `events[]`

### 7-8. 반품

| 메서드 · 경로 | 설명 |
|---|---|
| `GET /v1/returns` | 반품 목록. 반품 접수일시 최신순 |
| `GET /v1/returns/{return_id}` | 반품 단건 |

| 조회 조건 | 설명 |
|---|---|
| `client_id`, `status` | 공통 |
| `reason` | 반품 사유: `단순 변심`, `사이즈·색상 불만`, `상품 불량`, `오배송`, `배송 중 파손` |
| `disposition` | 처리 결과: `RESTOCKED`, `DISPOSED` |
| `date_from`, `date_to` | 기준: 반품 접수일시(`requested_at`) |

주요 필드: `reason`, `fee_payer`(반품 배송비 부담: 고객 또는 판매자), `return_tracking_no`(회수 운송장), `requested_at`, `collected_at`, `received_at`, `completed_at`, `disposition`, `inspection_note`

### 7-9. 정산

| 메서드 · 경로 | 설명 |
|---|---|
| `GET /v1/settlements` | 화주사별 월 정산. 2026-10은 확정 이후 상태, 2026-11은 산정중 |
| `GET /v1/settlements/{settlement_id}` | 정산 단건. 예: `ST-202610-C001` |

| 조회 조건 | 설명 |
|---|---|
| `client_id`, `status` | 공통 |
| `period` | 정산월(YYYY-MM) |

정산 항목(`lines[]`)

| 항목 | 단위 | 단가(원) | 산정 기준 |
|---|---|---|---|
| 입고료 | EA | 150 | 기간 내 입고완료 수량 |
| 보관료 | 팔레트·일 | 1,000 | 사용 팔레트 수 × 일수 |
| 피킹 기본료 | 건 | 500 | 기간 내 출고 주문 수 |
| 피킹 추가료 | EA | 100 | 출고 수량 − 출고 주문 수 |
| 패킹·부자재비 | 건 | 700 | 기간 내 출고 주문 수 |
| 택배비 | 건 | 2,800 | 기간 내 출고 주문 수 |
| 재배송비 | 건 | 2,000 | 기간 내 배송실패 건수 |
| 반품 처리비 | 건 | 3,000 | 기간 내 반품 처리완료 건수 |

주요 필드: `subtotal`(공급가액), `vat`(부가세), `total_amount`(합계), `confirmed_at`, `invoiced_at`, `due_date`(입금 기한), `paid_at`

### 7-10. 일일 운영 요약 리포트

| 메서드 · 경로 | 설명 |
|---|---|
| `GET /v1/reports/daily-summary` | 화주사별 하루 운영 수치 |

| 조회 조건 | 설명 |
|---|---|
| `client_id` | 생략하면 전체 화주사. 쉼표로 여러 개 지정 |
| `date` | YYYY-MM-DD. 생략하면 기준일(2026-11-16) |

응답 구성

| 구분 | 필드 |
|---|---|
| `inbound` | 입고예정(`scheduled`), 입고완료(`completed`), 이슈(`with_issue`) |
| `orders` | 주문접수(`received`), 주문취소(`cancelled`) |
| `outbound` | 당일 출고대상(`due_today`), 출고완료(`shipped`), 미출고(`not_shipped`), 출고지연(`delayed`) |
| `delivery` | 배송완료(`delivered`), 배송실패(`failed`) |
| `returns` | 반품접수(`requested`), 반품 처리완료(`completed`) |
| `current` | `date`와 관계없이 기준 시각 현재 상태: 출고보류 주문(`on_hold_orders`), 진행 중 반품 수(`returns_in_progress`), 안전재고 미달 SKU(`low_stock_skus`), 사용기한 임박 SKU(`expiry_near_skus`) |

### 7-11. 대시보드

| 메서드 · 경로 | 설명 |
|---|---|
| `GET /v1/reports/dashboard` | 대시보드 첫 화면에 필요한 값을 요청 한 번으로 계산 |

| 조회 조건 | 설명 |
|---|---|
| `date` | YYYY-MM-DD. 생략하면 기준일(2026-11-16) |
| `client_id` | `trend`와 `alerts`의 범위. 생략하면 전체 화주사 |

응답 구성

| 필드 | 내용 |
|---|---|
| `summaries` | 전체 화주사의 `date` 일일 운영 요약 (7-10과 같은 형식, `client_id`와 관계없이 전체) |
| `trend` | `date` 포함 최근 7일의 날짜별 주문접수(`received`)·출고완료(`shipped`)·배송완료(`delivered`) |
| `alerts` | 확인 필요 건수: 출고지연 미출고(`outbound_delay`), 배송실패(`delivery_failed`), 출고보류(`hold`), 안전재고 미달(`stock_alert`), 사용기한 임박(`expiry_alert`), 최근 7일 입고 이슈(`inbound_issue`) |

### 7-12. 메일 발송

| 메서드 · 경로 | 설명 |
|---|---|
| `POST /v1/mail/send` | Gmail로 메일 발송. 테스트 모드 여부는 4장 참고 |
| `GET /v1/mail/logs` | 발송 기록(`mail_logs` 시트). 최신순. 조회 조건: `client_id`, `report_type`, `trigger`, `batch_id`, `issue_id` |
| `GET /v1/mail/logs/{message_id}` | 발송 기록 단건 |

요청 본문 (JSON 문자열, 헤더 `Content-Type: text/plain;charset=utf-8`)

| 필드 | 필수 | 설명 |
|---|---|---|
| `api_key` | 필수 | API 키 |
| `to` | 필수 | 받는 사람. 이메일 문자열 또는 배열 |
| `cc` | 선택 | 참조. 이메일 문자열 또는 배열 |
| `subject` | 필수 | 제목 |
| `body` | 필수 | 본문 |
| `body_type` | 선택 | `text`(기본) 또는 `html` |
| `client_id` | 선택 | 대상 화주사. 발송 기록 조회용 |
| `report_type` | 선택 | 리포트 종류 코드. 발송 기록 조회용 |
| `attachments` | 선택 | 첨부 배열. 각 항목: `filename`, `content`, `content_type`, `encoding` |

- 첨부 `content`는 텍스트(CSV 등)를 그대로 넣습니다. 엑셀·PDF 같은 파일은 Base64 문자열로 넣고 `encoding: "base64"`를 함께 보냅니다.
- CSV 첨부는 엑셀에서 한글이 깨지지 않도록 자동으로 BOM이 붙습니다.
- 제한: 받는 사람과 참조 합계 50명 이하, Gmail 첨부 용량 한도 이하

요청 예시
```json
{
  "api_key": "key-…",
  "to": ["logistics@lumiere.example.com"],
  "cc": ["ops@my3pl.example.com"],
  "subject": "[일일 운영 요약] 루미에르 뷰티 2026-11-16",
  "body": "<p>금일 출고대상 8건 중 ...</p>",
  "body_type": "html",
  "client_id": "C005",
  "report_type": "daily_summary",
  "attachments": [
    { "filename": "출고현황_20261116.csv", "content_type": "text/csv", "content": "주문번호,상태\nOD-20261116-0001,패킹완료" }
  ]
}
```

응답
```json
{
  "data": {
    "message_id": "MAIL-00001",
    "status": "SENT",
    "status_label": "발송완료(테스트 모드)",
    "sent_at": "2026-11-16T17:22:41+09:00",
    "mode": "TEST",
    "to": ["logistics@lumiere.example.com"],
    "cc": ["ops@my3pl.example.com"],
    "actual_to": ["본인@gmail.com"],
    "attachments": [ { "filename": "출고현황_20261116.csv", "content_type": "text/csv", "size_bytes": 50 } ]
  }
}
```

`actual_to`는 실제로 메일을 받은 주소입니다.

발송 기록에는 `trigger`(발송 구분: `manual` 개별, `bulk` 일괄, `schedule` 자동, `issue` 이슈 안내, `digest` 이슈 감지 알림), `batch_id`(일괄·자동 발송 묶음 번호), `issue_id`(이슈 안내의 이슈 번호)가 함께 남습니다. 기존 `mail_logs` 시트에 이 열이 없으면 첫 발송 때 자동으로 추가됩니다.

### 7-13. 리포트 작성 · 발송

리포트 메일(받는 사람, 제목, 본문, CSV 첨부)을 서버에서 작성합니다. 콘솔의 개별 발송·일괄 발송과 자동 발송이 모두 이 양식을 씁니다.

| 메서드 · 경로 | 설명 |
|---|---|
| `GET /v1/reports/compose` | 화주사 1곳의 리포트 메일 미리보기 |
| `POST /v1/reports/send` | 리포트 메일 발송. 화주사 1곳이면 개별 발송, 여러 곳이면 일괄 발송 |

기간 조건 (두 경로 공통, 생략하면 기본값)

| 조건 | 대상 리포트 | 기본값 |
|---|---|---|
| `date` | 일일 운영 요약 | 기준일 |
| `from`, `to` | 출고지연 현황, 배송실패 현황, 입고 이슈 알림, 반품 처리 현황 | 기준일 7일 전 \~ 기준일 |
| `period` | 월 정산서 | 기준일의 전월 |
| (없음) | 안전재고 미달 알림, 사용기한 임박 알림 | 기준 시각 현재 재고 |

`GET /v1/reports/compose` 조회 조건: `report_type`, `client_id`, 기간 조건

응답 주요 필드: `to`(수신 담당자), `subject`, `intro`(안내 문구), `body_template`(본문. `{{INTRO}}` 자리에 안내 문구가 들어감), `row_count`(안내할 건수), `attachment_name`

`POST /v1/reports/send` 요청 본문

| 필드 | 필수 | 설명 |
|---|---|---|
| `api_key` | 필수 | API 키 |
| `report_type` | 필수 | 리포트 종류 코드 |
| `client_ids` | 선택 | 화주사 ID 배열 또는 `"all"`. 생략하면 전체 화주사 |
| `date`, `from`, `to`, `period` | 선택 | 기간 조건 |
| `skip_empty` | 선택 | `true`: 안내할 건수가 0건인 화주사는 보내지 않음. 기본 `false` |
| `attach` | 선택 | `false`: CSV 첨부 생략. 기본 `true` (0건이면 첨부하지 않음) |
| `dry_run` | 선택 | `true`: 메일을 보내지 않고 화주사별 발송 예정 여부만 계산 |
| `cc` | 선택 | 참조. 모든 메일에 같이 들어감 |
| `recipients` | 선택 | 받는 사람 직접 지정. 화주사 1곳일 때만 사용 (`to`는 기간 종료일이므로 받는 사람은 `recipients`로 지정) |
| `intro` | 선택 | 안내 문구 직접 지정. 화주사 1곳일 때만 사용 |

응답 주요 필드

| 필드 | 내용 |
|---|---|
| `batch_id` | 일괄 발송 묶음 번호 (개별 발송은 `null`) |
| `summary` | 대상 수(`total`), 발송(`sent`), 발송 예정(`ready`, `dry_run`일 때), 0건 제외(`skipped_empty`), 테스트 생략(`skipped_test`), 수신 담당자 없음(`no_recipient`), 오류(`error`) |
| `results[]` | 화주사별 `status`, `status_label`, `to`, `subject`, `row_count`, `message_id`, `actual_to`, `reason` |

화주사별 상태

| status | status_label | 의미 |
|---|---|---|
| `SENT` | 발송완료 | 발송함 |
| `READY` | 발송 예정 | `dry_run`에서 발송 대상 |
| `SKIPPED_EMPTY` | 0건 제외 | `skip_empty`이고 안내할 건수가 0건 |
| `SKIPPED_TEST` | 테스트 생략 | 테스트 모드에서 `TEST_BULK_LIMIT`곳을 넘음 |
| `NO_RECIPIENT` | 수신 담당자 없음 | 이 리포트를 받는 담당자가 없음 |
| `ERROR` | 오류 | 작성 또는 발송 실패. `reason`에 사유 |

발송 전 오늘 남은 메일 발송 한도를 확인하며, 한도보다 많으면 아무것도 보내지 않고 `QUOTA_EXCEEDED`를 반환합니다.

### 7-14. 자동 발송 (예약)

| 메서드 · 경로 | 설명 |
|---|---|
| `GET /v1/schedules` | 예약 목록. `meta.scheduler_installed`(실행기 설치 여부), `meta.test_mode`, `meta.test_bulk_limit` 포함 |
| `POST /v1/schedules/save` | 예약 추가·수정. 본문 `{ "schedule": { ... } }`, `schedule_id`가 있으면 수정 |
| `POST /v1/schedules/delete` | 예약 삭제. 본문 `{ "schedule_id": "S004" }` |
| `POST /v1/schedules/run` | 예약 지금 실행. 본문 `{ "schedule_id": "S001" }` |

예약 필드

| 필드 | 설명 |
|---|---|
| `schedule_id` | 예약 ID (`S001` 형식, 추가 시 자동 부여) |
| `name` | 예약 이름 |
| `report_type` | 리포트 종류 코드 |
| `frequency` | `daily`(매일), `weekdays`(평일), `weekly`(매주), `monthly`(매월) |
| `weekday` | `weekly`일 때 요일. 1(월) \~ 7(일) |
| `month_day` | `monthly`일 때 날짜. 1 \~ 28 |
| `hour` | 발송 시각. 0 \~ 23 (한국 시간) |
| `target` | `all`(전체 화주사) 또는 화주사 ID 목록 |
| `skip_empty`, `attach` | 7-13과 같음 |
| `enabled` | 사용 여부 |
| `last_run_at`, `last_batch_id`, `last_result` | 최근 실행 시각, 묶음 번호, 결과 요약 |

실행 규칙
- 실행기(`runSchedules`)가 매시간 한 번 실행되어, 사용 중이고 현재 시각(시 단위)·요일·날짜가 맞으며 오늘 아직 실행하지 않은 예약을 발송합니다. 따라서 해당 시각의 0\~59분 사이에 발송됩니다.
- 리포트 기간은 7-13의 기본값을 씁니다. 가상 데이터라 기준일(2026-11-16) 기준이며, 실제 운영에서는 `reportDate_` 함수를 오늘 날짜로 바꿉니다.
- 지금 실행은 실행기 설치 여부와 관계없이 동작하며, 최근 실행 기록을 남깁니다.

### 7-15. 이슈 (감지 · 처리 · 고객사 안내)

| 메서드 · 경로 | 설명 |
|---|---|
| `GET /v1/issues` | 이슈 목록. 미완료 먼저, 우선순위 · 처리 기한 순. 조회 조건: `client_id`, `status`, `type`, `priority`, `owner`, `is_overdue`, `notified`, `q`, `date_from`, `date_to`(감지일). `meta.summary`(건수 요약), `meta.owners`(담당자 목록) 포함 |
| `GET /v1/issues/{issue_id}` | 이슈 1건과 고객사 안내 메일 초안(`mail`: 받는 사람, 제목, 안내 문구, 본문 양식) |
| `POST /v1/issues/scan` | 지금 감지. 본문 `{ "by": "김다슬" }` |
| `POST /v1/issues/update` | 상태·담당자 변경, 처리 메모 추가. 본문 `{ "issue_ids": ["IS-00001"], "status": "DONE", "owner": "김다슬", "memo": "주소 회신 받아 출고", "by": "김다슬" }` |
| `POST /v1/issues/notify` | 고객사 안내 메일 발송. 본문 `{ "issue_ids": ["IS-00001"], "recipients": [...], "intro": "...", "by": "김다슬" }`. `recipients`·`intro`는 1건일 때만 사용 |

감지 기준

| 유형 (`type`) | 대상 | 감지 조건 | 우선순위 | 안내 받는 담당자 |
|---|---|---|---|---|
| `OUTBOUND_DELAY` 출고지연 | 주문 | 출고지연이고 아직 출고 전 | 높음 | 출고지연 현황 수신 담당자 |
| `HOLD` 출고보류 | 주문 | 출고보류 상태 | 높음 | 출고지연 현황 수신 담당자 |
| `DELIVERY_FAILED` 배송실패 | 운송장 | 배송실패 상태 | 높음 | 배송실패 현황 수신 담당자 |
| `LOW_STOCK` 재고 부족 | SKU | 가용재고 < 안전재고 | 가용재고 0 이하면 높음, 그 외 보통 | 안전재고 미달 알림 수신 담당자 |
| `EXPIRY_NEAR` 사용기한 임박 | SKU | 사용기한 180일 미만 | 90일 미만이면 보통, 그 외 낮음 | 사용기한 임박 알림 수신 담당자 |
| `INBOUND_ISSUE` 입고 이슈 | 입고 | 최근 7일 입고예정 중 이슈 있음 | 보통 | 입고 이슈 알림 수신 담당자 |

해당 리포트를 받는 담당자가 없는 화주사는 정산 전용 담당자를 뺀 나머지 담당자에게 안내합니다(`mail.to_basis`에 표시).

처리 규칙

| 항목 | 규칙 |
|---|---|
| 등록 | 같은 대상(유형 + 대상 번호)은 이슈 1건만 둠. 완료한 이슈는 감지 조건이 한 번 해소된 뒤 다시 생겨야 새 이슈로 등록 |
| 자동 완료 | 미완료 이슈의 감지 조건이 사라지면 `감지 조건 해소로 자동 완료` |
| 상태 (`status`) | `OPEN` 미처리 → `IN_PROGRESS` 처리 중 → `DONE` 완료. 완료할 때 입력한 메모가 처리 결과(`resolution`)로 남음 |
| 처리 기한 (`due_at`) | 감지 시각 + 우선순위 높음 4시간, 보통 24시간, 낮음 72시간. 미완료인데 기한이 지나면 `is_overdue: true` |
| 담당자 (`owner`) | 직접 지정. 담당자가 없는 이슈를 처리하거나 안내하면 처리한 사람(`by`)이 담당자로 지정됨 |
| 고객사 안내 | 발송하면 `notified_at`, `notify_message_id`가 기록되고, 미처리 이슈는 처리 중으로 바뀜 |
| 이력 (`history`) | 감지, 상태·담당자 변경, 메모, 안내 발송, 자동 완료를 시각·처리자와 함께 기록 |
| 이슈 감지 알림 | 실행기의 감지에서 새 이슈가 생기면 `CX_INBOX`로 알림 메일 1통 발송(발송 구분 `digest`). 콘솔의 지금 감지는 알림 메일을 보내지 않음 |

---

## 8. 호출 예시

### 브라우저 주소창

```
웹앱URL?path=/v1/orders&status=HOLD&api_key=API키
```

### 웹페이지 JavaScript (조회)

```js
const BASE_URL = 'https://script.google.com/macros/s/배포ID/exec';
const API_KEY = '화면에서 입력받은 값';

async function getDelayedOrders(clientId) {
  const params = new URLSearchParams({ path: '/v1/orders', client_id: clientId, is_delayed: 'true', page_size: '100', api_key: API_KEY });
  const res = await fetch(`${BASE_URL}?${params}`);
  const json = await res.json();
  if (json.error) throw new Error(json.error.message); // HTTP 상태 코드가 아니라 error 필드로 판단
  return json.data;
}
```

### 웹페이지 JavaScript (메일 발송)

`Content-Type`을 `application/json`으로 보내면 브라우저가 사전 요청(CORS preflight)을 보내는데, 앱스 스크립트는 이를 처리하지 못해 요청이 실패합니다. 본문은 JSON 문자열로 만들되 `Content-Type`은 `text/plain`으로 보냅니다.

```js
async function sendMail(mail) {
  const res = await fetch(`${BASE_URL}?path=/v1/mail/send`, {
    method: 'POST',
    headers: { 'Content-Type': 'text/plain;charset=utf-8' },
    body: JSON.stringify({ ...mail, api_key: API_KEY }),
  });
  const json = await res.json();
  if (json.error) throw new Error(json.error.details ? json.error.details.join('\n') : json.error.message);
  return json.data;
}
```

### 전체 페이지 가져오기

목록은 최대 100건씩 나뉘어 옵니다. 전체가 필요하면 `meta.total_pages`까지 반복 조회합니다.

```js
async function getAll(path, filters = {}) {
  let page = 1, all = [];
  while (true) {
    const params = new URLSearchParams({ path, ...filters, page, page_size: 100, api_key: API_KEY });
    const json = await (await fetch(`${BASE_URL}?${params}`)).json();
    if (json.error) throw new Error(json.error.message);
    all = all.concat(json.data);
    if (page >= json.meta.total_pages) return all;
    page++;
  }
}
```

---

## 9. 리포트 종류별 조회 조건

리포트 종류마다 사용할 엔드포인트와 조회 조건입니다. 실제 요청 주소는 `웹앱URL?path=경로&조회조건&api_key=API키` 형태입니다. 예: `GET /v1/inventory?client_id=C001&is_low_stock=true` → `웹앱URL?path=/v1/inventory&client_id=C001&is_low_stock=true&api_key=API키` 받는 사람은 `GET /v1/clients`에서 해당 리포트 코드가 `receives`에 포함된 담당자입니다.

| 리포트 종류 | 엔드포인트 · 조회 조건 | 발송 주기 예시 |
|---|---|---|
| 일일 운영 요약 (`daily_summary`) | `GET /v1/reports/daily-summary?client_id=C001&date=2026-11-16` | 매일 18:00 |
| 출고지연 현황 (`outbound_delay`) | `GET /v1/outbounds?client_id=C001&is_delayed=true&date_from=2026-11-16&date_to=2026-11-16` + 미출고 건은 `GET /v1/orders?client_id=C001&is_delayed=true&status=ALLOCATED,PICKING,PICKED,PACKING,PACKED` | 매일 17:00 |
| 배송실패 현황 (`delivery_failed`) | `GET /v1/shipments?client_id=C001&status=DELIVERY_FAILED` | 매일 10:00 |
| 안전재고 미달 알림 (`stock_alert`) | `GET /v1/inventory?client_id=C001&is_low_stock=true` | 매일 09:00 |
| 사용기한 임박 알림 (`expiry_alert`) | `GET /v1/inventory?client_id=C001&is_expiry_near=true` | 매주 월요일 |
| 입고 이슈 알림 (`inbound_issue`) | `GET /v1/inbounds?client_id=C001&has_issue=true&date_from=2026-11-16&date_to=2026-11-16` | 입고완료 시 |
| 반품 처리 현황 (`return_status`) | `GET /v1/returns?client_id=C001&date_from=2026-11-09&date_to=2026-11-15` | 매주 월요일 |
| 월 정산서 (`settlement`) | `GET /v1/settlements?client_id=C001&period=2026-10` | 매월 5일 |
| 출고보류 확인 요청 | `GET /v1/orders?client_id=C001&status=HOLD` | 발생 시 |
| 주문 단건 문의 응대 | `GET /v1/orders/{order_id}` (진행 이력 포함) | 문의 시 |

---

## 10. 콘솔 화면

| 메뉴 | 화면 | 주요 기능 |
|---|---|---|
| 모니터링 | 대시보드 | 오늘 지표 9개, 확인 필요 6개 항목(이슈·목록 이동, 리포트 메일 작성, 항목별 미완료 이슈 건수), 최근 7일 추이 차트(표로 보기), 화주사별 오늘 현황(검색·정렬·행 클릭 시 화주사 선택) |
| 모니터링 | 이슈 처리 | 미처리·처리 중·처리 기한 초과·우선순위 높음·고객사 미안내 건수, 지금 감지, 상태·유형·우선순위·담당자·기한·안내 여부·검색어 조회, 여러 건 선택 후 담당자 지정·상태 변경·고객사 안내 발송. 행을 누르면 감지 내용·처리·고객사 안내 메일·이력 |
| 업무 조회 | 주문 · 출고 | 상태·출고지연·반품·판매채널·검색어·주문일 조회. 행을 누르면 주문 상세(진행 이력·배송 이력·반품) |
| 업무 조회 | 배송 | 배송 상태·배송실패 이력·택배사·출고일 조회 |
| 업무 조회 | 입고 | 입고 상태·이슈·입고예정일 조회. 행을 누르면 품목별 LOT번호·사용기한·수량 |
| 업무 조회 | 재고 | 안전재고 미달·사용기한 임박·보관·검색어 조회 |
| 업무 조회 | 반품 | 반품 상태·사유·처리 결과·접수일 조회 |
| 업무 조회 | 정산 | 정산월·상태 조회. 행을 누르면 정산 항목 |
| 화주사 커뮤니케이션 | 리포트 발송 · 개별 발송 | 리포트 종류·화주사·기간 선택 → 데이터 불러오기 → 받는 사람·제목·안내 문구 자동 작성 → 메일 미리보기 → 발송 |
| 화주사 커뮤니케이션 | 리포트 발송 · 일괄 발송 | 리포트 종류·기간·대상 화주사(검색·전체 선택·수신 담당자 있는 화주사만) 선택 → 발송 대상 확인 → 일괄 발송 → 화주사별 결과 |
| 화주사 커뮤니케이션 | 자동 발송 | 예약 추가·수정·삭제, 사용 켜고 끄기, 지금 실행, 최근 실행 결과. 실행기 설치 여부 표시 |
| 화주사 커뮤니케이션 | 발송 기록 | `mail_logs` 시트의 발송 기록. 리포트 종류·발송 구분(개별·일괄·자동·이슈 안내·이슈 감지 알림)별 조회 |
| 화주사 커뮤니케이션 | 화주사 · 담당자 | 화주사별 담당자와 수신 리포트 |

공통 동작
- 상단 **화주사 선택**: 이름이나 ID로 검색해 고르면 모든 화면이 그 화주사 기준으로 바뀝니다. **×**를 누르면 전체 화주사로 돌아갑니다.
- **공개 데모(조회 전용)**: 모든 화면 조회와 필터·정렬·미리보기는 할 수 있고, 메일 발송·이슈 처리 저장·예약 변경은 막힙니다. 막힌 버튼을 누르면 안내 창이 열립니다.
- **연결 설정**의 **내 이름**: 이슈 담당자 지정(나에게 지정)과 처리 이력의 처리자로 쓰입니다. 이 브라우저에만 저장됩니다.
- **새로고침**: 웹페이지에 저장해 둔 조회 결과를 지우고 다시 불러옵니다. 앱스 스크립트의 10분 캐시는 `clearCache`로 비웁니다.
- 주문 상세는 목록에 있는 정보를 바로 표시하고, 진행 이력·배송 이력은 이어서 불러옵니다. 처음 여는 주문은 10초 안팎 걸립니다.

리포트 발송 규칙
- 받는 사람: 선택한 화주사 담당자 중 해당 리포트를 수신하는 담당자가 자동으로 입력됩니다. 직접 추가·삭제할 수 있습니다.
- 제목: `[리포트 종류] 화주사 기간` 형식으로 자동 작성됩니다.
- 본문: 인사말, 안내 문구, 표(최대 30건), 맺음말 순서입니다. 안내 문구는 직접 고칠 수 있습니다.
- 첨부: 전체 목록을 CSV로 첨부합니다. 안내할 내역이 0건이면 표와 첨부를 생략하고 "…없습니다" 문구로 바뀝니다.
- 발송 전 확인 창에서 제목·받는 사람·메일 모드를 한 번 더 보여 줍니다.
- 일괄 발송은 화주사마다 위 규칙으로 메일을 따로 작성합니다. 받는 사람과 안내 문구는 수정할 수 없으며, 대시보드의 '확인 필요'에서 **메일 작성**을 누르면 화주사를 선택하지 않은 상태에서는 일괄 발송 탭으로 이동합니다.
- 일괄 발송은 **발송 대상 확인**으로 화주사별 건수와 발송 예정 여부를 먼저 계산한 뒤에만 보낼 수 있습니다. 조건을 바꾸면 다시 확인해야 합니다.
- 일괄·자동 발송에서 '0건 제외'를 켜면 안내할 내역이 없는 화주사는 보내지 않습니다. 알림형 리포트(출고지연, 배송실패, 안전재고, 사용기한, 입고 이슈)는 기본으로 켜져 있습니다.

---

## 11. 로컬 가상 API(`server.js`)와 다른 점

조회 조건, 응답 필드, 상태 코드는 로컬 가상 API와 같습니다. 다른 점은 아래와 같습니다.

| 항목 | 로컬 가상 API | 앱스 스크립트 |
|---|---|---|
| 주소 | `http://localhost:3000/v1/orders` | `웹앱URL?path=/v1/orders` |
| API 키 | 요청 헤더 `X-API-Key` | 주소의 `api_key`, 발송은 본문의 `api_key` |
| 발송 요청 Content-Type | `application/json` | `text/plain;charset=utf-8` |
| 오류 판단 | HTTP 상태 코드 | 응답의 `error` 필드 (`error.status`) |
| 메일 발송 | 기록만 저장(샌드박스) | Gmail로 실제 발송(테스트 모드 기본) |
| 발송 상태 값 | `SANDBOX_SENT` | `SENT` |
| 발송 기록 보관 | 서버 메모리(재시작 시 초기화) | `mail_logs` 시트 |
| 데이터 수정 | 코드 수정 | 구글 시트에서 직접 수정 |

---

## 12. 문제 해결

| 증상 | 확인할 것 |
|---|---|
| 브라우저에 구글 로그인 화면이나 '권한 필요'가 나옴 | 배포 설정의 액세스 권한이 `모든 사용자`인지 확인 |
| 코드를 고쳤는데 반영되지 않음 | 2-6의 방법으로 새 버전 배포 |
| `UNAUTHORIZED` | 스크립트 속성 `API_KEY`와 요청의 `api_key`가 같은지 확인 |
| `SHEET_NOT_FOUND` | 시트 탭 이름이 바뀌지 않았는지 확인. `checkSetup` 실행 |
| 발송 요청만 `Failed to fetch` | `Content-Type`이 `text/plain;charset=utf-8`인지 확인 |
| 웹페이지에서 `Failed to fetch` | 경로를 `?path=/v1/…`로 보내는지 확인. 웹 앱 URL 뒤에 `/v1/…`를 붙이면 차단됨. 웹 앱 URL이 `…/exec`로 끝나는지도 확인 |
| 메일이 오지 않음 | 스팸함 확인. `mail_logs` 시트에 기록이 있는지, `actual_to`가 맞는지 확인 |
| 응답이 느림 | 처음 조회는 시트를 읽느라 4\~12초 걸림. 같은 조회는 10분간 캐시되어 1\~2초 |
| 시트를 고쳤는데 반영되지 않음 | 응답 캐시(10분) 때문. 편집기에서 `clearCache` 실행 |
| 예약 시각이 지났는데 발송되지 않음 | 자동 발송 화면에 '실행기 꺼짐'이 보이면 2-7의 `installScheduler` 실행. 예약의 사용 여부와 오늘 이미 실행했는지(최근 실행) 확인 |
| 일괄 발송이 3곳만 발송됨 | 테스트 모드의 정상 동작. 스크립트 속성 `TEST_BULK_LIMIT`로 개수 변경 |
| 일괄 발송에 '수신 담당자 없음'이 나옴 | 그 화주사에 해당 리포트를 받는 담당자가 없음. `contacts` 시트의 `receives` 확인 |
| 이슈 처리 화면이 비어 있음 | 아직 감지하지 않은 상태. **지금 감지**를 누르거나 2-7의 실행기를 켬 |
| 이슈 이력의 처리자가 '이름 미입력' | 연결 설정에서 내 이름을 입력 |
| 이슈 감지 알림 메일이 오지 않음 | 새 이슈가 생긴 감지에서만 발송됨. 가상 데이터는 기준 시각이 고정이라 첫 감지 이후에는 새 이슈가 생기지 않음 |

---

## 13. 구조의 한계와 실서비스 전환안

이 프로젝트는 별도 서버 없이 구글 시트·앱스 스크립트·GitHub Pages로 만든 포트폴리오용 구조입니다. 실제 운영에 쓰려면 아래 한계를 바꿔야 합니다.

| 항목 | 현재 구조의 한계 | 실서비스 전환안 |
|---|---|---|
| 데이터 | 기준 시각 고정 가상 데이터. 구글 시트 행 수가 늘면 조회가 느려짐 | WMS·OMS 데이터베이스와 연동하고, 조회용 데이터베이스(예: PostgreSQL)를 둠 |
| 속도 | 첫 조회 4\~12초. 같은 조회는 10분 캐시로 1\~2초 | 서버 API와 인덱스로 1초 이내 응답 |
| 인증·권한 | 전체 키와 조회 전용 키 2종만 구분. 누가 조회·발송했는지는 화면에 입력한 이름에 의존 | 회사 계정 로그인(SSO), 사용자별 권한(조회·발송·설정), 접속·발송 감사 기록 |
| 메일 발송 | Gmail 하루 약 100명 한도. 대량 발송·수신 확인 불가 | 메일 발송 서비스(예: Amazon SES, SendGrid)로 한도 확장, 열람·반송 추적 |
| 이슈 감지 | 시간 트리거로 1시간마다 감지 | 데이터 변경 이벤트(출고 마감, 배송 상태 수신) 발생 시 즉시 감지 |
| 알림 채널 | 메일만 사용 | 메신저(예: 슬랙) 알림, 화주사 전용 조회 화면(포털) |
| 동시 작업 | 시트 전체를 다시 쓰는 방식이라 여러 명이 동시에 처리하면 대기 발생 | 이슈를 건별로 저장하는 데이터베이스와 낙관적 잠금 |
