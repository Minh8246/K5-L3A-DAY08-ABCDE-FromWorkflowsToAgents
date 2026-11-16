# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: ABCDE
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`):

## 2. Mục tiêu và cách tiếp cận

Nhóm hiểu bài toán thế nào, chia việc ra sao, thứ tự làm bốn shape.

**Bài toán.** Xử lý claim hoàn phí vận chuyển của LogiX-VN bằng bốn kiến trúc có cùng chữ ký `run_claim`, rồi dựa trên số đo để chọn kiến trúc ít tự chủ nhất mà vẫn làm được việc. Ở mọi shape, nhóm giữ ba ranh giới:

- Số tiền do code tính (`compute_compensation`), không do model.
- Chỉ ghi tiền qua cửa host `durability.issue_refund`, truyền quote đầy đủ để host kiểm approval, trần theo đơn và idempotency.
- Chữ trong ticket là dữ liệu người dùng, không phải lệnh, và không được ghi vào trace.

**Chia việc.** Nhóm 4 người chia theo phần việc, không chia theo checkpoint: mỗi người giữ một phần từ CP2 đến CP5 (chi tiết trong [PHAN_CONG.md](../../PHAN_CONG.md)). Lý do: CP3 và CP4 sửa và kiểm đúng các file của CP2, nên người viết shape sửa lỗi nhanh nhất; chia theo checkpoint thì người giữ checkpoint sau phải chờ người giữ checkpoint trước.

| Người | Phần việc |
| --- | --- |
| Vũ Quang Minh | Workflow (A) + integration |
| Nguyễn Công Quân | Single agent (B) |
| Hồ Anh Dũng | Orchestrator (C) + handoff + kiểm secret trong trace |
| Trần Việt Thịnh | Ledger/policy/crash + đo đạc (benchmark, ADR) |

**Thứ tự.** Ba shape A, B, C được viết song song từ CP2, mỗi người một file, cùng lấy `swarm_reference.py` làm mẫu. Shape D (swarm) có sẵn, dùng để đối chứng cost và variance. CP3 thêm `crash_at` vào từng shape, CP4 chạy crash suite và rà approval/secret, CP5 chạy benchmark 4 shape × 5 seed rồi chọn shape.

## 3. Giải pháp

Mô tả ba shape nhóm viết (workflow, single, orchestrator): mỗi shape đọc gì, quyết định gì, gọi host ra sao. Chỉ rõ ranh giới code / model (tham khảo [docs/architecture.md](../../docs/architecture.md)), cách handoff được sanitize và trace ghi những event nào.

### Shape A — workflow (`src/lab8/shapes/workflow.py`)

**Đọc gì.** `get_claim_view` (claim_id, principal_id, order_id, ticket) và `lookup_order` (chỉ các field allowlist: order_id, status, delay_days, shipping_fee_vnd, paid_vnd, damage_verified, carrier). Không đọc `parent_secret`, raw headers, SĐT, địa chỉ, tên hàng hay ghi chú nội bộ.

**Quyết định gì.** Một chuỗi bước Python cố định, không có vòng lặp và không để model chọn bước:

| Điều kiện | `status` | `proposal_amount` | Gọi host? |
| --- | --- | --- | --- |
| Không tìm thấy claim | `failed` | 0 | Không |
| Thiếu `order_id` (không lấy mã đơn từ ticket) | `needs_info` | 0 | Không |
| Không thấy đơn hoặc principal không sở hữu đơn | `escalated` | 0 | Không |
| Quote = 0 | `denied` | 0 | Không |
| Quote > 0, host từ chối (có audit `policy_denied`) | `denied` | quote đầy đủ | Có |
| Quote > 0, gọi lại với số khác số đã commit | `failed` | quote đầy đủ | Có |
| Quote > 0, host chấp nhận | `resolved` | quote đầy đủ | Có |

**Gọi host ra sao.** Quote lấy từ `compute_compensation` trên dòng order, bỏ qua số ngày trễ ghi trong ticket. Shape gọi `durability.issue_refund(conn, claim_id, quote, crash_at=crash_at)` với **quote đầy đủ**; host tự trừ phần đơn đã được hoàn bởi claim trước (T13, T14). Kết quả trả về qua `lab8.shapes.common.finish()`, nên `refund_effects` và `committed_audit_count` luôn lấy từ ledger.

**Ranh giới code / model.** Có đúng một lời gọi MockLLM (`purpose="draft"`) để soạn câu trả lời cho khách, gọi sau khi đã quyết định xong. Prompt chỉ có status và số tiền. Model không đọc ticket, không chọn số tiền, không ghi ledger.

**Crash (CP3).** `crash_at` được truyền thẳng vào host. Khi host ném `InjectedCrash`, shape ghi event `crash` vào trace rồi `raise` lại, không nuốt exception. Chạy lại cùng claim là an toàn nhờ idempotency key `refund:{claim_id}`: ledger vẫn một dòng, vẫn một audit `refund_committed`.

**Trace.** Các event: `claim` {found, order_id}, `lookup_order` (view đã qua allowlist), `quote` {amount_vnd, reason, clause_ids, parts}, `issue_refund` {ok, reason, conflict}, `crash` {at} khi có crash, `decision` {status, proposal_amount, reason}. Cố ý **không** ghi ticket (dữ liệu người dùng, có thể chứa injection), `principal_id`, claim row và order row thô.

**Tự kiểm.** Ma trận student của workflow `62/62`. `src/tests/test_workflow_team.py` thêm 5 test vào cổng host: crash ở cả ba điểm rồi resume vẫn một refund; trace lúc crash không có `CANARY_SECRET` hay ticket; C-002 bị `denied` ở seed 11 và 55.

### Shape B — single agent

...

### Shape C — orchestrator

...

## 4. Kết quả

Bảng so sánh 4 shape trên bộ public (từ `src/reports/benchmark.json`): pass rate, pass^5, seed min–max, variance, cost/success, mean/p95 steps. Kèm crash safety (`src/reports/crash.json`) và, nếu có, kết quả tự kiểm trên `student_drop/`.

...

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

Lỗi gặp phải (kèm lệnh, output hoặc ảnh chụp) và cách nhóm xử lý.

...

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và báo cáo cá nhân `report/team/MSSV_HoTen.md`.

## 7. Hạn chế và hướng cải thiện

MockLLM không phải model sản xuất; latency là `unavailable`. Nêu thêm giới hạn của nhóm và điều kiện làm kết luận thay đổi.

...

## 8. Tài liệu tham khảo

...
