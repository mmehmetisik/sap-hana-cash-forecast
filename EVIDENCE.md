# Evidence — 01 Treasury

Every number in the README or in a post is listed here with the command that produced it.
A number whose command cannot be written down is not published. Commands run from the project
root with the virtual environment active and `PYTHONPATH=.;01-hazine`. (The engine code is not in
this repository; the commands are listed so the claims can be traced.)

| Date | Claim | Command | Result |
|---|---|---|---|
| 2026-11-21 | Latest HANA Express image on Docker Hub is SPS 08 | `Invoke-RestMethod 'https://hub.docker.com/v2/repositories/saplabs/hanaexpress/tags?page_size=5&ordering=last_updated'` | `2.00.088.00.20251110.1`, 1.76 GB compressed |
| 2026-11-21 | HANA is up; first start in the correct mode | `docker logs sap-hana-finans-hxe` · `python -m ortak.hana.kurulum` | `Start type: initial`, 326 s · `SELECT * FROM DUMMY -> X` · version 2.00.088.00.1760424921 · database HXE |
| 2026-11-21 | Real CBRT FX rates | `python -m ortak.veri.tcmb 2024-07-25 2026-11-20` | 579 business-day rates, 28 weekdays without publication · 2026-11-20 USD 50.4028 · EUR 55.2974 |
| 2026-11-21 | Business calendar matches the central bank exactly | `pytest ortak/tests/test_veri.py::test_takvim_tcmb_yayin_gunleriyle_birebir_tutar` | Passed. The first holiday list missed 30 Aug 2024 and 6 half-day eves; the comparison caught it. |
| 2026-11-21 | Data volume, trial balance | `python -m ortak.veri.uret` | BKPF 27,021 · BSEG/ACDOCA 67,931 · KNA1 80 · LFA1 40 · TCURR 2,316 · open receivable items 758 · open payable items 260 · SHA-256 `32f564f6…6e471` · lowest TRY bank balance 11,736,415.57 (31 Jul 2025) |
| 2026-11-21 | HANA load is correct | `python -m ortak.hana.yukle` | 3.9 s · row counts match · trial balance 0 · unbalanced documents 0 · BSEG–ACDOCA mismatches 0 |
| 2026-11-21 | The API cannot write to the database | `python -m ortak.hana.yukle` (last step) · `pytest -k okuyucu` | Read-only user `DELETE FROM T001` → `insufficient privilege` |
| 2026-11-21 | Forecast speed and riskiest week | `python -m hazine ozet` | Setup 0.87 s · band 0.005 s (N = 2,000) · opening cash TRY 80.8M · Nov 30 – Dec 6: P50 87.0 · Excel 148.4 · risk 30.0% |
| 2026-11-21 | Where the risk comes from | `python -m hazine risk` | 0.28 s · if Sedir Tarım or Gelincik Hidrolik paid on time, risk 30.0% → 0.0% |
| 2026-11-21 | Blind test | `python -m hazine kortest` | All 7 of the top-7 risk sources are late or slowing payers in the hidden profile; across all customers the share is 25/80 |
| 2026-11-21 | Scenario speed and effect | `POST /api/senaryo {"kunnr": Sedir, "ek_gecikme": 30}` · `{"kur_soku": 0.10}` | 10.6 ms · highest risk 30.0% → 77.0% · FX +10%: 5.4% (5.3 ms) |
| 2026-11-21 | Calibration | `python -m hazine kalibrasyon` | 26 start dates (27 Feb – 21 Aug 2026), 338 observations · coverage of the 80% band 76.9% (below 13.0% · above 10.1%) · by week 65 77 81 81 69 81 81 81 81 81 81 73 69 · MAE model 3.6 / Excel 40.2 TRY M · bias +1.2 / +40.2 TRY M · PIT [44, 28, 33, 34, 38, 42, 36, 32, 17, 34] |
| 2026-11-21 | Can we make this payment? | `POST /api/odeme {"tutar": 5000000, "tarih": "2026-11-23"}` | 51.2 ms · Not advisable · Nov 30 – Dec 6 risk 30.0% → 83.6% · lowest P10 cash TRY 77.6M · earliest safe dates 7–9 Dec 2026 (increase below 1%) |
| 2026-11-21 | Tests | `python -m pytest -q` | 33 passed (8.4 s), including 5 payment-check tests |
| 2026-11-21 | The tests really measure something | Each fix reverted temporarily, matching test run | Tail fix reverted → `test_karistir_…` RED · social-security due date reverted → `test_ayni_ay_iki_sgk_…` RED · fixes restored → 28 passed (the suite at that time, before the 5 payment-check tests) |
| 2026-11-21 | UI builds | `npx tsc --noEmit` · `npm run build` | 0 type errors · build 1.7 s · main bundle 1.87 MB (455 KB gzip) — UI5 + Recharts are large |
| 2026-11-21 | Screenshots are not edited by hand | `ekran_al.ps1` | 5 PNGs at 2880 px width in `images/`, taken from URLs of the running app (English UI) with a fresh browser profile on every run |

## Intermediate results (before fixes, for the record)

| Change | Before | After |
|---|---|---|
| The company's own payment drift added to the model | Coverage 76.9% · week 1 58% · actual cash always above P50 (+1.75M) | Coverage 79.0% · week 1 65% |
| Tail completion fixed | Coverage 79.0%, but an invoice 64 days overdue was predicted to be paid "7 months later" | Coverage 76.9% · dates are sensible |
