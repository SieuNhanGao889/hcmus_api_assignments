# Static review — PHASE_7_AUTOMATION_CORRECTION

## 1. Phạm vi

Human đã review `reports/analysis/newman_execution_analysis.md` và phê duyệt:

- PB-01 đến PB-08: `CONFIRMED_BUG` cho mục đích bug reporting;
- AF-01 và AF-02: sửa Postman automation trước khi rerun Order Status.

Phase này chỉ sửa automation/data cần thiết cho AF-01 và AF-02:

- `postman/HW06_EShop_API_Tests.postman_collection.json`
- `postman/data/order_status_data.json`

Không sửa SUT, không sửa confirmed bug behavior, không chạy Postman/Newman, không gửi API request và không tạo execution evidence mới.

## 2. Human decision record

| Finding | Human decision |
|---|---|
| PB-01 Login response exposes plaintext password/sensitive fields | `CONFIRMED_BUG` for bug reporting |
| PB-02 Stack trace/path/framework leakage | `CONFIRMED_BUG` for bug reporting |
| PB-03 Lockout after two failures | `CONFIRMED_BUG` for bug reporting |
| PB-04 Coupon equality boundary rejected | `CONFIRMED_BUG` for bug reporting |
| PB-05 Object-valued `user_id` grants discount | `CONFIRMED_BUG` for bug reporting |
| PB-06 Incorrect negative percent discount | `CONFIRMED_BUG` for bug reporting |
| PB-07 Normal-user JWT updates admin order status | `CONFIRMED_BUG` for bug reporting |
| PB-08 `canceled -> delivered` accepted | `CONFIRMED_BUG` for bug reporting |
| AF-01 invalid/unknown order-ID assertion mode | `APPROVED_AUTOMATION_CORRECTION` |
| AF-02 wrong-Content-Type hard state assertion | `APPROVED_AUTOMATION_CORRECTION` |

### PB-05 audit-history separation

Original AI analysis remains unchanged: PB-05 was classified `POTENTIAL_SUT_BUG` with **Medium** confidence because observed acceptance was clear while the specification did not provide a complete type/rejection table.

Human decision is recorded separately here: after reviewing the real behavior (`user_id` object accepted with `success=true` and a valid discount), human confirms PB-05 as `CONFIRMED_BUG` for bug reporting. This does not rewrite the AI's historical classification or confidence.

## 3. AF-01 correction

Affected cases:

- `ORDERSTATUS-GEN-012`
- `ORDERSTATUS-GEN-013`
- `ORDERSTATUS-GEN-014`
- `ORDERSTATUS-GEN-015`
- `ORDERSTATUS-GEN-016`
- `ORDERSTATUS-GEN-017`
- `ORDERSTATUS-GEN-018`

Correction applied:

- `assertion_mode` changed from `STATE_CHANGED_TO_TARGET` to `STATE_UNCHANGED_SAFE_REJECTION`.
- CASE assertion checks a safe rejection by requiring a non-2xx response, without inventing an exact status for historical `SPEC_UNDEFINED` cases.
- Read-after verification asserts the controlled path order remains at `source_state` (`pending`).
- Existing no-internal-leakage assertion remains active.

This aligns the automation with each original objective: unknown/invalid path IDs must not cause an unrelated controlled order to change.

## 4. AF-02 correction

Affected case: `ORDERSTATUS-GEN-035`.

Correction applied:

- `assertion_mode` changed from `STATE_CHANGED_TO_TARGET` to `OBSERVE_STATE_NO_INTERNAL_LEAKAGE`.
- Verification still confirms the controlled order exists, then records `before`, `after` and response status as an exploratory observation.
- No success status or required state transition is asserted.
- The CASE-level assertion `[ORDERSTATUS-GEN-035] no internal details leakage` remains unchanged and continues to support PB-02.

This preserves the historical `SPEC_UNDEFINED`/exploratory wrong-Content-Type behavior without weakening the independent security assertion.

## 5. Static validation

| Check | Result | Evidence |
|---|---|---|
| JSON parsing | **PASS** | `5/5` Postman JSON artifacts parse successfully |
| JavaScript syntax | **PASS** | `86/86` scripts compile with `new Function`; `0` syntax errors |
| AF-01 modes | **PASS** | `7/7` affected rows use `STATE_UNCHANGED_SAFE_REJECTION` |
| AF-02 mode | **PASS** | `1/1` affected row uses `OBSERVE_STATE_NO_INTERNAL_LEAKAGE` |
| AF-01 response assertion | **PASS** | Mode has non-exact non-2xx safe-rejection assertion |
| AF-01 persisted state | **PASS** | Mode is included in unchanged-state verification |
| AF-02 exploratory behavior | **PASS** | Mode only logs persisted observation; no hard state-change assertion |
| Original history fields | **PASS** | All 8 affected rows retain every checked `original_*` field |
| Traceability | **PASS** | All 8 affected `case_id` and `scenario_id` values remain unchanged and unique |
| PB assertions | **PASS** | Static presence/mode checks pass for PB-01 through PB-08 |
| Execution reports | **PASS** | Original Newman report hashes remain unchanged |
| Original AI analysis | **PASS** | `newman_execution_analysis.md` hash remains unchanged |

## 6. PB-01 through PB-08 assertion preservation

| Finding | Preserved automation/evidence assertion |
|---|---|
| PB-01 | Login `no sensitive user fields` assertion remains present |
| PB-02 | Login `no internal details leak` and Order Status `no internal details leakage` assertions remain present; AF-02 explicitly retains the latter |
| PB-03 | Two-attempt success boundary and three-attempt locked-behavior assertions remain present |
| PB-04 | `COUPON-GEN-006` and `COUPON-EXT-002-EQUAL` retain `SUCCESS_SCHEMA` boundary assertions |
| PB-05 | `COUPON-GEN-025` retains `REJECT_NO_DISCOUNT` / `no valid discount is returned` |
| PB-06 | `COUPON-EXT-001` retains `PERCENT_FORMULA` assertion |
| PB-07 | `ORDERSTATUS-GEN-010/011/031` and `ORDERSTATUS-EXT-001` retain `STATE_UNCHANGED_AUTHORIZATION` |
| PB-08 | `ORDERSTATUS-EXT-003-CANCELED-TO-DELIVERED` retains `STATE_UNCHANGED_INVALID_TRANSITION` |

No assertion supporting PB-01 through PB-08 was removed, weakened or changed to make a rerun pass.

## 7. Historical artifact integrity

SHA-256 values after correction:

- `reports/analysis/newman_execution_analysis.md`: `673DDE4743D9DB703DE6C9DD775811EDED745400FF78EE9EAFD577CDFE199410`
- `reports/newman/login_run.json`: `85A661E642CB7D92C48A41C6534EC5EC44225ABE5170D5076188ECCC6211D1C2`
- `reports/newman/coupon_run.json`: `DCFBB6098E7C27FABDFF7DDA77B26043A2318EFFAA58EFC727CD128203C48842`
- `reports/newman/order_status_run.json`: `B1C5C5A8914B66097F228C25FEBE5C025A7FE7634D0805EC69154FE85A6C8B8F`

These artifacts were read-only during this correction. The original AI classifications, confidence statements, observed evidence and original Newman executions remain historical records; human confirmations are recorded separately in this review.

## 8. Conclusion

Static review result: **PASS — AF-01_AND_AF-02_CORRECTED**.

The corrected Order Status automation is ready for final human approval before rerun. No rerun or new execution evidence has been created. Stop at this checkpoint.
