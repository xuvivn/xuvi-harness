# Quyết định công nghệ — lần chạy hiện tại

> **Đây là file bạn điền trước mỗi lần xây/chạy.** Sao chép nó, điền mọi ô, và giữ nó ở đầu ngữ cảnh làm việc. Các file chi tiết trong thư mục này giải thích từng lựa chọn; file này chỉ tổng kết.
>
> Để trống một ô = chưa quyết định = **không được viết code phụ thuộc vào nó**.

| # | Quyết định | Lựa chọn của lần này | File chi tiết |
|---|---|---|---|
| 1 | Ngôn ngữ + runtime chính | Swift 6 (SwiftPM, swift-tools 5.10), macOS ≥ 14, arm64 trước; không third-party dependency ở v1 | [runtime-va-ngon-ngu.md](runtime-va-ngon-ngu.md) |
| 2 | Cơ chế module/plugin (framework hay tự viết) | Tự viết tối thiểu theo core/02: context/inject/effect-disposer/dispatch; plugin Waldorf đăng ký vào harness | [runtime-va-ngon-ngu.md](runtime-va-ngon-ngu.md) |
| 3 | Mô hình tiến trình (một tiến trình / host+client / worker) | Một tiến trình trong app macOS; CLI headless là executable thứ hai dùng chung library targets; agent con (worker) cùng tiến trình, session riêng | [runtime-va-ngon-ngu.md](runtime-va-ngon-ngu.md) |
| 4 | Nhà cung cấp + model mặc định | Provider duy nhất `meta`; mặc định `muse-wonder-1.0` vai **orchestrator** (agent chính, việc phức tạp, tích hợp, phán quyết) | [ai-models.md](ai-models.md) |
| 5 | Nhà cung cấp + model phụ (nếu có) | Cùng provider `meta`: `muse-spark-1.3-contributor` vai **worker** (agent con việc tự chứa) và `aux` (tiêu đề/nén); Waldorf capability: `muse-image-1.0`, `muse-voice-transcribe-1.0` | [ai-models.md](ai-models.md) |
| 6 | Model chạy ở đâu (API cloud / tự host / local) | API cloud Meta (`api.meta.ai`) là đường gọi model duy nhất; không local LLM, không provider khác | [ai-model-runtime.md](ai-model-runtime.md) |
| 7 | Cách lấy bí mật API (biến môi trường / kho bí mật OS / OAuth) | Chỉ tham chiếu `META_API_KEY`: biến môi trường tiến trình → file `.env` do chính app/CLI nạp phía runtime (không in/log) → người dùng tự nhập trong Settings lưu Keychain của chính app. Agent làm việc không đọc giá trị bí mật | [ai-model-runtime.md](ai-model-runtime.md) |
| 8 | Mặt tiếp xúc chính (headless / web chat / macOS native / nhúng SDK) | Chính: ứng dụng macOS native SwiftUI thuần (không WKWebView). Phụ: headless CLI cho test/dogfood/CI | [chat-va-tools-integration.md](chat-va-tools-integration.md), [ung-dung-macos-native.md](ung-dung-macos-native.md) |
| 9 | Bộ tool bật mặc định trong chat | File (đọc/ghi/sửa/glob/grep), shell (sandbox-exec), todo, subagent (chỉ orchestrator, độ sâu ≤ 1), hỏi người/approval | [chat-va-tools-integration.md](chat-va-tools-integration.md) |
| 10 | Nguồn tool ngoài (MCP server nào, nếu có) | Không ở v1 (đỗ lại) | [chat-va-tools-integration.md](chat-va-tools-integration.md) |
| 11 | Chính sách approval mặc định (`ask` / `never` / preset) | App: `ask` cho ghi-file/chạy-lệnh, đọc tự do. Headless dogfood: `never` chỉ trong bản sao workspace dùng-một-lần + sandbox-exec | [chat-va-tools-integration.md](chat-va-tools-integration.md) |
| 12 | Định dạng lưu session trên đĩa | Mỗi session một file JSONL append-only trong `~/Library/Application Support/XuViHarness/sessions/`; Waldorf domain: JSON Codable ghi nguyên tử (atomic) + backup, seam persistence thay được (SQLite đỗ lại khi quy mô đòi) | [nen-tang-luu-tru-va-dong-goi.md](nen-tang-luu-tru-va-dong-goi.md) |
| 13 | Cơ chế sandbox cho thực thi của model, theo OS | macOS: Seatbelt qua `sandbox-exec` cho shell tool, báo mức full/partial thành thật; App Sandbox của Apple: tắt ở v1 (hai lớp khác nhau) | [nen-tang-luu-tru-va-dong-goi.md](nen-tang-luu-tru-va-dong-goi.md) |
| 14 | Hệ điều hành đích + cách đóng gói/phát hành | macOS ≥ 14 arm64 trước; SwiftPM → script lắp `.app`, ký ad-hoc chạy local; Developer ID/notarize đỗ lại | [nen-tang-luu-tru-va-dong-goi.md](nen-tang-luu-tru-va-dong-goi.md), [ung-dung-macos-native.md](ung-dung-macos-native.md) |
| 15 | Chiến lược ngữ cảnh | Đếm token xấp xỉ từ log + usage thật của provider làm mốc; cắt kết quả tool (trần cấu hình, mặc định 8000 ký tự/lần, lấy thêm theo tham chiếu) là bậc đầu; agent con cách ly ngữ cảnh cho việc khám phá dài; nén bằng tuyến worker (`aux`) chỉ khi áp lực cao, recall trước độ gọn; ghi chú tiến độ = todo + file ghi chú trong workspace; trần uỷ quyền: sâu 1, ≤ 3 worker song song, ngân sách token/tool riêng từng worker | [ai-model-runtime.md](ai-model-runtime.md), [chat-va-tools-integration.md](chat-va-tools-integration.md) (nguyên tắc: [core/07](../core/07-prompt-va-ngu-canh.md), [core/10](../core/10-phoi-hop-va-mo-rong.md)) |

## Ghi chú riêng của lần chạy này

Bản này là lượt bootstrap: DeepSeek Harness sinh hệ lần đầu; qua cổng dogfood (xem `plugins/waldorf/specs/11_macos-native-swift-va-tich-hop-harness.md` và benchmark của đợt) thì chuyển hẳn sang xuvi-harness. Chỉ chạy máy cá nhân này; bỏ qua đa người dùng, telemetry, deploy.

-

## Lịch sử thay đổi quyết định

| Ngày | Ô # | Từ → Thành | Lý do một dòng |
|---|---|---|---|
| 2026-11-03 | #1–#15 | trống → bộ quyết định Swift/Meta ở trên | Yêu cầu người dùng: macOS native thuần Swift, chỉ META_API_KEY, Wonder orchestrator / Spark Contributor worker, bootstrap trên DeepSeek Harness rồi tự chủ |

> Khi một quyết định đổi giữa chừng: cập nhật bảng trên, rồi kiểm tra mục "Ảnh hưởng lên core" trong file chi tiết của quyết định đó trước khi sửa code.
