# Phân tích lỗi Phase A

Nhóm: VuHaiDang-2A202602821

Người phụ trách: Vu Hai Dang - 2A202602821

Nguồn: `artifacts/ragas_50q.json`; run timestamp: 2026-10-13T04:08:44+00:00.
Generation model: gpt-4o-mini-2024-07-18; Evaluator model: gpt-4o-mini.

## Điểm theo distribution

| Metric | factual (20) | multi_hop (20) | adversarial (10) |
|---|---|---|---|
| faithfulness | 0.8833 | 0.5646 | 0.6000 |
| answer_relevancy | 0.8031 | 0.7483 | 0.5325 |
| context_precision | 0.9833 | 0.9083 | 0.9833 |
| context_recall | 0.9750 | 0.8042 | 0.6833 |
| avg_score | 0.9112 | 0.7563 | 0.6998 |

## Bottom 10 và nguyên nhân

| Rank | ID | Distribution | Worst Metric | Avg Score | Diagnosis | Suggested Fix |
|---|---|---|---|---|---|---|
| 1 | 39 | multi_hop | faithfulness | 0.0000 | LLM hallucinating | Tighten system prompt, lower temperature |
| 2 | 44 | adversarial | faithfulness | 0.3750 | LLM hallucinating | Tighten system prompt, lower temperature |
| 3 | 33 | multi_hop | faithfulness | 0.3750 | LLM hallucinating | Tighten system prompt, lower temperature |
| 4 | 43 | adversarial | faithfulness | 0.4167 | LLM hallucinating | Tighten system prompt, lower temperature |
| 5 | 50 | adversarial | faithfulness | 0.4167 | LLM hallucinating | Tighten system prompt, lower temperature |
| 6 | 42 | adversarial | faithfulness | 0.4167 | LLM hallucinating | Tighten system prompt, lower temperature |
| 7 | 9 | factual | faithfulness | 0.5000 | LLM hallucinating | Tighten system prompt, lower temperature |
| 8 | 24 | multi_hop | faithfulness | 0.6927 | LLM hallucinating | Tighten system prompt, lower temperature |
| 9 | 5 | factual | faithfulness | 0.7012 | LLM hallucinating | Tighten system prompt, lower temperature |
| 10 | 38 | multi_hop | faithfulness | 0.7060 | LLM hallucinating | Tighten system prompt, lower temperature |

**Phân tích:** Hầu hết bottom 10 có worst_metric = faithfulness, cho thấy LLM sinh ra câu trả lời không khớp với context. Các câu multi_hop và adversarial chiếm đa số trong bottom 10, phản ánh độ khó cao hơn của các câu hỏi này.

## Matrix và nhóm yếu nhất

| Worst Metric / Distribution | factual | multi_hop | adversarial |
|---|---|---|---|
| faithfulness | 3 | 14 | 4 |
| answer_relevancy | 16 | 1 | 1 |
| context_recall | 1 | 4 | 5 |
| context_precision | 0 | 1 | 0 |

**Nhóm yếu theo mean thấp nhất:** adversarial (0.6998)
**Worst metric xuất hiện nhiều nhất:** faithfulness (21 câu)

**Giải thích:** Nhóm adversarial có mean thấp nhất (0.6998) vì các câu hỏi dạng này thường chứa version conflict, negation và multi-hop reasoning. Metric faithfulness là điểm yếu chủ đạo (21/50 câu), cho thấy LLM hay hallucinate khi context không rõ ràng hoặc có mâu thuẫn.

**Đề xuất sửa:**
1. Giảm temperature xuống 0.0-0.2 cho generation
2. Cải thiện chunking để context đầy đủ hơn cho multi-hop
3. Thêm validation layer kiểm tra answer vs context trước khi trả về

## Bẫy adversarial và giới hạn

Adversarial mean (0.6998) thấp hơn factual (0.9112) nhưng không tự chứng minh pipeline đã nhận ra bẫy. Điểm thấp có thể do:
- Version conflict: câu trả lời đúng theo policy cũ nhưng câu hỏi hỏi policy mới
- Negation: câu hỏi phủ định bị hiểu sai
- Multi-hop: cần kết hợp nhiều document

**Giới hạn:** 50 câu là mẫu nhỏ; model gpt-4o-mini có giới hạn reasoning. Không có fallback extractive trong kết quả này.
