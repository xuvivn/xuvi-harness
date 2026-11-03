# 15 — Bản đồ đối chiếu với DeepSeek Harness (tham chiếu)

> File này không phải một phần của logic cốt lõi. Nó chỉ đường: mỗi spec generic trong bộ này tương ứng với package/tài liệu nào trong DeepSeek Harness, để bạn đọc cài đặt tham chiếu khi cần. Khi hai bên khác nhau, **spec generic thắng** (xem README, mục "Nguồn gốc").

Kho tham chiếu: repository `deepseek-harness` — `docs/architecture.md` là bản đồ gốc, `docs/subsystems/` là tài liệu từng hệ con, mỗi package có README riêng.

---

## 1. Đối chiếu theo file core

| File specs | DeepSeek Harness tương ứng |
|---|---|
| 01 Tổng quan | `docs/architecture.md`; `docs/cordis-primer.md` (framework plugin bên dưới) |
| 02 Module & plugin | Framework Cordis (context/service/inject/effect, 5 chế độ dispatch); `packages/core/scope` (scope theo agent); `docs/subsystems/scope.md` |
| 03 Agent loop | `packages/core/agent` (giao diện Agent, registry) + `packages/core/agent-loop` (cài đặt); `docs/subsystems/core.md`, `docs/agent-lifecycle.md` |
| 04 Session log | `packages/core/session`; `docs/subsystems/session.md` (SessionEventMap, surface, deriveMessages) |
| 05 Model gateway | `packages/llm/llm` (từ vựng + registry); adapter: `packages/llm/llm-deepseek`, `packages/llm/llm-pi-ai`; retry: `packages/llm/llm-retry`; `docs/subsystems/llm-streaming.md` |
| 06 Tools | `packages/core/tools` + các executor `packages/core/tool-executor-*`; `docs/subsystems/tools.md`, `docs/tool-execution-pipeline.md` |
| 07 Prompt & ngữ cảnh | `packages/core/system-prompt`; `packages/context/*` (ngữ cảnh model thấy: chỉ thị workspace, thời gian, tham chiếu); `docs/subsystems/system-prompt.md`. Ngân sách token và truy hồi đúng lúc còn dựa vào `packages/compaction/*`, `packages/skill/*` và các tool tìm/đọc ở `packages/fs/*` — cơ chế của chúng ở file 10 và 06 |
| 08 Capability seams | `docs/capability-seams.md` (đồ thị seam sinh tự động); các nhóm `packages/fs`, `packages/shell`, `packages/subprocess`, `packages/sandbox`, `packages/web`, `packages/lsp`, `packages/mcp`, `packages/storage`, `packages/credentials`… |
| 09 Tương tác người | `packages/interaction/*` (`user-approval`, `user-questions`, `permission-presets`); `packages/core/commands`-tương đương trong nhóm lệnh; plan mode `packages/plan/*`; `docs/subsystems/approval.md` |
| 10 Phối hợp & mở rộng | `packages/subagent/*`, `packages/jobs/*`, `packages/workflow/*`, `packages/schedule/*`, `packages/goal/*`, `packages/compaction/*`, `packages/skill/*`, `packages/todo/*`, `packages/guard/*`, `packages/extensions/*`; session projection `packages/session/session-projection` |
| 11 Lưu trữ & khôi phục | `packages/session/session-persistence` + `session-persistence-jsonl`; `packages/attachment/*`, `packages/spill/*`, `packages/session-query/*`; `docs/subsystems/persistence.md`, `docs/session-format-status.md` |
| 12 Composition & cấu hình | `packages/boot/app-boot` (profile/bundle/lớp vá), `packages/bundle/*` (base, web-app, headless, sdk-app, sdk-minimal, acp-app), `packages/boot/plugin-manager`, `packages/settings/*`, `packages/preset/*` |
| 13 Bảo mật, lỗi, quan sát | `docs/defensive-patterns/`; sandbox `packages/sandbox/*`; telemetry `packages/telemetry/*`, `packages/session/session-telemetry*` |
| 14 Lộ trình & kiểm thử | `docs/testing.md` (chính sách test, replay snapshot); adapter phát lại `packages/test-support/llm-replay` |

## 2. Đối chiếu các kiểu profile

| Profile trong specs (12, mục 2) | DeepSeek Harness |
|---|---|
| `web` | Profile `web`: bundle base + `dsh-web-app`, host `packages/host/*`, client `packages/client/*`, shell web ở `apps/web` |
| `headless` | Profile `headless`: bundle base + `dsh-headless` |
| `sdk` / `sdk-minimal` | Profile `sdk` / `sdk-minimal`: `packages/sdk/*` (JSON-RPC), `packages/bundle/sdk-minimal` |
| Tự động hoá | Profile `acp`: `packages/acp/*` |
| Desktop | `apps/desktop` (Electron) giữ một profile riêng do ứng dụng sở hữu (`$DSH_HOME/profiles/desktop`) — mẫu tham chiếu cho `technology/ung-dung-macos-native.md` dù công nghệ đóng gói khác nhau |

## 3. Những chỗ specs *cố ý* khác cài đặt tham chiếu

- Specs dùng từ vựng trung lập ("module system", "dịch vụ") thay cho tên framework cụ thể (Cordis) — để bạn được chọn framework khác ở `technology/runtime-va-ngon-ngu.md`.
- Specs không mang các chi tiết thuần-DeepSeek (ví dụ trường mở rộng riêng trên wire API của một nhà cung cấp, cơ chế tải log lên máy chủ của hãng). Khi cần, chúng thuộc về file technology của nhà cung cấp đó, không thuộc core.
- Specs rút gọn một số cơ chế rất sâu của bản gốc (ví dụ header/series ở mức khái niệm) tới mức đủ để cài đặt lại đúng hành vi quan sát được, không chép nguyên văn kiểu dữ liệu.
- Phần kỹ thuật ngữ cảnh (file 07, file 06 mục 2, file 10 mục 6) diễn đạt lại các nguyên tắc từ bài viết của Anthropic về context engineering, viết tool cho agent và hệ nhiều agent (liên kết đầy đủ ở [README, mục "Nguồn gốc"](../README.md)) thành bất biến và tiêu chí nghiệm thu trung lập công nghệ; các con số cụ thể trong bài viết (ví dụ kích thước bản chưng cất của agent con) được specs chuyển thành giới hạn cấu hình được, không chép thành hằng số.
