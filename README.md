# INCOTO (투자관제탑) 프로젝트 전체 분석 정리

> 작성일: 2026-05-02  
> 분석 대상: 2015~2017년 재무 투자성과 평가 시스템 (1차 ~ 3차 개발)

---

## 1. 프로젝트 개요

| 항목        | 내용                                             |
| --------- | ---------------------------------------------- |
| **프로젝트명** | INCOTO (Investment Control Tower) — 투자관제탑      |
| **목적**    | 재무설계사가 고객의 자산·투자 포트폴리오를 통합 관리하고 성과를 분석하는 웹 시스템 |
| **발주처**   | 개인 재무설계 전문가 (외주 수주)                            |
| **개발이력**  | 1차(2015) → 2차(2016) → 3차 추가개발 및 유지보수(2017)     |
| **배포환경**  | Cafe24 호스팅, Tomcat 7                           |
|           |                                                |

---

## 2. 기술 스택

| 분류 | 기술 |
|------|------|
| **언어** | Java 1.6 |
| **프레임워크** | Spring MVC 3.2.4 |
| **ORM** | MyBatis 3.2.2 |
| **DB** | MySQL 5.6 |
| **서버** | Tomcat 7 |
| **프론트엔드** | JSP, Bootstrap, jQuery 2.1.3 |
| **빌드** | Maven |
| **버전관리** | SVN (Subversion) |
| **IDE** | Eclipse (STS) |
| **암호화** | SEED Algorithm + Base64 |
| **OS(개발)** | Windows 8 / JDK 7 / Chrome |

---

## 3. 아키텍처 구조

```
[Browser] → JSP
    ↕ Ajax / Form Submit
[Controller] (Spring MVC)
    ↕ Service Layer
[Service / ServiceImpl]
    ↕ DAO Layer
[DataAccessObject] + MyBatis XML Mapper
    ↕
[MySQL DB]
```

### 패키지 구조 (net.su.*)

```
net.su
├── login/               ← 로그인, 회원관리, 게시판(BBS), SEED 암호화
├── customer/            ← 고객, 가족구성원, 목표, 설계자 관리
├── asset/               ← 자산 전체 CRUD (8종 자산유형)
├── cusPage/             ← 고객 전용 페이지
└── output/              ← 분석 및 출력 화면
    ├── fcr/             ← FCR(미래현금소요) 분석
    ├── ivstAct/         ← 투자매트릭스
    ├── ivstcrnstn/      ← 투자현황 (통장, 지출, 세금, 현금흐름)
    ├── ivstDtl/         ← 투자상세 테이블
    ├── ivstRslt/        ← 투자결과 (지속투자, 자체투자)
    ├── subData/         ← 보조데이터 (집계시트, 카드지급, 재무추세, 입출금추적)
    └── taa/             ← TAA(전술적 자산배분)
```


---

## 4. DB 구조 

### 4-1. 사용자 / 고객 관리

#### mem_tb (회원 — 관리자 계정)
| 컬럼명 | 설명 |
|--------|------|
| mem_seq | PK |
| mem_id | 아이디 |
| mem_pw | 비밀번호 (SEED 암호화) |
| mem_nme | 이름 |
| mem_cntact | 연락처 |
| memo | 메모 |

#### cus_tb (고객)
| 컬럼명 | 설명 |
|--------|------|
| cus_seq | PK |
| cus_fmy_nme | 가족(고객) 이름 |
| cus_file_box | 파일박스 경로 |
| cus_use_at | 사용여부 |
| cus_cal_date | 계산 기준일 |

#### fmy_mem_tb (가족구성원)
| 컬럼명 | 설명 |
|--------|------|
| fmy_mem_seq | PK |
| cus_seq | FK → cus_tb |
| fmy_mem_nme | 이름 |
| fmy_mem_role | 가족관계 (본인/배우자/자녀 등) |
| fmy_mem_birth | 생년월일 |

#### splist_tb (설계자 / 담당자)
| 컬럼명 | 설명 |
|--------|------|
| splist_seq | PK |
| cus_seq | FK → cus_tb |
| splist_nme | 설계자 이름 |
| splist_cmpn | 소속 회사 |
| splist_cntact | 연락처 |

#### cus_account_tb (고객 계좌 — 3차 추가)
고객의 은행 계좌 정보 관리

#### cus_memo_tb (고객 메모 — 3차 추가)
고객 관련 상담/메모 내용 저장

#### bbs_tb (게시판)
| 컬럼명 | 설명 |
|--------|------|
| bbs_seq | PK |
| bbs_title | 제목 |
| bbs_content | 내용 |
| bbs_date | 작성일 |
| mem_id | 작성자 |

---

### 4-2. 자산 관리 (핵심)

#### asset_tb (자산 마스터 — 모든 자산의 공통 정보)
| 컬럼명 | 설명 |
|--------|------|
| asset_seq | PK |
| cus_seq | FK → cus_tb |
| asset_nme | 자산명 |
| asset_type_seq | FK → asset_type_tb (유형 코드) |
| asset_ctrt_date | 계약일 |
| ctrt_fmy_mem | 계약자 가족구성원 |
| splist_seq | FK → splist_tb (담당 설계자) |
| asset_use_at | 사용여부 |
| asset_keep | 유지여부 (현재 보유 중인지) |

#### asset_type_tb (자산유형 코드)
| seq | 자산유형 |
|-----|---------|
| 1 | 유동성 (입출금 통장) |
| 2 | 현금성 (예금/적금) |
| 3 | 채권형 (채권/채권형 펀드) |
| 4 | 주식형 (주식/주식형 펀드) |
| 5 | 부동산 |
| 6 | 기타자산 |
| 7 | 대출 |
| 8 | 보험 |
| 9 | 신용카드 |

#### liqdty_tb (유동성 자산 — 입출금 통장)
| 컬럼명 | 설명 |
|--------|------|
| asset_seq | PK + FK → asset_tb |
| bnkbok_goal_seq | 통장 목표 번호 |
| mtrx_type_seq | FK → mtrx_type_tb |
| strtg_seq | FK → strtg_tb (전략) |
| liqdty_dome_fore_div | 국내/해외 구분 |
| liqdty_nation_div | 투자 국가 |
| liqdty_finac_instn_nme | 금융기관명 |
| liqdty_finac_instn_cntact | 금융기관 연락처 |
| liqdty_finac_instn_calctr | 담당자 |
| liqdty_acont_num | 계좌번호 |
| liqdty_last_mon_balnc | 전월 잔액 |
| liqdty_this_mon_balnc | 당월 잔액 |
| liqdty_minus_loan_ra | 마이너스 대출 비율 |

#### ivst_asset_tb (투자자산 공통 — cash/bond/stock 공통 추가 정보)
| 컬럼명 | 설명 |
|--------|------|
| asset_seq | PK + FK → asset_tb |
| goal_seq | FK → goal_tb (연결 목표) |
| mtrx_type_seq | FK → mtrx_type_tb |
| strtg_seq | FK → strtg_tb |
| bm_type_seq | FK → bm_type_tb (벤치마크) |
| ivst_asset_dome_fore_div | 국내/해외 구분 |
| ivst_asset_nation_div | 투자 국가 |
| ivst_asset_finac_instn_nme | 금융기관명 |
| ivst_asset_acont_num | 계좌번호 |
| ivst_asset_spd_div | 지출 구분 |
| ivst_asset_spd_method | 지출 방법 |
| ivst_asset_spd_asset | 지출 자산 |
| ivst_asset_mon_mony | 월 납입금 |
| ivst_asset_tmpry_mony | 임시 납입금 |
| ivst_asset_reduce_tax_type | 절세 유형 |
| ivst_asset_profit_fmy_mem | 수익자 가족구성원 |
| ivst_asset_pay_end_date | 납입 만기일 |
| ivst_asset_ctrt_end_date | 계약 만기일 |
| ivst_asset_irr | 내부수익률 (IRR) |
| ivst_asset_aprs_mony | 평가금액 |

#### cash_tb (현금성 — 예금/적금)
| 컬럼명 | 설명 |
|--------|------|
| asset_seq | PK + FK |
| cash_end_date_mony | 만기 수령금액 |
| cash_intrst_ra | 금리 |

#### bond_tb (채권형)
| 컬럼명 | 설명 |
|--------|------|
| asset_seq | PK + FK |
| bond_end_date_mony | 만기 금액 |
| bond_intrst_ra | 금리 |
| bond_main_insur_fmy_mem | 주 피보험자 |
| bond_sub_insur1/2_fmy_mem | 부 피보험자 |
| bond_pnsin_start_date | 연금 개시일 |

#### stock_tb (주식형)
| 컬럼명 | 설명 |
|--------|------|
| asset_seq | PK + FK |
| stock_main_insur_fmy_mem | 주 피보험자 |
| stock_sub_insur1/2_fmy_mem | 부 피보험자 |
| stock_pnsin_start_date | 연금 개시일 |

#### ivst_dtl_tb (투자상세 — 매수/매도 내역)
| 컬럼명 | 설명 |
|--------|------|
| ivst_dtl_seq | PK |
| asset_seq | FK → asset_tb |
| ivst_dtl_date | 투자일 |
| ivst_dtl_mon_mony | 월납 금액 |
| ivst_dtl_tmpry_mony | 임시납 금액 |
| ivst_dtl_add_mony | 추가납입 금액 |
| ivst_dtl_mon_biz_ra | 월납 사업비율 |
| ivst_dtl_tmpry_biz_ra | 임시납 사업비율 |
| ivst_dtl_add_biz_ra | 추가납 사업비율 |
| ivst_dtl_resale_mony | 환매(매도) 금액 |

#### realest_etc_tb (부동산 / 기타자산)
| 컬럼명 | 설명 |
|--------|------|
| asset_seq | PK + FK |
| goal_seq | FK → goal_tb |
| realest_etc_mon_mony | 월 납입금 |
| realest_etc_tmpry_mony | 임시납 |
| realest_etc_spd_div | 지출 구분 |
| realest_etc_spd_method | 지출 방법 |
| realest_etc_spd_asset | 지출 자산 |
| realest_etc_ctrt_end_date | 계약 만기일 |
| realest_etc_reduce_tax_type | 절세 유형 |
| realest_etc_aprs_mony | 평가금액 |

#### realest_etc_dtl_tb (부동산/기타 거래내역)
| 컬럼명 | 설명 |
|--------|------|
| asset_seq | FK |
| realest_etc_dtl_tmpry_mony | 임시납 |
| realest_etc_dtl_add_mony | 추가납 |
| realest_etc_dtl_resale_mony | 환매금 |

#### loan_tb (대출)
| 컬럼명 | 설명 |
|--------|------|
| asset_seq | PK + FK |
| goal_seq | FK → goal_tb |
| loan_finac_instn_nme | 금융기관명 |
| loan_acont_num | 계좌번호 |
| loan_mon_mony | 월 상환액 |
| loan_loan_mony | 대출 원금 |
| loan_balnc | 현재 잔액 |
| loan_repaymt_method | 상환 방식 |
| loan_fixed_intrst | 고정금리 여부 |
| loan_intrst | 대출 금리 |
| loan_wad_type | 인출 유형 |
| loan_wad_objt | 인출 목적 |
| loan_ctrt_end_date | 만기일 |
| loan_dfrmt_end_date | 거치 만료일 |

#### insur_tb (보험)
| 컬럼명 | 설명 |
|--------|------|
| asset_seq | PK + FK |
| goal_seq | FK → goal_tb |
| insur_finac_instn_nme | 보험사명 |
| insur_scrts_num | 증권번호 |
| insur_mon_mony | 월 보험료 |
| insur_tmpry_mony | 임시납 |
| insur_reduce_tax_type | 절세 유형 |
| insur_profit_fmy_mem | 수익자 |
| insur_pay_end_date | 납입 만기일 |
| insur_ctrt_end_date | 계약 만기일 |
| insur_end_date_mony | 만기 수령금 |
| insur_intrst_ra | 금리 |
| insur_main/sub_insur_fmy_mem | 주/부 피보험자 |
| insur_pnsin_start_date | 연금 개시일 |

#### crdt_cad_tb (신용카드)
| 컬럼명 | 설명 |
|--------|------|
| asset_seq | PK + FK |
| crdt_cad_finac_instn_nme | 카드사명 |
| crdt_cad_num | 카드번호 |
| crdt_cad_balnc | 잔액(한도 잔여) |
| crdt_cad_ctrt_end_date | 만기일 |

---

### 4-3. 목표 관리

#### goal_tb (목표 마스터 — 공통 16가지 목표 코드)
목돈마련, 연금플랜, 결혼자금, 주택마련, 교육자금, 사업자금, 여행자금, 의료비,
부동산구입, 자동차구입, 생활비마련, 비상예비금, 증여, 상속, 기부, 기타

#### goal_dtl_tb (고객별 목표 상세)
| 컬럼명 | 설명 |
|--------|------|
| goal_dtl_seq | PK |
| cus_seq | FK → cus_tb |
| goal_seq | FK → goal_tb (목표 유형) |
| goal_dtl_nme | 목표 이름 (고객 지정) |
| goal_dtl_now_mony | 현재 보유 금액 |
| goal_dtl_need_date | 목표 달성 필요 시점 |
| goal_dtl_goal_mony | 목표 금액 |
| goal_dtl_need_mony | 필요 금액 (부족분) |

#### bnkbok_goal_tb (통장 목표)
유동성 자산(통장)을 특정 목표에 연결하여 목표별 잔액 추적

---

### 4-4. 전략 / 분석 관련

#### strtg_tb (투자전략)
| 컬럼명 | 설명 |
|--------|------|
| strtg_seq | PK |
| cus_seq | FK → cus_tb |
| strtg_type | 유지 / 비율조정 / 비율기반금액조정 / 기타 |
| strtg_nme | 전략명 (예: 유지, 변액ERR, 펀드ERR, Golden Cross 등) |
| strtg_smary | 전략 설명 |

#### mtrx_type_tb (매트릭스 유형 코드)
자산 분류 기준:
- 요구불 / 예금 / 적금
- 단기/중기/장기 × 상등/중등/하등
- 대형/중형/소형 × 가치/혼합/성장

#### bm_type_tb (벤치마크 유형)
| 컬럼명 | 설명 |
|--------|------|
| bm_type_seq | PK |
| asset_type_seq | 해당 자산유형 |
| bm_type_dome_fore_div | 국내/해외 |
| bm_type_nme | 지수명 |

지수 예시: KRX지수, 기타(현금), 코스피, 다우존스, 상해종합,
러시아/브라질/인도/일본 지수 등

#### bm_dtl_tb (벤치마크 상세 수치)
날짜별 벤치마크 지수 데이터 (공동 활용 데이터로 관리)

#### ratio_dtl_tb (비율 상세)
| 컬럼명 | 설명 |
|--------|------|
| asset_seq | FK |
| ratio_dtl_fund_nme | 펀드명 |
| ratio_dtl_adjst_before | 조정 전 비율 |
| ratio_dtl_adjst_after | 조정 후 비율 |

#### ra_adjst_asset_dtl (비율조정 자산 상세)
자산별 목표 비율 조정 내역

#### mony_adjst_tb (금액 조정)
금액 기반 조정 내역

#### case_cst_tb (경우의수 비용)
시나리오별(경우의수) 비용 계산

---

### 4-5. 지출 관리

#### spd_type_tb (지출 유형 코드)
| 코드 | 유형 |
|------|------|
| 1 | 저축/투자 |
| 2 | 대출상환 |
| 3 | 보험 |
| 4 | 주거비 |
| 5 | 차량/교통 |
| 6 | 정보통신 |
| 7 | 교육 |
| 8 | 자기계발 |
| 9 | 어른지원 |
| 10 | 용돈 |
| 11 | 의료 |
| 12 | 기부 |
| 13 | 경조사 |
| 14 | 계비 |
| 15 | 기타생활비 |
| 16 | 연간단위 |
| 17 | 사업경비 |

#### spd_dtl_tb (지출 상세)
| 컬럼명 | 설명 |
|--------|------|
| spd_dtl_seq | PK |
| cus_seq | FK → cus_tb |
| spd_type_seq | FK → spd_type_tb |
| spd_dtl_nme | 지출 항목명 |
| spd_dtl_mony | 금액 |
| spd_dtl_date | 날짜 |
| spd_dtl_io_div | 입금/출금 구분 |
| mon_div_date | 월 구분 기준일 |

#### anu_spd_dtl_tb (연간지출 상세)
연간 단위 지출 항목 (보너스, 연간보험료 등)

#### reglr_cst_tb / reglr_cst_col_tb (정기비용 / 정기비용 집계)
월별 고정 정기비용 및 집계 관리

---

### 4-6. 기타 테이블

| 테이블 | 설명 |
|--------|------|
| `financial_trend_tb` | 재무 추세 데이터 (시계열 재무지표) |
| `dump_tb` | 시점별 데이터 스냅샷 *(3차 추가)* |

---

### 4-7. 주요 View (뷰)

| 뷰명 | 설명 |
|------|------|
| `view_asset_total` | 모든 자산유형(liqdty/cash/bond/stock/realest/loan/insur/crdt_cad)을 UNION으로 통합한 전체 자산 뷰 |
| `ivst_dtl_sum` | 투자상세 집계 — 자산별 월납/임시납/추가납/사업비/환매/실질투자금 합계 |
| `realest_etc_dtl_sum` | 부동산/기타자산 거래내역 집계 |
| `total_dtl_sum` | 전체 자산 통합 집계 뷰 (투자자산 + 부동산 통합) |
| `asset_incr` / `asset_incrs` | 자산 증가 현황 뷰 (asset_incrs는 유지 자산만 필터) |

---

## 5. 메뉴 구조 및 기능 설명

```
[로그인]
    ↓
[홈 대시보드]
    │
    ├── 고객 관리
    │   ├── 고객 목록 / 등록 / 수정 / 삭제
    │   ├── 가족구성원 관리
    │   ├── 목표 설정 (16가지 인생목표 + 고객별 목표금액/시점)
    │   └── 담당 설계자 정보
    │
    ├── 자산 관리
    │   ├── 자산 목록 (assetList — 전체 자산 현황)
    │   ├── 유동성 자산 (입출금 통장 — 잔액, 마이너스대출)
    │   ├── 현금성 자산 (예금/적금 — 금리, 만기금액)
    │   ├── 채권형 (채권/채권형펀드 — 투자상세 매수/매도)
    │   ├── 주식형 (주식/주식형펀드 — 투자상세 매수/매도)
    │   ├── 부동산 / 기타자산
    │   ├── 대출 (대출금액, 금리, 상환방식)
    │   ├── 보험 (납입, 만기수령, 피보험자)
    │   ├── 신용카드
    │   └── 벤치마크 등록 / 수정 (지수별 데이터 관리)
    │
    ├── 투자현황 분석 (output/ivstcrnstn)
    │   ├── 통장현황 (IoBnkbokCrntSt) — 전체 입출금 통장 잔액 현황
    │   ├── 지출현황 (SpdAcct) — 지출 계정별 현황 요약
    │   ├── 지출지급 (SpdPay) — 지출 항목별 지급 내역
    │   ├── 세금체크 (TaxChecking) — 절세/세금 계산
    │   └── 현금흐름체크 (CashFlowChecking) — 월별 현금흐름 분석
    │
    ├── 투자결과 (output/ivstRslt)
    │   ├── 지속투자결과 (CntinuIvstRslt) — 현재 전략 유지 시 미래 결과 분석
    │   └── 자체투자결과 (OwnIvstRslt) — 고객 자체 운용 실적
    │
    ├── FCR 분석 (output/fcr)
    │   └── 미래현금소요(Future Cash Requirement)
    │       — 목표 달성을 위해 향후 필요한 현금 소요량 계산
    │       — 부족분 시각화 및 상품별 추천
    │
    ├── 투자매트릭스 (output/ivstAct)
    │   └── 자산유형 × 투자등급 매트릭스 분석 (채권형/주식형 포트폴리오 구성 시각화)
    │
    ├── TAA (output/taa)
    │   └── 전술적 자산배분 (Tactical Asset Allocation)
    │       — 현재 자산배분 vs 목표 자산배분 비교 및 리밸런싱 제안
    │
    ├── 투자상세 테이블 (output/ivstDtl)
    │   └── 자산별 매수/매도 전체 내역 통합 테이블 조회
    │
    ├── 보조데이터 (output/subData)
    │   ├── 투자현황표 (IvstStatusTable) — 전체 자산 현황 요약표
    │   ├── 집계시트 (CollectSheet) — 통합 집계 (엑셀 출력 지원)
    │   ├── 카드지급 (CardPay) — 신용카드 지급 내역
    │   ├── 날짜관리 (DateMng) — 기준일 관리 (totalDateMng, 엑셀 출력 지원)
    │   ├── 재무추세 (FinancialTrend) — 시계열 재무지표 추세
    │   └── 입출금추적 (InOutTracking) — 상품별 입출금 추적
    │
    ├── 고객 전용 페이지 (cusPage)
    │   └── 고객이 직접 본인 포트폴리오를 조회하는 화면
    │
    └── 게시판 / 공지사항 (BBS)
```

---

## 6. 핵심 비즈니스 로직

### 6-1. FCR (Future Cash Requirement — 미래현금소요)
- 고객의 인생 목표별(결혼, 주택, 은퇴 등) 필요 금액과 현재 적립 상황 비교
- 부족분(FCR) 계산 → 어떤 상품에 얼마를 더 투자해야 하는지 제시
- `AssetIncrGoalValueObject`, `AssetIncrProductValueObject`, `FcrValueObject` 등 복잡한 계산 객체 사용

### 6-2. IRR (Internal Rate of Return — 내부수익률)
- 투자자산별 실제 IRR 계산 (FV/IRR Java 클래스 직접 구현 — `fv_irr` 패키지)
- 매수/매도 내역(`ivst_dtl_tb`)의 현금흐름 기반 계산
- 벤치마크 IRR과 고객 자산 IRR 비교

### 6-3. 벤치마크 대비 성과 분석
- 국내외 지수 대비 포트폴리오 성과 비교
- 자산유형·국내해외·매트릭스유형별 세분화된 비교

### 6-4. 투자전략 (Strategy) 관리
- 유지 / 비율조정 / 비율기반금액조정 / 기타 4가지 전략 유형
- 자산별 전략 태그 → 리밸런싱 제안 및 결과 추적
- 전략명 예시: 유지, 변액ERR, 펀드ERR, Golden Cross 등

### 6-5. CCR / FT (자산증가 분석)
- CCR (Cash Compensation Rate 추정): 자산 유지 여부 기반 현금흐름 분석
- Financial Trend: 시계열 재무지표 추세 분석

### 6-6. 지출 카테고리화
- 17개 지출유형으로 분류 (저축/대출상환/보험/주거비/교육/의료 등)
- 월별·연간 집계 및 현금흐름 분석
- 카드지급 연동 추적

---

## 7. 개발 이력별 주요 변경사항

### 1차 개발 (2015)
- 프로젝트 시작 (`incotore` 프로젝트명)
- 핵심 자산관리 DB 설계 및 기본 CRUD 구현
- 계약서 기준 기능명세서 작성 및 ERD 설계

### 2차 개발 (2016)
- 프로젝트명 `incoto`로 변경
- 분석/출력 기능 대거 추가 (FCR, TAA, 투자매트릭스, 투자결과, 현금흐름 등)
- 전체 Java 소스 약 200여 개 파일로 확장
- Bootstrap 기반 UI 완성

### 3차 추가개발 및 유지보수 (2017)
- **홈페이지 추가**: 재무전문가 소개, 투관 소개, 교육/멘토링/서비스 안내
- **고객 계좌/메모 관리**: `cus_account_tb`, `cus_memo_tb` 신규 테이블 추가
- **PMT 계산**: 목표금액 ↔ 월납입금 자동 계산 기능
- **상품별 투자 Tracking**: 현금/채권/주식/부동산/기타 개별 추적
- **dump_tb**: 시점별 데이터 스냅샷 저장 기능
- **관리자 메뉴 재구성**: 고객/회원/자산별 관리 화면 개선
- **fmy_mem_tb / insur_tb / mem_tb 구조 변경**: 컬럼 추가

---

## 8. 프로젝트 파일 구조

```
투자관제탑/
├── 2015 재무 투자성과 평가(투자관제탑)/
│   ├── incoto/incotore/         ← 1차 Eclipse 프로젝트 (Spring MVC, Maven)
│   │   └── src/main/java/net/su/...
│   └── 투자관제탑 자료/
│       ├── 00계약서/            ← 개발 계약서, 가격제안서, 기능명세서
│       ├── 01회의록/            ← 주간 회의록 (2015.03 ~ 07)
│       ├── 02기능명세서/        ← (완)INCOTO_기능명세서_20151108_종합.xlsx
│       ├── 03스토리보드/        ← UI 스토리보드 PPT
│       ├── 04ERD/               ← ERD (erwin), Data Dictionary
│       ├── 05DB/                ← SQL 덤프 파일 (incoto_cafe, incoto_test)
│       ├── 06컴포넌트정의서/    ← 컴포넌트 정의서
│       ├── 07원본자료/          ← 고객 사례, 벤치마크 데이터, 로고
│       ├── 08기능 및 공식/      ← FV/IRR 구현, 화면별 계산 공식 PPT
│       ├── 09사용자 메뉴얼/     ← 사용자 매뉴얼 PPT
│       └── 10WBS/               ← 프로젝트 일정 관리 (MS Project)
│
├── 2016 투자관제탑 2차/
│   ├── incoto/incoto/           ← 2차 Eclipse 프로젝트 (최종 완성본)
│   │   └── src/main/java/net/su/ ... (약 200+ Java 파일)
│   ├── Dump20160103/            ← 테이블별 개별 SQL 덤프
│   ├── incoto20160103.sql       ← 전체 DB 덤프
│   └── (2차)Entity Relationship Diagram.erwin
│
└── 2017 투자관제탑 3차/
    ├── INCOTO3DB(table)/        ← 3차 테이블 DDL SQL (테이블별 개별 파일)
    ├── INCOTO3DB(view)/         ← View 정의 SQL
    ├── INCOTO3DB(dump)/         ← 전체 DB 덤프
    ├── 개발자료/                ← 3차 화면 구성안, 투자Tracking 양식
    ├── 투자관제탑 유지보수/
    │   ├── incoto3차 개발/      ← 3차 개발 소스, 추가개발 질문 TXT
    │   └── 카페24/              ← 배포 서버 관련 파일
    └── (3차)incoto_DD_2017.xlsx ← 3차 Data Dictionary
```

---

## 9. 약어 사전 (컬럼명 규칙)

| 약어 | 의미 |
|------|------|
| cus | Customer (고객) |
| mem | Member (회원/가족구성원) |
| fmy | Family (가족) |
| splist | Specialist List (설계자) |
| asset | 자산 |
| ivst | Investment (투자) |
| liqdty | Liquidity (유동성) |
| bond | 채권 |
| cash | 현금/예금 |
| stock | 주식 |
| insur | Insurance (보험) |
| loan | 대출 |
| realest | Real Estate (부동산) |
| crdt_cad | Credit Card (신용카드) |
| bm | Benchmark (벤치마크) |
| strtg | Strategy (전략) |
| mtrx | Matrix (매트릭스) |
| ratio | 비율 |
| spd | Spend (지출) |
| anu | Annual (연간) |
| reglr_cst | Regular Cost (정기비용) |
| goal | 목표 |
| bnkbok | Bankbook (통장) |
| seq | Sequence (순번, PK) |
| nme | Name (이름) |
| mony | Money (금액) |
| ra | Rate (비율/금리) |
| dtl | Detail (상세) |
| tb | Table |
| div | Division (구분) |
| dome | Domestic (국내) |
| fore | Foreign (해외) |
| intrst | Interest (이자/금리) |
| ctrt | Contract (계약) |
| adjst | Adjust (조정) |
| aprs | Appraisal (평가) |
| pnsin | Pension (연금) |
| balnc | Balance (잔액) |
| io | In/Out (입출금) |
| cntact | Contact (연락처) |
| finac_instn | Financial Institution (금융기관) |
| fcr | Future Cash Requirement |
| irr | Internal Rate of Return |
| taa | Tactical Asset Allocation |
| ccr | Cash Compensation Rate |
| ft | Financial Trend |
| biz | Business (사업비) |
| resale | 환매 |
| tmpry | Temporary (임시납) |
