# REQUIREMENT TRACEABILITY MATRIX (RTM)
## Hệ thống Kiểm soát Đầu tư Tài sản Cũ
```
Version : RTM v1.1
Owner : Dang
Status : Governance Controlled – Dev & QA Baseline
Region : Japan
Date : 2026-02-19
Related :
- CR v1.0
- SR v1.1
- BR v1.1
- UC v1.1
```

---

# 1. PURPOSE

RTM đảm bảo:

- Mọi yêu cầu từ CR đều được đặc tả trong SR
- Mọi logic trong SR đều có BR định nghĩa
- Mọi requirement đều có Use Case thực thi
- Không có requirement "mồ côi"
- Có thể trace ngược từ UC → SR → CR

---

# 2. CR ↔ SR ↔ BR ↔ UC TRACE TABLE

---

## 2.1 Core Financial & Decision Logic

| CR Ref | Requirement | SR Ref | BR Ref | UC Ref |
|--------|------------|--------|--------|--------|
| CR-RuleEngine | Tính tổng chi phí | FR-03 | FRL-01 | UC-03 |
| CR-RuleEngine | Tính ROI | FR-05 | FRL-03 | UC-03 |
| CR-RuleEngine | Tính Scrap Floor | FR-06 | FRL-04 | UC-03 |
| CR-RuleEngine | BUY/NEGOTIATE/SKIP (RESALE) | FR-07 | DR-01 | UC-04 |
| CR-RuleEngine | BUY/NEGOTIATE/SKIP (SCRAP) | FR-08 | DR-03 | UC-04 |
| CR-RuleEngine | BUY/NEGOTIATE/SKIP (ART) | FR-09 | DR-02 | UC-04 |
| CR-Risk | Risk ≥ 80 → Không BUY | FR-10 | RR-01 | UC-05 |
| CR-Mode | Mode switching | FR-11 | DR-04 | UC-03 / UC-04 |

---

## 2.2 Data Governance & Completeness

| CR Ref | Requirement | SR Ref | BR Ref | UC Ref |
|--------|------------|--------|--------|--------|
| CR-Data | Thiếu dữ liệu nghiêm trọng → Không BUY | NFR-18 | DGR-01 / DGR-02 | UC-08 |
| CR-Data | AI không thay thế quyết định | FR-21 | AR-01 | UC-07 / UC-09 |
| CR-Confidence | Confidence Score | FR-21 / FR-22 | AR-02 | UC-08 |
| CR-AI | AI Output Contract | FR-20 | AR-05 | UC-07 |

---

## 2.3 Large Deal Governance (Freeze v1.1)

| CR Ref | Requirement | SR Ref | BR Ref | UC Ref |
|--------|------------|--------|--------|--------|
| CR-Governance | Deal lớn cần kiểm soát | FR-31 | CRL-03 | UC-16 |
| CR-Governance | Không finalize nếu chưa AI | NFR-21 | CRL-03 | UC-09 / UC-16 |

---

## 2.4 ART Provenance Governance

| CR Ref | Requirement | SR Ref | BR Ref | UC Ref |
|--------|------------|--------|--------|--------|
| CR-Rule | ART ROI ≥ 40% | FR-09 | DR-02 | UC-04 |
| CR-Governance | ART ≥ 300k cần provenance | FR-32 | DGR-01 | UC-17 |
| CR-Governance | Provenance thiếu → block BUY | NFR-22 | DGR-02 | UC-17 / UC-09 |

---

## 2.5 Image Handling & AI Vision

| CR Ref | Requirement | SR Ref | BR Ref | UC Ref |
|--------|------------|--------|--------|--------|
| CR-AI | Phân tích hình ảnh | FR-15 / FR-16 | AR-01 | UC-07 |
| CR-AI | Upload ảnh | FR-36 | DGR | UC-13 |
| CR-AI | Validate ảnh | FR-37 | DGR | UC-14 |
| CR-AI | Image gate trước AI | FR-38 | DGR-02 | UC-06 |
| CR-AI | Folder convention | FR-39 | CFG | UC-14 |
| CR-AI | Permission control | FR-40 | CRL | UC-15 |
| CR-AI | Retention 1 năm | FR-35 | CRL | UC-15 |
| CR-NFR | Image retention compliance | NFR-24 | CRL | UC-15 |

---

## 2.6 Audit & Governance

| CR Ref | Requirement | SR Ref | BR Ref | UC Ref |
|--------|------------|--------|--------|--------|
| CR-Governance | Quyết định minh bạch | FR-25 | CRL-02 | UC-09 |
| CR-Governance | Audit trail | FR-26 | CRL-03 | UC-11 |
| CR-Governance | Audit critical only | FR-33 | CRL-03 | UC-11 |
| CR-Governance | Manual override cần lý do | FR-28 | CRL-01 | UC-10 |
| CR-Governance | Rule version tracking | FR-27 | CRL-02 | UC-09 |
| CR-NFR | Logging reliability | NFR-05 | CRL-04 | UC-11 |

---

## 2.7 Configuration & Threshold Control

| CR Ref | Requirement | SR Ref | BR Ref | UC Ref |
|--------|------------|--------|--------|--------|
| CR-Config | Threshold configurable | FR-12 | CFG-01 | UC-12 |
| CR-Config | Real-time effect | NFR-07 | CFG-02 | UC-12 |
| CR-Config | Retry policy | FR-20 | CFG-03 | UC-06 |

---

# 3. UC COVERAGE CHECK

| UC ID | Covered by SR | Covered by BR | Traceable to CR |
|--------|--------------|--------------|----------------|
| UC-01 | FR-01 | — | ✔ |
| UC-02 | FR-02 | — | ✔ |
| UC-03 | FR-03~06 | FRL-01~04 | ✔ |
| UC-04 | FR-07~09 | DR-01~03 | ✔ |
| UC-05 | FR-10 | RR-01 | ✔ |
| UC-06 | FR-13~20 | AR-05 | ✔ |
| UC-07 | FR-15~21 | AR-01~05 | ✔ |
| UC-08 | NFR-18 | DGR-01/02 + AR-02 | ✔ |
| UC-09 | FR-25~32 | CRL-01~03 | ✔ |
| UC-10 | FR-28 | CRL-01 | ✔ |
| UC-11 | FR-26/33 | CRL-03/04 | ✔ |
| UC-12 | FR-12 | CFG-01/02 | ✔ |
| UC-13 | FR-36 | — | ✔ |
| UC-14 | FR-37/39 | CFG | ✔ |
| UC-15 | FR-35/40 | CRL | ✔ |
| UC-16 | FR-31 | CRL-03 | ✔ |
| UC-17 | FR-32 | DGR-01/02 | ✔ |

---

# 4. REQUIREMENT COVERAGE VALIDATION

✔ Không có FR bị orphan 
✔ Không có BR không được implement qua UC  
✔ Không có UC không trace được về SR  
✔ Mọi governance freeze (deal lớn, provenance, audit scope, image retention) đều có trace chain đầy đủ  

Trace chain mẫu:

CR → SR FR-31 → BR CRL-03 → UC-16 → UC-09  
CR → SR FR-32 → BR DGR-02 → UC-17 → UC-09  
CR → SR FR-38 → BR DGR-02 → UC-06  

---

# 5. GOVERNANCE GUARANTEE

RTM v1.1 đảm bảo:

- Tính nhất quán giữa tài liệu khách hàng và kỹ thuật
- Không lệch rule giữa SR và BR
- QA có thể viết test case từ UC và trace ngược về CR
- Freeze policy được kiểm soát version

---

# 6. CHANGE LOG

| Version | Date | Description | Author |
|----------|------|------------| ---- |
| v1.0 | 2026-02-19 | Initial, Core mapping | Dang |
| v1.1 | 2026-02-20 | Align with SR v1.1 freeze: Large Deal AI, ART Provenance, Image Handling, Audit Scope | Dang |

---

# END OF RTM