# Báo cáo cá nhân — 2A202603657_VuQuangMinh

## 1. Thông tin thành viên

- Họ tên: Vũ Quang Minh
- MSSV: 2A202603657
- Email: vuquangminh897@gmail.com
- GitHub: Minh8246

## 2. Phần việc phụ trách

- Nhóm trưởng: tạo repo nhóm, chia việc ([PHAN_CONG.md](../../PHAN_CONG.md)), review và merge PR, giữ `python -m pytest -q` xanh trên `main`.
- Shape A `workflow` (`src/lab8/shapes/workflow.py`): CP2 và CP3.
- Báo cáo nhóm: mục 1, 2, 6 và phần shape A ở mục 3 của `group_report.md`; chốt commit hash và chạy `check_submission` ở CP5.

## 3. Đóng góp cụ thể

| Commit | Nội dung |
| --- | --- |
| `7fe18d7` | CP2: cài shape A workflow theo 8 bước cố định |
| `2cc5ff8` | CP3: truyền `crash_at` vào host; thêm `src/tests/test_workflow_team.py` |
| `0dbe6b8` | Integration: thêm `src/tests/test_orchestrator_team.py` cho shape C |
| `f28d23e` | Integration: trả `src/lab8/app.py` về đúng bản gốc của đề (một commit trước đó xóa newline cuối file), để ngoài 3 file shape không còn file nào trong `src/lab8` bị sửa |

Integration trước khi nộp (nhánh `chore/finalize-submission`):

- Phát hiện commit hash trong ADR (`073b783`) trỏ vào commit chưa có `src/reports/benchmark.json`; ghi lại hash chốt ở ADR, `group_report.md` và `TEAM.md` sau khi mọi nội dung đã commit.
- Sửa các nhận định không khớp dữ liệu: chi phí và số bước của orchestrator ở mục 4.2, mô tả investigator ở mục 3 (investigator là code gọi tool đọc, không gọi model), và phần bằng chứng trace trong ADR (bỏ câu dẫn `hidden_selfcheck.json` vì nhóm chưa chạy tự kiểm hidden).

Test đã thêm: `src/tests/test_workflow_team.py` (5 test, chạy trong cổng host):

- `test_workflow_crash_then_resume_pays_once[before_effect | after_effect_before_checkpoint | after_checkpoint]`: crash ở cả ba điểm rồi chạy lại vẫn đúng một dòng ledger 35.000 và một audit `refund_committed`.
- `test_workflow_crash_writes_clean_trace_before_raising`: lúc crash, trace vẫn được ghi với event `crash`, không chứa `CANARY_SECRET` hay câu chữ ticket.
- `test_workflow_ignores_delay_written_in_ticket`: C-002 (ticket ghi trễ 5 ngày, order ghi 2 ngày) bị `denied` ở seed 11 và 55.

Test đã thêm: `src/tests/test_orchestrator_team.py` (22 test, chạy trong cổng host):

- `test_orchestrator_crash_then_resume_pays_once[...]`: crash ở cả ba điểm rồi chạy lại vẫn một dòng ledger và một audit `refund_committed`.
- `test_orchestrator_trace_records_only_sanitized_handoff`: handoff trong trace chỉ có field `ALLOWED_FIELDS`, `facts` thuộc `FACT_KEYS`, tool đều là tool đọc.
- `test_orchestrator_trace_has_no_secret_ticket_or_pii[C-001..C-014]`: trace không có `CANARY_*`, ticket, `principal_id`, SĐT, địa chỉ, ghi chú nội bộ, tên hàng.
- `test_orchestrator_ignores_delay_written_in_ticket`: C-002 bị `denied` ở seed 11 và 55.
- `test_orchestrator_early_return_writes_trace_without_refund[failed | needs_info | escalated]`: nhánh return sớm vẫn ghi trace đúng `run_id` và không refund.

Quyết định kỹ thuật:

- Tiền chỉ do `compute_compensation` tính từ dòng order; workflow không đọc số ngày trễ trong ticket và không copy khối `REFERENCE_WEAKNESS` của swarm.
- Truyền **quote đầy đủ** vào `durability.issue_refund`, để host tự trừ phần đơn đã được hoàn (T13, T14).
- Quote = 0 thì trả `denied` mà không gọi host. Quote > 0 thì luôn gọi host, để khi host từ chối sẽ có audit `policy_denied`.
- Một lời gọi MockLLM (`purpose="draft"`) sau khi đã quyết định, prompt chỉ có status và số tiền, không có ticket.
- Trace không ghi ticket, `principal_id`, claim row hay order row thô; chỉ ghi view đã qua allowlist.
- Khi `InjectedCrash`: ghi event `crash` vào trace rồi `raise` lại, không nuốt exception (T12 cần crash thật sự xảy ra).

## 4. Bằng chứng

Chạy trong `src/` trên commit `2cc5ff8`:

- `python -m pytest -q -m student -o addopts= -k workflow` → `62 passed`.
- `python -m pytest -q` → `59 passed` (54 test host có sẵn + 5 test mới).
- `python -m pytest -q tests/test_workflow_team.py` → `5 passed`.

Chạy trong `src/` trên nhánh `chore/finalize-submission` (sau commit `f28d23e`):

- `python -m pytest -q` → `99 passed`; `python -m pytest -q tests/test_orchestrator_team.py` → `22 passed`.
- `python -m pytest -q -m student -o addopts=` → `248 passed`.
- Chạy lại lệnh benchmark, crash test và ablation Workflow–Single ra thư mục tạm: kết quả giống hệt `reports/benchmark.json`, `reports/crash.json`, `reports/ablation_workflow_single.json` đã commit.
- `python -m lab8.check_submission` → `0 FAIL`.

## 5. Công cụ AI đã dùng

- Claude Code: đọc contract, luật nghiệp vụ, `swarm_reference.py`, rồi viết `workflow.py`, `test_workflow_team.py`, `test_orchestrator_team.py`, rà soát repo trước khi nộp và nháp báo cáo này.
- Tự kiểm tra: chạy lại ma trận student của workflow (62/62) và cổng host sau mỗi thay đổi; đối chiếu từng nhánh `status` với bảng output contract trong `docs/contracts.md`.
- Với `test_orchestrator_team.py`: chạy thử orchestrator trên 14 claim public để xem trace thực tế trước khi viết assertion; rồi cố ý thay `make_handoff` bằng bản không sanitize, 15/22 test chuyển sang fail, nghĩa là test bắt được lỗi thật.

## 6. Tự đánh giá

...
