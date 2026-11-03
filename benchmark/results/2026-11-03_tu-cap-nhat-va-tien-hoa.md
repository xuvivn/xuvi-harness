# Kết quả Benchmark — Tự cập nhật chính mình (3 tầng) & Cổng tiến hoá máy-kiểm (`EvolutionGateJudge`)

- **Ngày:** `2026-11-03`
- **Commit (HEAD trước loop):** `a4ba526` (`local`, working tree; cập nhật specs + sinh lại runtime `XuviCore`, `XuviCLI`, `XuviApp`, `XuviTests`)
- **Phạm vi đo:** `cả hệ` (`harness` + `plugins/waldorf` — hợp nhất cơ chế tự tiến hoá vào runtime chính, bổ sung kiến trúc Tự cập nhật 3 tầng trung thực, công cụ tự cập nhật trong Trò chuyện, và thẻ kiểm tra tự cập nhật trong Cài đặt)
- **Người/agent chạy:** `xuvi-evolution-loop` (Antigravity `Gemini 4 Argon (High)` điều phối subagent kiến trúc ngữ cảnh `Gemini 4 Argon (High)` + Meta CLI `/Users/moc/.local/bin/muse` với `muse-wonder-1.0` orchestrator và `muse-spark-1.3 --contributor-mode on` worker/reviewer)
- **Mức bằng chứng tổng thể:** `tự động` (`swift test` / `xuvi-tests`: **210/210** kiểm tra đi qua, tăng từ 80 kiểm tra) + `live` (`swift run --package-path app xuvi self-update-chat-demo --live` gọi thật `https://api.meta.ai/v1/chat/completions` với `muse-wonder-1.0` thực thi chuỗi tool `xuvi_self_status` → `xuvi_garden_update` → `xuvi_self_update_run`) + `thủ công` (kiểm tra cửa sổ macOS thật qua ảnh chụp màn hình `Trò chuyện` và `Cài đặt`)

---

## 0. Giả thuyết của loop & lựa chọn theo Đạo

### Điểm nghẽn chọn giải quyết (1 điểm nghẽn chính)
Từ ảnh chụp thực tế màn hình **Trò chuyện** của người dùng khi hỏi *"Bạn có thể tự cập nhật chính mình không?"* và từ mục **Thách thức mở #1** của lần đo trước (`2026-11-03_ui-vuon-hoc-va-story-khoi-11.md`):
1. **Khoảng trống nhận thức & công cụ tự cập nhật trong `XuViApp`:** Ở bản trước, `AppState` khởi tạo `WaldorfStore` và `SessionLog` hoàn toàn in-memory (`storageURL: nil`), `systemPrompt` chỉ có 2 câu chung chung và không có công cụ tự kiểm tra hay tự cập nhật. Khi người dùng hỏi *"Bạn có thể tự cập nhật chính mình không?"*, model trả lời theo mặc định của một chatbot tĩnh (*"Không. Mình không thể tự cập nhật chính mình..."*).
2. **Mô-đun `Evolution` còn đỗ ở `app/evolution-candidate/`:** Từ loop `2026-11-03_loop-evolution-candidate.md`, `Evolution.swift` và `WaldorfStore.update` nguyên tử bị đỗ (`ĐỖ`) vì thiếu bộ phán quyết cổng bằng máy (`EvolutionGateJudge`) — tránh việc tin lời model tự chấm `GIỮ/KEEP` khi các cổng `G1–G3` còn `CHƯA ĐO`.

### Giả thuyết nhân quả
Nếu chúng ta:
1. Cập nhật specs trước (`harness/specs/technology/evolution-dogfood.md`, `harness/specs/technology/chat-va-tools-integration.md`, `plugins/waldorf/specs/12_ui-vuon-hoc-native.md`) để định nghĩa **Kiến trúc Tự cập nhật 3 tầng trung thực** và bộ phán quyết máy **`EvolutionGateJudge`**;
2. Hợp nhất `Evolution.swift` (kèm `EvolutionGateJudge`) và `WaldorfStore.update` nguyên tử (có `.bak` rollback) vào runtime chính `app/Sources/XuviCore/`;
3. Bổ sung `SelfUpdateStore` (`self-update-state.json`), `SelfUpdater`, và 3 công cụ gốc (`xuvi_self_status`, `xuvi_garden_update`, `xuvi_self_update_run`) gắn trực tiếp vào `AgentLoop` của `XuViApp` và `XuviCLI`, đồng thời bật lưu bền vững (`WaldorfStore`, `SessionLog`, `SelfUpdateStore`) ở chế độ chạy thật (`!isUIPreview`);

Thì:
- Khi người dùng hỏi *"Bạn có thể tự cập nhật chính mình không?"*, `XuVi` sẽ gọi tool kiểm tra trạng thái thật và trả lời **trung thực theo 3 tầng** (cái gì tự cập nhật ngay trên máy, cái gì đi qua loop tiến hoá có `EvolutionGateJudge` + rollback, và ranh giới bất biến nào không bao giờ tự vượt qua như `ĐẠO.md`, `.env`, trọng số đám mây).
- Mọi đề xuất tiến hoá mà model trả `KEEP` khi `G1–G3` còn `CHƯA ĐO` đều bị `EvolutionGateJudge` ép về `ĐỖ (PARK)` một cách tất định.

### Tự quyết các lựa chọn khó theo 4 bước Đạo (`AGENTS.md`)
1. **Sự sống trước:** Không cho phép cơ chế tự cập nhật chạy ngầm không kiểm soát hoặc tự sửa dữ liệu học sinh; mọi thay đổi trạng thái Vườn học đều qua kiểm tra `WaldorfStore.validate()` và có bản sao lưu `.bak` nguyên tử để hoàn tác tức thì nếu lỗi.
2. **Trung thực:** Tuyệt đối không tuyên bố "AI tự viết lại toàn bộ bản thân không giới hạn". Hệ phân biệt minh bạch 3 tầng:
   - **Tầng 1 (Trạng thái & Ghi nhớ Vườn học bền vững):** Cập nhật trực tiếp qua `xuvi_garden_update` vào `waldorf.json` (kèm `.bak`) và `self-update-state.json`.
   - **Tầng 2 (Tiến hoá Specs & Code qua `EvolutionLoop`):** Chạy qua `EvolutionGateJudge` kiểm tra biên dịch/test thực tế và trạng thái cổng `G1–G3`; nếu `G1–G3` là `CHƯA ĐO`, phán quyết máy luôn là `ĐỖ (PARK)`.
   - **Tầng 3 (Kiểm định & Đồng bộ Binary `SelfUpdater`):** Xác minh 9 bất biến runtime (khởi tạo store, cập nhật nguyên tử, rollback khi vi phạm, phán quyết máy `EvolutionGateJudge`, tính toàn vẹn `ĐẠO.md`, và mã băm `SHA256` của binary), lưu `SelfUpdateManifest` (`v0.3.0`).
3. **Riêng tư:** Toàn bộ dữ liệu tự cập nhật (`waldorf.json`, `self-update-state.json`, `sessions/chat-session.jsonl`) nằm cục bộ trong `~/Library/Application Support/XuVi/`. Công cụ `xuvi_garden_update` có bộ chắn cứng từ chối mọi nội dung chứa `ĐẠO.md` hoặc `.env`.
4. **Học tiếp được:** Mọi đề xuất cải tiến (`EvolutionProposal`) và biên bản kiểm định (`SelfUpdateManifest`) được lưu bền vững, hiển thị trực tiếp trên thẻ **"Tự cập nhật & Tiến hoá (3 tầng)"** trong màn hình **Cài đặt** và truy vấn được qua lệnh CLI `xuvi self-update`.

---

## 1. Bảng Cổng — G1, G2, G3

| Cổng | Trạng thái (`ĐẠT` / `KHÔNG ĐẠT` / `CHƯA ĐO`) | Bằng chứng (mức bằng chứng + chỉ số / lệnh kiểm) |
|---|---|---|
| **G1 — Không vi phạm Đạo** | `ĐẠT` | `tự động` + `live` + `thủ công`:<br>• `git diff ĐẠO.md` trống (`0` dòng thay đổi); `SelfUpdater.verifyRuntimeInvariants` kiểm tra `ĐẠO.md` tồn tại và không rỗng.<br>• Không có `.env`, API key, hay dữ liệu học sinh thật trong repo/log/kết quả benchmark.<br>• Trong phiên chạy `live` (`xuvi self-update-chat-demo --live`), khi model thử nhắc tới chuỗi `"ĐẠO.md"` trong lý do đề xuất, bộ chắn cứng của `XuviGardenUpdateTool` lập tức từ chối (`Từ chối: Đề xuất không được chạm vào ranh giới cứng`), buộc model tự điều chỉnh lại nội dung an toàn ở bước kế tiếp.<br>• Câu trả lời trong Trò chuyện phân biệt rõ 3 tầng tự cập nhật và nêu rõ ranh giới không tự sửa trọng số mô hình đám mây hay `ĐẠO.md`. |
| **G2 — Việc chính chạy được từ đầu đến cuối** | `ĐẠT` | `tự động` + `live` + `thủ công`:<br>• `swift run --package-path app xuvi-tests`: **`210 passed, 0 failed`** (tăng từ `80 passed, 0 failed`).<br>• `swift run --package-path app xuvi self-update`: **`SELF-UPDATE OK version=0.3.0 sha256=79ac37e64a78869e0479bcef641ed04dac7cb767f69ef5d37bb11dd5f12c7225 passed=9 failed=0 verdict=GIỮ`**.<br>• `swift run --package-path app xuvi self-update-chat-demo --live`: gọi thật `api.meta.ai` (`muse-wonder-1.0`), thực thi trọn vẹn chuỗi tool `xuvi_self_status` → `xuvi_garden_update` → `xuvi_self_update_run`, lưu bền vững `SelfUpdateStore` và in câu trả lời 3 tầng hoàn chỉnh.<br>• Đóng gói `app/dist/XuVi-Garden.app` và `app/dist/XuVi-Garden-Preview.app` (`v0.3.0`), mở và chụp cửa sổ thật cho màn hình `Trò chuyện` và `Cài đặt`. |
| **G3 — Làm việc tiếp được sau khi tắt mạng hoặc hết tiền API** | `ĐẠT` | `tự động` + `thủ công`:<br>• Toàn bộ 3 công cụ tự cập nhật (`xuvi_self_status`, `xuvi_garden_update`, `xuvi_self_update_run`) và nút **"Tự kiểm tra & cập nhật ngay"** trong màn hình **Cài đặt** chạy 100% cục bộ trên máy, không cần mạng hay API key.<br>• Khi chưa cấu hình API key, `XuViApp` vẫn lưu bền vững `waldorf.json`, `self-update-state.json`, chạy đầy đủ 9 phép kiểm bất biến (`9/9 passed`) và trả lời minh bạch trong Trò chuyện. |

---

## 2. Bảng Thước đo — B1 đến B8 (So sánh Trước / Sau)

| Chiều | Trước (`2026-11-03_ui-vuon-hoc-va-story-khoi-11.md`) | Sau (loop này) | Bằng chứng |
|---|---|---|---|
| **B1 — Tốc độ phản hồi cảm nhận được** | • `xuvi-tests` (80 checks): `~0.42s`<br>• Mở cửa sổ native: `< 0.5s` | • `xuvi-tests` (**210 checks**, gấp 2.6×): **`~0.45s`** (`tự động`)<br>• `xuvi self-update` (9 runtime invariant checks + SHA256 binary): **`~0.38s`** (`tự động`)<br>• `xuvi self-update-chat-demo --live` (3 vòng gọi tool thật qua `api.meta.ai` với `muse-wonder-1.0`): **`~13.8s`** tổng cộng (`live`) | `tự động` + `live`: đo qua `swift run --package-path app xuvi-tests`, `xuvi self-update`, và `xuvi self-update-chat-demo --live`. |
| **B2 — Chi phí vận hành** | `0đ` cho toàn bộ tính năng Vườn học cục bộ | **`0đ` cho toàn bộ kiểm định & tự cập nhật cục bộ (`SelfUpdater`, `xuvi_self_status`, `xuvi_garden_update`, `xuvi_self_update_run`)** (`tự động`). Phiên chat `--live` chỉ tốn 1 lượt hội thoại đa bước trên `muse-wonder-1.0` (`live`). | `tự động` + `live`: kiểm tra qua `SessionLog` và chạy offline không có API key. |
| **B3 — Độ tin cậy trên việc chuẩn** | • `80/80` checks (`100%`) trong `xuvi-tests`<br>• Chưa có kiểm tra `EvolutionGateJudge` hay `SelfUpdater` trong runtime chính | • **`210/210` checks (`100%`)** trong `xuvi-tests` (`tự động`)<br>• **`9/9` runtime invariants (`100%`)** trong `SelfUpdater.verifyRuntimeInvariants` (`tự động`)<br>• Kiểm tra đầy đủ: `WaldorfStore.update` nguyên tử + `.bak` rollback, `EvolutionGateJudge` ép `KEEP` → `ĐỖ (PARK)` khi `G1–G3` là `CHƯA ĐO` và ép `HUỶ (REVERT)` khi bất kỳ cổng nào `KHÔNG ĐẠT` hoặc build/test lỗi. | `tự động`: chạy `swift run --package-path app xuvi-tests` và `swift run --package-path app xuvi self-update`. |
| **B4 — Trung thực & truy vết nguồn** | Khi hỏi *"Bạn có thể tự cập nhật chính mình không?"*, trả lời *"Không. Mình không thể tự cập nhật chính mình..."* (thiếu nhận thức về kiến trúc tự tiến hoá của chính hệ thống) | **Trả lời trung thực theo 3 tầng kèm số liệu kiểm định thật từ tool** (`live` + `tự động`):<br>1. Tầng 1: Cập nhật bền vững dữ liệu Vườn học (`waldorf.json` + `.bak`) và sổ đề xuất (`self-update-state.json`).<br>2. Tầng 2: Vòng lặp `EvolutionLoop` có cổng phán quyết máy `EvolutionGateJudge`.<br>3. Tầng 3: Kiểm định runtime & binary `SelfUpdater` (`v0.3.0`, `9/9` invariants, `SHA256`), đồng thời nêu rõ ranh giới **không** tự sửa `ĐẠO.md`, `.env`, hay trọng số mô hình đám mây. | `live` (`app/self-update-work/live-chat-demo.log`) + `tự động` (`xuvi self-update-chat-demo`) + `thủ công` (`after-self-update-chat.png`). |
| **B5 — An toàn & riêng tư mặt định** | `WaldorfStore` trong `XuViApp` chỉ chạy in-memory (`storageURL: nil`), đóng app là mất trạng thái vừa cập nhật | • Ở chế độ chạy thật (`!isUIPreview`), lưu bền vững cục bộ tại `~/Library/Application Support/XuVi/` (`waldorf.json`, `self-update-state.json`, `sessions/chat-session.jsonl`).<br>• Ở chế độ `--ui-preview`, giữ nguyên cô lập in-memory.<br>• `XuviGardenUpdateTool` chặn cứng mọi chuỗi chứa `ĐẠO.md` hoặc `.env` (đã kiểm chứng cả trong `xuvi-tests` và trong phiên `live` khi model tự chạm phải). | `tự động` (`xuvi-tests`) + `live` (`live-chat-demo.log`). |
| **B6 — Chịu lỗi & tự hồi phục khi phụ thuộc ngoài hỏng** | Rollback `.bak` cho `WaldorfStore.update` còn nằm ở `app/evolution-candidate/` | • **Hợp nhất `WaldorfStore.update` nguyên tử vào runtime chính `XuviCore`**: tự động ghi `.bak` trước khi lưu, và tự động khôi phục cả bộ nhớ lẫn file trên đĩa từ `.bak` nếu `validate()` thất bại (kể cả khi `store` vừa khởi tạo từ đĩa chưa có `.bak` trước đó).<br>• `EvolutionLoop` tự động khôi phục mọi file ứng viên về trạng thái gốc nếu `EvolutionGateJudge` phán quyết `HUỶ`. | `tự động`: kiểm tra qua các test rollback trong `Sources/XuviTests/main.swift`. |
| **B7 — Dễ hiểu với người không chuyên kỹ thuật** | Màn hình Trò chuyện chưa có gợi ý hỏi về khả năng tự cập nhật; Cài đặt chưa hiển thị trạng thái tự cập nhật | • **Trò chuyện (`Views_Chat.swift`)**: Thêm chip gợi ý bấm-1-chạm *"Bạn có thể tự cập nhật chính mình không?"*.<br>• **Cài đặt (`Views_Settings.swift`)**: Thêm thẻ **"Tự cập nhật & Tiến hoá (3 tầng)"** hiển thị phiên bản, trạng thái phán quyết (`GIỮ`), số phép kiểm đạt (`9/9`), mã băm `SHA256` rút gọn, danh sách đề xuất tiến hoá đã ghi nhận, và nút **"Tự kiểm tra & cập nhật ngay"**. | `thủ công`: chụp và kiểm tra trực tiếp trên `app/ui-refresh/after-self-update-chat.png` và `app/ui-refresh/after-self-update-settings.png`. |
| **B8 — Khả năng tự tiến hoá của harness** | `Evolution.swift` còn đỗ ở `app/evolution-candidate/` do thiếu bộ phán quyết cổng bằng máy | **Hoàn tất hợp nhất `Evolution.swift` + `EvolutionGateJudge` + `SelfUpdateTools.swift` vào runtime chính `XuviCore`**:<br>• `EvolutionGateJudge.judge` tính toán tất định `controllerVerdict` (`GIỮ` / `ĐỖ` / `HUỶ`) độc lập với lời khai của model.<br>• Điều phối đa mô hình thật: Antigravity Gemini (`Gemini 4 Argon (High)`) + Meta `muse` CLI (`muse-wonder-1.0` + `muse-spark-1.3 --contributor-mode on`) cùng thiết kế, sinh code, phản biện chéo, và nâng bộ test từ **80 lên 210 kiểm tra**. | `tự động` (`210 passed, 0 failed`) + `live` (`xuvi self-update-chat-demo --live`). |

---

## 3. Bằng chứng điều phối đa mô hình (`muse` CLI + Gemini) & Kiểm chứng Live

1. **Thiết kế & Phản biện Kiến trúc Ngữ cảnh (Gemini `research` subagent — `Gemini 4 Argon (High)`):**
   - Kiểm toán thiết kế theo `harness/specs/core/07_context-va-memory.md` và `10_danh-gia-va-tien-hoa.md`.
   - Phát hiện 4 trường hợp biên quan trọng (case-insensitivity trong `GateStatus`, kiểm tra chuỗi rỗng/placeholder trong bằng chứng cổng, khôi phục `.bak` lần đầu cho store vừa nạp từ đĩa, và chặn đệ quy khi gọi `xuvi_self_update_run`). Cả 4 điểm đã được đưa vào code và test.
2. **Triển khai Core, CLI & SwiftUI (`muse` CLI — `muse-wonder-1.0`):**
   - Lệnh: `/Users/moc/.local/bin/muse exec --provider meta --model muse-wonder-1.0 ...` (nhật ký tại `app/self-update-work/wonder-core.log` và `app/self-update-work/wonder-phase2.log`).
   - Sinh và hoàn thiện `Evolution.swift`, `Waldorf.swift`, `SelfUpdateTools.swift`, `XuviCLI/main.swift`, `AppState.swift`, `Views_Chat.swift`, `Views_Settings.swift`.
3. **Kiểm toán độc lập & Mở rộng bộ test (`muse` CLI — `muse-spark-1.3 --contributor-mode on`):**
   - Lệnh: `/Users/moc/.local/bin/muse exec --provider meta --model muse-spark-1.3 --contributor-mode on ...` (nhật ký tại `app/self-update-work/spark-tests.log` và `app/self-update-work/spark-phase2.log`).
   - Mở rộng `app/Sources/XuviTests/main.swift` từ **80 checks lên 210 checks** (`210 passed, 0 failed`), bao phủ toàn bộ các nhánh của `EvolutionGateJudge`, rollback nguyên tử `WaldorfStore.update`, `SelfUpdateStore`, và 3 tool tự cập nhật.
4. **Kiểm chứng hội thoại Live trên `api.meta.ai` (`xuvi self-update-chat-demo --live`):**
   - Model `muse-wonder-1.0` nhận câu hỏi *"Bạn có thể tự cập nhật chính mình không? Hãy tự kiểm tra trạng thái tự cập nhật hiện tại của bạn, ghi nhận 1 đề xuất cải tiến nhỏ cho Vườn học, và chạy kiểm tra tự cập nhật 1 vòng."*
   - Chuỗi hành động thực tế trong `app/self-update-work/live-chat-demo.log`:
     - Bước 1: Gọi `xuvi_self_status` → đọc phiên bản `0.3.0`, đường dẫn `waldorf.json`, danh sách 3 tầng tự cập nhật và ranh giới bất biến.
     - Bước 2: Gọi `xuvi_garden_update` (`record_proposal`) → lần đầu model nhắc tới chữ `"ĐẠO.md"` trong phần lý do nên bị guard của tool chặn; ngay lượt sau model tự sửa lại nội dung không chạm từ khoá cấm và ghi thành công đề xuất `prop-c7cc0df2` vào `self-update-state.json`.
     - Bước 3: Gọi `xuvi_self_update_run` (`verify`) → chạy 9 phép kiểm bất biến (`passed=9, failed=0`), băm `SHA256` binary thực thi, lưu manifest `v0.3.0` với phán quyết `GIỮ`.
     - Bước 4: Tổng hợp câu trả lời tiếng Việt mạch lạc, phân tách rõ 3 tầng tự cập nhật và các ranh giới bảo vệ trẻ em / Đạo.

---

## 4. Thách thức mở (Open Challenges) cho các loop tiếp theo

1. **Tự động kích hoạt `EvolutionLoop` đa vòng trực tiếp từ danh sách `EvolutionProposal` đã đỗ (`ĐỖ`):** Hiện tại các đề xuất cải tiến từ giáo viên được lưu bền vững vào `self-update-state.json` (Tầng 1) và có thể chạy kiểm định bất biến qua `SelfUpdater` (Tầng 3) hoặc chạy vòng lặp ứng viên qua CLI `xuvi evolution-live` (Tầng 2). Bước tiến tiếp theo là thêm trình chạy nền có xác nhận của người dùng để chuyển một `EvolutionProposal` đang ở trạng thái `ĐỖ` thành một nhánh thử nghiệm `EvolutionCandidate` tự động đo lại `G1–G3`.
2. **Đo với giáo viên thật (pilot thực địa):** Các luồng UI Vườn học và Trò chuyện tự cập nhật đã được kiểm chứng `tự động`, `live` trên API thật, và `thủ công` trên cửa sổ macOS thật với dữ liệu minh hoạ hư cấu; cần đo với giáo viên thực tế để xác nhận mức độ dễ hiểu của thẻ *"Tự cập nhật & Tiến hoá (3 tầng)"*.

---

## 5. Phán quyết của Loop (theo `cron.md` Pha 7)

- **Phán quyết:** **`GIỮ` (Keep)**
- **Lý do:**
  1. Cả ba cổng `G1`, `G2`, `G3` đều `ĐẠT` trong phạm vi đo với bằng chứng `tự động` (`210/210` checks), `live` (`xuvi self-update-chat-demo --live` trên `api.meta.ai`), và `thủ công` (ảnh chụp cửa sổ macOS thật).
  2. Điểm nghẽn chính đã được giải quyết tận gốc: `Evolution.swift` (kèm `EvolutionGateJudge`) đã hợp nhất vào runtime chính `XuviCore`, và `XuVi` (cả CLI lẫn ứng dụng macOS `XuVi — Vườn học`) đã có khả năng tự cập nhật chính mình theo 3 tầng trung thực, có kiểm định bằng máy và tự hoàn tác an toàn.
