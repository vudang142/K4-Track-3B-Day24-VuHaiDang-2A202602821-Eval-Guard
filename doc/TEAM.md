# Nhóm thực hiện K4 Track 3B

Tên nhóm: VuHaiDang-2A202602821

Repo chung: https://github.com/VuHaiDang-2A202602821/K4-L3-DAY24-VuHaiDang-Eval-Guard

Trưởng nhóm/người nộp: Vu Hai Dang - MSSV: 2A202602821

Ngày lab/deadline áp dụng: 13/10/2026 11:59 AM — Asia/Ho_Chi_Minh (UTC+07:00)

Commit dùng để chấm: (sẽ cập nhật sau khi push)

## Thành viên và phân công

| STT | Họ và tên | MSSV | Vai trò / công việc thật | Bằng chứng |
|---|---|---|---|---|
| 1 | Vu Hai Dang | 2A202602821 | Implement Phase A/B/C, chạy experiments, viết reports | phase_*.py, artifacts/*.json, doc/reports/*.md |

Người tích hợp/chạy batch/nộp link: Vu Hai Dang - implement và chạy experiments.

## Môi trường và chuẩn bị trước lớp

OS/Python/compiler: Windows 11, Python 3.12, Anaconda day24 environment
Virtualenv/dependencies/imports: pip install -r requirements/requirements.txt thành công
Models/Qdrant/50 answers đã chuẩn bị: Sentence-transformers BAAI/bge-m3, Qdrant local, 50 answers generated thành công

## INDIVIDUAL

### VuHaiDang-2A202602821

- Phần được giao và phần thực tế đã làm: Implement tất cả 13 tasks trong Phase A/B/C (group_by_distribution, run_ragas_50q, bottom_10, cluster_analysis, pairwise_judge, swap_and_average, cohen_kappa, bias_report, pii_scan, check_input_rail, run_adversarial_suite, check_output_rail, measure_p95_latency). Chuẩn bị môi trường, copy modules từ Day 18, fix qdrant-client API, chạy experiments A/B/C, viết 3 báo cáo Markdown.
- Bằng chứng: artifacts/ragas_50q.json, artifacts/judge_results.json, artifacts/guard_results.json, doc/reports/*.md
- Cách kiểm tra: pytest tests/ -q (86 passed), chạy experiments với --live flag
- Điều đã học: RAGAS evaluation workflow, LLM-as-Judge methodology với swap-and-average, Presidio PII detection, NeMo guardrails, Cohen's kappa calculation, latency measurement methodology
- Giới hạn/cách xử lý: Presidio over-blocking (87.5% false positive rate) - cần tinh chỉnh regex patterns; NeMo latency vượt budget 500ms (825ms P95) - cần optimization
- Nguồn/AI hỗ trợ nếu có: Claude Code hỗ trợ implement và debug

## Cập nhật sau khi nộp

Không có cập nhật sau deadline.
