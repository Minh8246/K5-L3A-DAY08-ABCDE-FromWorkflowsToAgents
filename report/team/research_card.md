# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R2 |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | MAST, NeurIPS 2025, https://arxiv.org/abs/2503.13657, empirical failure taxonomy of multi-agent LLM systems |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Multi-agent systems thường phát sinh lỗi lệch mục tiêu (misalignment) và lỗi kiểm chứng (verification failure) khi tăng số lượng tác tử giao tiếp mà thiếu ranh giới kiểm soát code cứng. |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Kiểm nghiệm trên các hệ thống multi-agent xử lý tác vụ suy luận và cộng tác công cụ; chịu ảnh hưởng bởi năng lực ngữ cảnh của mô hình cơ sở. |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Chuyển từ Workflow (A) sang Swarm (D) sẽ làm tăng chi phí giả lập (dự kiến tăng >50%) và làm tăng variance (từ 0.0 lên >0), có thể xuất hiện lỗi kiểm chứng điều kiện claim ở các seed nhiễu. |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Cố định: MockLLM, bộ 14 claim public, 5 seed (11, 22, 33, 44, 55), bảng giá VND mô phỏng, SQLite store, evaluator. Thay đổi duy nhất: Kiến trúc topology (Workflow A vs Swarm D). |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Lỗi phân loại sai quyết định bồi thường (status resolved != denied trên C-002 ở seed 55). Đo bằng: pass rate (1.000 vs 0.986), pass^5 (1.000 vs 0.929), variance (0.000000 vs 0.000816), cost/success (28.87 vs 49.57 VND). |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json`
- Run id / file report: `reports/benchmark.json`
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: **Đúng**. Kết quả thực nghiệm cho thấy Swarm có chi phí cao hơn 71.7% (49.57 VND so với 28.87 VND của Workflow) và độ biến thiên tăng lên (variance = 0.000816, seed 55 fail claim C-002 với lỗi `status resolved != denied x1`). Trong khi đó, Workflow duy trì độ ổn định tuyệt đối (pass rate 1.000, variance 0.000000).
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
