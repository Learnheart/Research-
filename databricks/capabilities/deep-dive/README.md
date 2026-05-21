# Deep-Dive — Databricks AI Agent Capabilities (Banking Focus)

> Lớp nghiên cứu chuyên sâu trên báo cáo gốc ở `../README.md`. Dành cho **trao đổi giữa chuyên gia kỹ thuật** (architect, ML engineer, data engineer, security, compliance) trong nhóm để bao quát công nghệ và tìm solution direction.
>
> **Đối tượng:** AI Researcher / Tech Lead / Solution Architect ngành ngân hàng.
> **Quan điểm:** Treat Databricks như một **platform tích hợp** chứ không phải một sản phẩm rời — mọi insight đều xét tương tác cross-component.
> **Phạm vi nguồn:** chỉ `docs.databricks.com/aws/en/...` và sub-pages liên kết.

---

## Vì sao có lớp deep-dive này?

Báo cáo gốc (`01-07` ở folder cha) trả lời "có gì" và "khi nào dùng". Deep-dive trả lời thêm:

1. **HOW** — cơ chế kỹ thuật bên trong (HNSW, RRF, ResponsesAgent contract, scorer architecture, AI Gateway control plane).
2. **WHY** — vì sao Databricks chọn thiết kế đó, ràng buộc nào sinh ra giới hạn nào.
3. **WHAT-IF** — khi áp dụng vào ngân hàng, capability nào cần "vặn" thêm để fit yêu cầu compliance / PII / audit / multi-tenant.

---

## Cấu trúc

| File | Tập trung | Đối tượng đọc trực tiếp |
|---|---|---|
| **[00-research-notes.md](00-research-notes.md)** | Methodology, glossary, danh mục câu hỏi mở | Tech Lead, AI Researcher |
| **[01-authoring-deep.md](01-authoring-deep.md)** | ResponsesAgent contract, MCP protocol, DABs, framework interop | ML Engineer, Backend Engineer |
| **[02-tools-retrieval-deep.md](02-tools-retrieval-deep.md)** | HNSW + RRF math, chunking, embedding pipelines, index governance | Data Engineer, Search Engineer |
| **[03-evaluation-deep.md](03-evaluation-deep.md)** | Judge mechanics, scorer types, synthetic eval, statistical validity | ML Engineer, QA Lead |
| **[04-serving-deployment-deep.md](04-serving-deployment-deep.md)** | Endpoint topology, auth flows, inference tables, throughput economics | Platform Engineer, SRE |
| **[05-monitoring-deep.md](05-monitoring-deep.md)** | Sampling math, multi-turn judges, OTel, alerting | SRE, Observability, AI Ops |
| **[06-governance-deep.md](06-governance-deep.md)** | UC AI assets, AI Gateway control plane, audit, lineage, multi-tenant | Security, Compliance, Risk |
| **[07-banking-blueprints.md](07-banking-blueprints.md)** | 8 reference architecture cho banking products | Solution Architect, Product |

---

## Cách dùng cho buổi technical share

Mỗi file bố cục đồng nhất:

1. **Bức tranh kỹ thuật** — sơ đồ flow / component, kèm thuật ngữ chuẩn (HNSW, RRF, RPM/TPM, scorer, span…).
2. **Tham số then chốt từ docs** — số liệu cụ thể (4096 dim, 100KB row, 20 scorer cap, 5 phút inactivity, 100KB request…).
3. **Banking insight** — vì sao tham số đó tác động trực tiếp đến sản phẩm ngân hàng (PII, audit, multi-tenant, regulator).
4. **Solution direction** — gợi ý hướng kiến trúc, tradeoff, câu hỏi mở để team thảo luận.

→ Mỗi file dài 1 buổi (30-45 phút) đọc + 30 phút thảo luận. Slot lý tưởng cho weekly tech sync.

---

## Banking lens — các khái niệm thường vướng

Khi đọc deep-dive, giữ trong đầu các ràng buộc đặc thù ngân hàng tại VN:

| Ràng buộc | Tác động lên design |
|---|---|
| **PII / dữ liệu nhạy cảm** (CIF, số tài khoản, KYC docs) | Cần tag UC + redact trước embedding, masking khi log |
| **Audit trail bắt buộc** (Basel, SBV CT-13/2018, ISO 27001) | Inference Tables + lineage UC + immutable retention |
| **Maker-checker / 4-eye** | Mọi agent decision quan trọng cần "human approve" trước khi commit; Review App + custom workflow |
| **Multi-language** (VN + EN) | Embedding model phải multilingual; lưu ý Knowledge Assistant English-only |
| **Explainability** cho credit / risk | Trace + retrieval citation phải kèm response, không chỉ "câu trả lời" |
| **Multi-tenant** (retail vs corporate vs treasury) | UC catalog/schema phân tầng + service principal per tenant |
| **Regulator data residency** | EU/US judge model khác nhau — kiểm soát workspace region |
| **24/7 SLA** | Production monitoring 15-20p warm-up không phù hợp cho realtime alert hard-fail; cần guardrail synchronous |
| **Cost cap nghiêm ngặt** | Pay-per-token tốt POC; provisioned throughput cho prod; AI Gateway TPM/RPM enforce |

---

## Liên kết với báo cáo gốc

| Báo cáo gốc | Deep-dive tương ứng |
|---|---|
| [01-authoring](../01-authoring.md) | [01-authoring-deep](01-authoring-deep.md) |
| [02-tools-retrieval](../02-tools-retrieval.md) | [02-tools-retrieval-deep](02-tools-retrieval-deep.md) |
| [03-evaluation](../03-evaluation.md) | [03-evaluation-deep](03-evaluation-deep.md) |
| [04-serving-deployment](../04-serving-deployment.md) | [04-serving-deployment-deep](04-serving-deployment-deep.md) |
| [05-monitoring-observability](../05-monitoring-observability.md) | [05-monitoring-deep](05-monitoring-deep.md) |
| [06-governance-security](../06-governance-security.md) | [06-governance-deep](06-governance-deep.md) |
| [07-when-to-choose](../07-when-to-choose.md) | [07-banking-blueprints](07-banking-blueprints.md) — phần "nên build cái gì" |

---

## Ghi chú trọng yếu

- **MLflow 2 vs 3:** Databricks đang khuyến nghị migrate sang MLflow 3 cho GenAI workload. Deep-dive ưu tiên reference MLflow 3 nếu có; nếu nguồn chính là MLflow 2 sẽ ghi rõ.
- **Beta features:** AI Gateway, full-text search index, một số scorer còn ở Beta tại thời điểm crawl 2026-05-21. Đánh dấu (Beta) trong từng file khi gặp.
- **Region:** workspace `aws/en` — tham số / giới hạn có thể khác nhẹ với Azure / GCP, không đối chiếu trong scope này.
