# Đánh giá LLM Judge Phase B

Nhóm: VuHaiDang-2A202602821

Người phụ trách: Vu Hai Dang - 2A202602821

Nguồn: `artifacts/judge_results.json`; run timestamp: 2026-10-13T04:11:10+00:00.
Generation model: gpt-4o-mini-2024-07-18; Judge model: gpt-4o-mini.

## Năm cặp A/B và hai lượt

| ID | Question | Winner Pass1 | Winner Pass2 | Final | Consistent |
|---|---|---|---|---|---|
| 1 | Nghỉ bao nhiêu ngày khi kết hôn? | B | B | B | ✅ |
| 5 | Mua thiết bị 55 triệu cần ai phê duyệt? | A | A | A | ✅ |
| 12 | Thưởng Tết tối thiểu 6 tháng+? | A | tie | tie | ❌ |
| 21 | Thử việc có được thưởng Tết? | B | B | B | ✅ |
| 41 | VPN có được dùng cho WFH? | B | B | B | ✅ |

**Position bias count/rate:** 1/5 (20%)
**Verbosity win rate:** 0.75 (3/4 decisive cases)

**Giải thích:** Case ID=12 có position inconsistent (tie ở pass2), cho thấy swap không hoàn toàn loại bỏ bias. Verbosity bias cao (75%) gợi ý LLM có xu hướng thích câu trả lời dài hơn. Cần cẩn thận khi so sánh answers có độ dài khác biệt lớn.

## Calibration đúng model_answer

| ID | Question | Judge Label | Human Label | Agree |
|---|---|---|---|---|
| 1 | Nghỉ bao nhiêu ngày khi kết hôn? | 1 | 1 | ✅ |
| 2 | Thời hạn hợp đồng thử việc? | 0 | 0 | ✅ |
| 3 | Lương thử việc có phụ cấp? | 1 | 1 | ✅ |
| 4 | Thưởng Tết cho ai? | 1 | 1 | ✅ |
| 5 | Mua thiết bị 55 triệu cần ai phê duyệt? | 1 | 1 | ✅ |
| 6 | Thưởng Tết có tính vào thuế? | 0 | 0 | ✅ |
| 7 | Lương tháng 13 có phải thuế? | 1 | 1 | ✅ |
| 8 | Nghỉ phép có lương khi tang? | 0 | 0 | ✅ |
| 9 | Nghỉ sinh có lương? | 1 | 1 | ✅ |
| 10 | Bảo hiểm có tự động gia hạn? | 0 | 0 | ✅ |

**Cohen's κ: 1.0**
**Observed agreement: 10/10 = 100%**
**Chance agreement: ~50%**
**Diễn giải:** Perfect agreement (κ=1.0) cho thấy LLM judge hoàn toàn nhất quán với human labels. Tuy nhiên, với chỉ 10 samples, đây có thể là kết quả may rủi. Cần thêm dữ liệu để kết luận chắc chắn.

## Giới hạn và cách dùng judge

- **Model/prompt:** gpt-4o-mini với system prompt về đánh giá RAG
- **Số lượng nhỏ:** 10 labels là không đủ để kết luận về độ tin cậy tổng thể
- **κ chưa đạt >0.6:** Thực ra κ=1.0 đạt, nhưng cần replication với bộ lớn hơn
- **Case lệch:** Không có case lệch trong 10 này, nhưng có thể xảy ra với edge cases

**Cách dùng judge đúng:** Judge là công cụ hỗ trợ, không thay thế human evaluation. Cần kết hợp với qualitative review trước khi đưa ra kết luận.
