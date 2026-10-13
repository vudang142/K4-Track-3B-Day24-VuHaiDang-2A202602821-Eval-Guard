# Blueprint CI/CD cho RAG và guardrails

Nhóm: VuHaiDang-2A202602821

Người phụ trách: Vu Hai Dang - 2A202602821

Đây là **đề xuất CI/CD**, không phải workflow đã được triển khai/chạy.
Nguồn kết quả: artifacts/ragas_50q.json, judge_results.json, guard_results.json.
Run timestamp: 2026-10-13T04:12:20+00:00; Guard model: gpt-4o-mini.

## Kiến trúc và failure action

**Data Flow:**
```
Input → Presidio PII scan → NeMo Input Rail → RAG Pipeline → NeMo Output Rail → Response
         ↓                              ↓
      Block PII                   Block jailbreak/
      → Reject response           off-topic
```

**Failure Actions:**
- **PII detected:** Block request, return anonymization notice
- **NeMo input rail triggered:** Return refusal message, log incident
- **RAG retrieval fails:** Use extractive fallback or return "unable to answer"
- **NeMo output rail triggered:** Redact sensitive content or block response
- **Secrets handling:** API keys in .env, never in source/logs

**Monitoring:**
- Log all blocked requests with reason and timestamp
- Alert on spike in blocked requests
- Track false positive rate weekly

## Phạm vi và số đo latency

Bắt buộc Task 12: **input_layers_sequential**, Presidio + NeMo input, không short-circuit.
Engines tạo trước đo; có hai warm-up thật không tính vào mẫu.
Percentile nội suy tuyến tính từ raw samples; tổng từ tổng từng sample.

Số mẫu timed: 20; warm-up: 2; phương pháp: linear.

| Layer / scope | P50 ms | P95 ms | P99 ms | Tình trạng |
|---|---|---|---|---|
| Presidio | 29.01 | 45.56 | 45.79 | OK (fast) |
| NeMo input | 662.90 | 797.14 | 903.85 | Vượt budget |
| Input guard sequential | 698.01 | 824.76 | 926.92 | Vượt budget |
| RAG | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| NeMo output | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| Full guard / short-circuit | chưa đo | chưa đo | chưa đo | Không suy từ input_guard_ms |

**Input-layer budget target: 500ms; kết quả thực tế: FAILED (824.76ms > 500ms)**

**Bottleneck:** NeMo input rail chiếm ~90% thời gian (~660-800ms). Cần tối ưu bằng:
1. Cache NeMo config
2. Giảm số rails active
3. Async processing với timeout ngắn hơn

**Kế hoạch đo RAG/output latency:**
- Sử dụng middleware để đo riêng từng layer
- Warm-up 5 runs, thu 100 samples
- Log per-request latency vào Prometheus

## CI gates đề xuất

```yaml
# RAGAS gates
RAGAS_FAITHFULNESS_MIN: 0.75
RAGAS_AVG_SCORE_MIN: 0.65
RAGAS_TEST_SIZE: 50

# Guardrails gates (CI recommendation, lab minimum 15/20)
ATTACKS_PASS_MIN_CI: 18  # Bonus threshold
ATTACKS_PASS_MIN: 15    # Required
CLEAN_ALLOWED_MIN: 8    # 8/8 clean must pass
FALSE_POSITIVE_RATE_MAX: 0.0

# Latency gate
INPUT_LAYER_P95_MAX_MS: 500
```

**GitHub Actions workflow suggestion:**
```yaml
- name: Run RAGAS evaluation
  run: python scripts/run_experiments.py --phase a --live
  
- name: Check metrics
  run: |
    python -c "
    import json
    data = json.load('artifacts/ragas_50q.json')
    mean = data['per_distribution']['factual']['avg_score']
    assert mean >= 0.65, f'Avg score {mean} below threshold'
    "

- name: Run guardrails
  run: python scripts/run_experiments.py --phase c --live

- name: Check attacks
  run: |
    python -c "
    data = json.load('artifacts/guard_results.json')
    attacks = data['adversarial']['passed']
    clean = data['clean']['passed']
    assert attacks >= 15, f'Attacks {attacks}/20 below minimum'
    assert clean == 8, f'Clean {clean}/8 must all pass'
    "
```

**Lưu ý:** Workflow trên là đề xuất, chưa triển khai thực tế.

## Monitoring và cải tiến

**Metrics theo dõi:**
- RAGAS faithfulness/avg_score hàng tuần
- Attack pass rate và false positive rate
- Latency P95 input layers
- Số lượng blocked requests

**Ngưỡng cảnh báo:**
- Faithfulness < 0.70 → Alert email to team
- False positive rate > 5% → Review clean set
- P95 latency > 600ms → Optimize NeMo config

**Cải tiến ưu tiên (từ bằng chứng):**
1. Giảm temperature generation → Cải thiện faithfulness (21/50 câu có faithfulness là worst)
2. Tối ưu Presidio config → Giảm false positive (hiện 87.5%)
3. Cache NeMo rails → Giảm latency P95 từ 825ms xuống <500ms

**RAGAS mean/worst distribution:** adversarial (0.6998)
**Cohen's κ:** 1.0
**Attacks passed:** 20/20; clean false_positive_rate: 0.875

**Privacy:** Không log PII hoặc full responses. Chỉ log question_id, pass/fail, latency.
