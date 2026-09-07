# KRiC 철도데이터포털 — 교통약자정보 API 전수 목록 (19건)

조사일: 2026-09-07 · 출처: https://openapi.kric.go.kr/rips/M_01_02/intro.do?lcd=D

> 포털·API 모두 **https 정상**. 인증서 검증도 통과한다(`curl` 200 / `ssl_verify_result 0`).
> 간헐적으로 첫 요청이 실패할 수 있으니 그때는 재시도할 것.

## 공통 요청 파라미터

| 파라미터 | 의미 | 예시 |
|---|---|---|
| `serviceKey` | 발급받은 서비스키 | — |
| `format` | 응답 형식 | `json` / `xml` |
| `railOprIsttCd` | 철도운영기관 코드 | `S1`(서울교통공사) `AR`(공항철도) `KR`(코레일) `BS`(부산) `DG`(대구) |
| `lnCd` | 호선 코드 | `1`~`9`, `A1` 등 |
| `stinCd` | 역 코드 | `322`(불광) `321`/`331`(충무로) `152` 등 |
| `plfNo` | 승강장 번호 | `2` |
| `scarSqno` | 차량 순번 | `1` |
| `cpsTpCd` | 편성 유형 코드 | `162` |

서울 1~9호선은 `railOprIsttCd=S1` 로 조회. **역 코드 체계가 서울시 열린데이터광장의
`stnCd`/`stop_id` 와 다를 수 있으므로, 덤프의 `station_master` 와 매핑 테이블을 먼저 만들어야 함.**

---

## A. 실내 동선 — C2(실내 3D 라우팅) 핵심

| # | 서비스 | 엔드포인트 (`/openapi/` 이하) | 파라미터 | 활용신청수 |
|---|---|---|---|---|
| 1 | **환승 이동경로** | `vulnerableUserInfo/transferMovement` | railOprIsttCd, lnCd, stinCd, prevStinCd, chthTgtLn, chtnNextStinCd | 118 |
| 2 | 환승 이동경로 (표준) | `handicapped/transferMovement` | 위와 동일 | 58 |
| 3 | **출입구 승강장 이동경로** | `vulnerableUserInfo/stationMovement` | railOprIsttCd, lnCd, stinCd, nextStinCd | 40 |
| 4 | 출입구 승강장 이동경로 (표준) | `handicapped/stationMovement` | 위와 동일 | 70 |
| 5 | **교통약자 역사 내 엘리베이터 이동동선** | `trafficWeekInfo/stinElevatorMovement` | railOprIsttCd, lnCd, stinCd | 92 |
| 6 | 역사별 엘리베이터 이동동선 | `vulnerableUserInfo/stationElevatorMovement` | railOprIsttCd, lnCd, stinCd | 69 |
| 7 | 역사별 휠체어리프트 이동동선 | `vulnerableUserInfo/stationWheelchairLiftMovement` | railOprIsttCd, lnCd, stinCd | 72 |
| 8 | 역사별 휠체어리프트 위치 | `vulnerableUserInfo/stationWheelchairLiftLocation` | railOprIsttCd, lnCd, stinCd | 72 |
| 9 | 역사별 승강장 정보 | `convenientInfo/stPlf` | railOprIsttCd, lnCd, stinCd | 46 |

**#1/#2, #3/#4 는 "일반 / 표준" 이 쌍으로 존재한다.** 응답 스키마가 다를 가능성이 높으니
같은 역으로 둘 다 호출해 비교한 뒤 하나를 채택할 것.
**#9 `stPlf` 는 수정일이 2026-05-11 로 유일하게 최근 갱신본** — 나머지는 2019~2020년 고정.

## B. 승강장 위험도 — C3(연단간격 선형참조) 핵심

| # | 서비스 | 엔드포인트 | 파라미터 | 활용신청수 |
|---|---|---|---|---|
| 10 | **역사별 승강장 이격거리** | `vulnerableUserInfo/stationPlatformTrainDistance` | railOprIsttCd, lnCd, stinCd, plfNo | 40 |
| 11 | 역사별 안전발판 설치유무 | `vulnerableUserInfo/stationSafetyPlatform` | railOprIsttCd, lnCd, stinCd, plfNo | 32 |
| 12 | 역사별 인접 승강기 차량번호 | `vulnerableUserInfo/stationElevatorCarNumber` | railOprIsttCd, lnCd, stinCd | 23 |
| 13 | 역사별 인접 계단 차량번호 | `vulnerableUserInfo/stationStairCarNumber` | railOprIsttCd, lnCd, stinCd | 22 |

**#12/#13 이 C3의 결정적 데이터** — "몇 번째 칸이 승강기/계단과 가까운가"를 주므로
`LineString(M)` 승강장 모델에 M값 이벤트로 바로 붙는다.

## C. 역사 편의시설

| # | 서비스 | 엔드포인트 | 파라미터 | 활용신청수 |
|---|---|---|---|---|
| 14 | 역사별 장애인 화장실 위치 | `vulnerableUserInfo/stationDisabledToilet` | railOprIsttCd, lnCd, stinCd | 111 |
| 15 | 역사별 점자표시유무 | `vulnerableUserInfo/stationBrailleDisplays` | railOprIsttCd, lnCd, stinCd | 40 |

## D. 차량 — B1(경로탐색) 탑승칸 추천용

| # | 서비스 | 엔드포인트 | 파라미터 | 활용신청수 |
|---|---|---|---|---|
| 16 | 차량별 휠체어 승차가능 차량정보 | `vulnerableUserInfo/trainWheelchairBoardPossible` | railOprIsttCd, scarSqno, cpsTpCd | 21 |
| 17 | 차량별 휠체어 안전벨트 유무 | `vulnerableUserInfo/trainWheelchairSeatBelt` | railOprIsttCd, scarSqno, cpsTpCd | 8 |
| 18 | 차량별 노약자 좌석 유무 | `vulnerableUserInfo/trainPrioritySeat` | railOprIsttCd, scarSqno, cpsTpCd | 16 |
| 19 | 차량별 임산부 좌석 유무 | `vulnerableUserInfo/trainSeatPregnantWoman` | railOprIsttCd, scarSqno, cpsTpCd | 19 |

---

## 샘플 호출 (키 발급 후 그대로 사용 가능)

```
# 충무로역 3호선 → 4호선 환승 이동경로
https://openapi.kric.go.kr/openapi/vulnerableUserInfo/transferMovement?serviceKey=KEY&format=json&railOprIsttCd=S1&lnCd=3&stinCd=321&prevStinCd=422&chthTgtLn=4&chtnNextStinCd=424

# 불광역 출입구→승강장 이동경로
https://openapi.kric.go.kr/openapi/vulnerableUserInfo/stationMovement?serviceKey=KEY&format=json&railOprIsttCd=S1&lnCd=3&stinCd=322&nextStinCd=323

# 불광역 엘리베이터 이동동선
https://openapi.kric.go.kr/openapi/trafficWeekInfo/stinElevatorMovement?serviceKey=KEY&format=json&railOprIsttCd=S1&lnCd=3&stinCd=322
```

## 상태

- [ ] KRiC 회원가입 — **미확인** (활용신청 클릭 시 JS 모달 발생, 로그인 요구로 추정)
- [ ] 활용신청 — API별 개별 신청 방식으로 보임 (19건 전부 신청 필요할 수 있음)
- [ ] 서비스키 발급
- [ ] `station_master` ↔ KRiC `stinCd` 매핑 테이블 작성

## 신청 우선순위

전부 신청하되, 승인이 부분적으로만 날 경우를 대비한 순서:
**1 → 5 → 3 → 10 → 12 → 13 → 9 → 8 → 14 → 나머지**

---

# 서울 열린데이터광장 — 조사 메모 (2026-09-07)

## 1. 「서울시 경사도」의 실체 — A1 설계가 확정됨

검색 결과에 나온 데이터셋 설명 원문:

> "요청한 경사도 shp은 없고, **경사도를 추출할 수 있는 표고점, 등고선 shp파일** 제공입니다."

즉 **완성된 경사도 래스터는 제공되지 않는다.** 표고점(spot height) + 등고선(contour) 벡터만 준다.
따라서 A1의 `등고선 → TIN/IDW 보간 → DEM → Horn 경사 계산` 파이프라인은
선택이 아니라 **반드시 직접 수행해야 하는 필수 공정**이다. 표고점이 같이 오므로
등고선만 쓸 때보다 보간 품질이 올라간다 (breakline + mass point 조합).

- 데이터셋 ID: `OA-22241` · 제공유형 **FILE** · 수정일 2025-03-20
- 제공부서: 도시공간본부 도시계획상임기획과

## 2. 노션 문서에 없던 추가 데이터셋 (검색 중 발견)

| 데이터셋 | 유형 | 수정일 | 쓸 곳 |
|---|---|---|---|
| **서울교통공사 지하철 역사 엘리베이터 및 에스컬레이터 길이·높이 정보** | SHEET / OpenAPI / FILE | 2026-04-21 | **C2 수직 이동비용, C4 3D 단면** — 길이·높이 실측값이라 심도 추정을 대체 가능 |
| 서울교통공사_휠체어경사로 설치 현황 | SHEET / FILE | 2024-11-01 | C2 실내 경사로 에지 |
| 서울시 보행자 출입구 정보 | SHEET / OpenAPI / FILE | 2023-04-21 | A2 보행 네트워크 ↔ 출입구 접합 (※ 일회성 구축, 현행화 없음) |
| 서울시 장애인편의시설 목록정보 (한국사회보장정보원) | SHEET / OpenAPI | 2026-09-07 | A3 시설 공급면 (전국 단위) |

**EV/ES 길이·높이 데이터가 특히 중요하다.** 기존 설계에서는 「역사 심도정보」로 수직 부담을
추정하려 했는데, 이건 승강기별 실제 높이를 주므로 C2 비용함수가 추정이 아니라 실측이 된다.

## 3. 포털 접근 방식 — 자동화 제약

- **데이터셋 상세 페이지는 직접 URL 접근이 차단된다.** `/dataList/OA-22241/F/1/datasetView.do`
  로 직접 GET 하면 "URL 오류 안내"가 뜬다. Referer를 붙여도(페이지 내부 `location.href`) 동일하게 차단.
- 검색 결과의 링크는 `<a class="goView" data-rel="OA-22241/F/1/datasetView.do" href="#">` 형태로,
  jQuery 클릭 핸들러가 처리한다. 핸들러가 별도 POST/세션 처리를 하는 것으로 보인다.
- 검색창 Enter 입력은 **간헐적으로 키워드가 반영되지 않고** 전체 8,249건이 나온다. 재시도 필요.
- 폼을 JS로 직접 `submit()` 하면 히든 필드가 세팅되지 않아 `알 수 없는 오류 (105)` 발생.

**결론: 이 포털은 브라우저 자동화로 긁는 것보다 OpenAPI 인증키를 받아 API로 가져오는 편이 훨씬 안정적이다.**
인증키 발급에는 로그인이 필요하다.

