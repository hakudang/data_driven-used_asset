# REQUIREMENT TRACEABILITY MATRIX (RTM)
## Hệ thống Kiểm soát Đầu tư Tài sản Cũ
```
version : v1.0
status : Full Trace Coverage (CR ↔ SR ↔ FR ↔ NFR ↔ TEST ↔ ACCEPTANCE)
owner : Dang
```
---

## 1. DOCUMENT CONTROL

| Item | Value |
|------|-------|
| Owner | Dang |
| Region | Japan |
| Version | RTM v1.0 |
| Status | QA Baseline |
| Related Docs | CR v1.0, SR v1.2 ENTERPRISE |
| Purpose | Đảm bảo 100% requirement được truy vết và kiểm thử |

---

# 2. TRACEABILITY: CR → SR (BUSINESS COVERAGE)

| CR ID | Business Objective | Covered By SR | Related FR | Related NFR | Coverage |
|-------|--------------------|--------------|------------|------------|----------|
| CR-01 | Chuẩn hóa quyết định | Rule Engine | FR-03~FR-12 | NFR-07 | ✔ |
| CR-02 | Bảo vệ vốn | Scrap Floor + Risk | FR-06, FR-08, FR-10 | NFR-15 | ✔ |
| CR-03 | Giảm mua cảm tính | Decision + Gating | FR-07, FR-08, FR-22 | NFR-18 | ✔ |
| CR-04 | Tăng chất lượng thương lượng | Counter Offer | FR-19 | - | ✔ |
| CR-05 | Kiểm soát rủi ro | Risk Override + Completeness | FR-10, FR-17, FR-22 | NFR-18 | ✔ |
| CR-06 | AI chỉ tư vấn | Confidence + Contract | FR-20, FR-22 | NFR-02 | ✔ |
| CR-07 | Quyết định truy vết | Audit + Logging | FR-24~FR-29 | NFR-12 | ✔ |

✅ 100% Business Objectives được cover bởi SR.

---

# 3. TRACEABILITY: FR → TEST → ACCEPTANCE

## 3.1 RULE ENGINE

| FR ID | Requirement | Test ID | Acceptance Ref | Critical |
|-------|------------|----------|----------------|----------|
| FR-01 | Create Asset | TC-01 | FR-01 Acceptance | Medium |
| FR-02 | Update Asset | TC-02 | FR-02 Acceptance | Medium |
| FR-03 | Calculate Total Cost | TC-03 | FR-03 Acceptance | High |
| FR-04 | Calculate Expected Profit | TC-04 | FR-04 Acceptance | High |
| FR-05 | Calculate ROI | TC-05 | FR-05 Acceptance | High |
| FR-06 | Calculate Scrap Floor | TC-06 | FR-06 Acceptance | Critical |
| FR-07 | Decision Engine Resale | TC-07 | FR-07 Acceptance | Critical |
| FR-08 | Decision Engine Scrap | TC-08 | FR-08 Acceptance | Critical |
| FR-09 | Decision Engine Art | TC-09 | FR-09 Acceptance | High |
| FR-10 | Risk Override | TC-10 | FR-10 Acceptance | Critical |
| FR-11 | Mode Switching | TC-11 | FR-11 Acceptance | Medium |
| FR-12 | Configurable Threshold | TC-12 | FR-12 Acceptance | High |

---

## 3.2 AI REVIEWER

| FR ID | Requirement | Test ID | Acceptance Ref | Critical |
|-------|------------|----------|----------------|----------|
| FR-13 | Trigger Evaluate | TC-13 | FR-13 Acceptance | Medium |
| FR-14 | Payload Builder | TC-14 | FR-14 Acceptance | Critical |
| FR-15 | OCR Nameplate | TC-15 | FR-15 Acceptance | Medium |
| FR-16 | Visual Analysis | TC-16 | FR-16 Acceptance | Medium |
| FR-17 | Data Completeness | TC-17 | FR-17 Acceptance | Critical |
| FR-18 | Market Advisory | TC-18 | FR-18 Acceptance | Medium |
| FR-19 | Counter Offer | TC-19 | FR-19 Acceptance | High |
| FR-20 | AI Output Contract | TC-20 | FR-20 Acceptance | Critical |
| FR-21 | Confidence Score | TC-21 | FR-21 Acceptance | High |
| FR-22 | Confidence Gating | TC-22 | FR-22 Acceptance | Critical |
| FR-23 | Sidebar Rendering | TC-23 | FR-23 Acceptance | Medium |
| FR-24 | AI Logging | TC-24 | FR-24 Acceptance | Critical |

---

## 3.3 GOVERNANCE & AUDIT

| FR ID | Requirement | Test ID | Acceptance Ref | Critical |
|-------|------------|----------|----------------|----------|
| FR-25 | Decision Final Record | TC-25 | FR-25 Acceptance | Critical |
| FR-26 | Audit Trail | TC-26 | FR-26 Acceptance | Critical |
| FR-27 | Rule Version Tracking | TC-27 | FR-27 Acceptance | High |
| FR-28 | Manual Override Control | TC-28 | FR-28 Acceptance | Critical |
| FR-29 | AI Summary Storage | TC-29 | FR-29 Acceptance | Medium |
| FR-30 | Batch Ready | TC-30 | FR-30 Acceptance | Low |

---

# 4. TRACEABILITY: NFR → TEST → ACCEPTANCE

| NFR ID | Requirement | Test ID | Acceptance Ref | Critical Level |
|--------|------------|----------|----------------|----------------|
| NFR-01 | Performance | TC-N01 | NFR-01 Acceptance | High |
| NFR-02 | Availability | TC-N02 | NFR-02 Acceptance | Critical |
| NFR-03 | Security | TC-N03 | NFR-03 Acceptance | Critical |
| NFR-04 | No Crawling | TC-N04 | NFR-04 Acceptance | Medium |
| NFR-05 | Logging Reliability | TC-N05 | NFR-05 Acceptance | Critical |
| NFR-06 | Scalability | TC-N06 | NFR-06 Acceptance | Medium |
| NFR-07 | Config Management | TC-N07 | NFR-07 Acceptance | High |
| NFR-08 | Data Integrity | TC-N08 | NFR-08 Acceptance | Critical |
| NFR-09 | Error Handling | TC-N09 | NFR-09 Acceptance | High |
| NFR-10 | Usability | TC-N10 | NFR-10 Acceptance | Medium |
| NFR-11 | Maintainability | TC-N11 | NFR-11 Acceptance | Medium |
| NFR-12 | Audit Compliance | TC-N12 | NFR-12 Acceptance | Critical |
| NFR-13 | Extensibility | TC-N13 | NFR-13 Acceptance | Medium |
| NFR-14 | Confidentiality | TC-N14 | NFR-14 Acceptance | Critical |
| NFR-15 | Threshold Safety | TC-N15 | NFR-15 Acceptance | Critical |
| NFR-16 | Recovery | TC-N16 | NFR-16 Acceptance | High |
| NFR-17 | Provider Swap Ready | TC-N17 | NFR-17 Acceptance | Medium |
| NFR-18 | Data Completeness Enforcement | TC-N18 | NFR-18 Acceptance | Critical |
| NFR-19 | Versioning Discipline | TC-N19 | NFR-19 Acceptance | Medium |
| NFR-20 | SaaS Ready | TC-N20 | NFR-20 Acceptance | Medium |

---

# 5. END-TO-END TRACE EXAMPLE

## Scenario: ROI cao nhưng Risk cao

- CR-02: Bảo vệ vốn  
- FR-10: Risk Override  
- NFR-15: Threshold Safety  
- TC-10: risk=85, roi=0.5  
- Expected Result: decision_suggested = SKIP  

Trace đầy đủ từ Business → Rule → Test → Safety.

---

# 6. COVERAGE SUMMARY

| Layer | Total | Covered | % |
|-------|--------|----------|---|
| CR | 7 | 7 | 100% |
| FR | 30 | 30 | 100% |
| NFR | 20 | 20 | 100% |

Không có requirement bị bỏ sót.

---

# 7. GO-LIVE GATE CRITERIA

Hệ thống chỉ được phép Go-Live khi:

- 100% FR Pass
- 100% NFR Critical Pass
- Không có Severity 1 bug
- Risk Override không có false negative
- Confidence Gating hoạt động chính xác
- Audit Trail truy vết đầy đủ decision_final

---

# 8. VERSION CONTROL POLICY

- Mỗi thay đổi SR phải cập nhật RTM
- rule_version phải sync với SR version
- Mỗi release phải có changelog
- RTM là tài liệu bắt buộc trước UAT

---

# END OF RTM DOCUMENT
