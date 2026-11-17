# Báo cáo cá nhân — 2A202603661_TranVietThinh

## 1. Thông tin thành viên

- Họ tên: Trần Việt Thịnh
- MSSV: 2A202603661
- Email: thinhkesat@gmail.com
- GitHub: ThinhTran-codee

## 2. Phần việc phụ trách

- **Vai trò:** Ledger/Policy/Crash + Evaluation.
- **Nhiệm vụ:**
  - Viết test kiểm thử các ràng buộc bảo vệ của Host (Policy gate, Approval validity, Order cap, Idempotency).
  - Viết test tích hợp các shape policy (`src/tests/test_shapes_policy_integration.py`).
  - Chạy và đánh giá crash test suite (`reports/crash.json`), bảo đảm đạt chỉ tiêu 20/20.
  - Chạy benchmark đo đạc độ ổn định, chi phí, pass‑rate cho 4 kiến trúc qua 5 seed (`reports/benchmark.json`).
  - Cập nhật mục 4 (Kết quả) và mục 7 (Hạn chế) trong `group_report.md` và cung cấp số liệu thực nghiệm cho `decision_adr.md`.

## 3. Đóng góp cụ thể

- **Test đã thêm:**
  - `src/tests/test_thinh_ledger_policy.py` (5 tests) – kiểm tra các luật host và durability.
  - `src/tests/test_shapes_policy_integration.py` (9 tests, 3 shapes) – kiểm tra tích hợp chính sách qua các shape khác nhau.
- **Crash suite:** `reports/crash.json` – 20/20 thành công.
- **Benchmark:** `reports/benchmark.json` – 4 shape architectures, mỗi seed 5 lần, tổng 20 runs. Kết quả nổi bật:
  - Workflow: pass 1.0, cost 28.87 VND, mean steps 4.43.
  - Single: pass 1.0, cost 30.70 VND, mean steps 8.21.
  - Orchestrator: pass 1.0, cost 40.00 VND, mean steps 2.0.
  - Swarm: pass 0.986, cost 49.57 VND, variance 0.000816 (failure ở seed 55).

## 4. Bằng chứng

- File test: `src/tests/test_thinh_ledger_policy.py` (5 passed).
- File test: `src/tests/test_shapes_policy_integration.py` (9 passed).
- Test suite host: `python -m pytest -q` → 59/59 passed.
- Crash suite: `reports/crash.json` – 20/20 passed.
- Benchmark: `reports/benchmark.json` – chi tiết ở trên.
- Nhánh làm việc: `feat/ledger-tests` (đã merge vào `main`).

## 5. Công cụ AI đã dùng

- **Công cụ:** Antigravity AI Assistant.
- **Mục đích sử dụng:** Phân tích quy tắc trong `src/lab8/policy.py` và `src/lab8/durability.py` để phát hiện các edge case chưa có test bao phủ.
- **Kiểm chứng:** Tự chạy kiểm thử độc lập trong môi trường ảo `.venv` bằng pytest, kiểm tra các câu lệnh SQL và logic assertion trước khi xác nhận.

## 6. Tự đánh giá

- Hoàn thành đầy đủ các yêu cầu
- Đạt 0 lỗi trong toàn bộ test suite, crash suite và benchmark.
- Đảm bảo tính ổn định và hiệu suất cho các kiến trúc mới (Workflow, Single, Orchestrator, Swarm).
- Đóng góp các test mới, báo cáo kết quả, và hỗ trợ nhóm hoàn thiện báo cáo nhóm.
