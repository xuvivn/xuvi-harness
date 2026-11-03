# 03 — Agent loop: turn, step, inbox

> Đây là trái tim của harness: vòng lặp biến đầu vào của người dùng thành các lần gọi mô hình và công cụ. File này định nghĩa hành vi, không định nghĩa công nghệ. Bất kỳ cài đặt nào tuân theo file này đều thay thế lẫn nhau được.

Đọc trước: [01](01-tong-quan-kien-truc.md), [02](02-module-va-plugin.md), [04](04-session-log.md).

---

## 1. Các khái niệm

| Khái niệm | Định nghĩa |
|---|---|
| **Agent** | Thực thể sống điều khiển một session: có `id` (= id của session), `session`, `status` (`idle`/`running`), hộp thư đến, lựa chọn model, và thao tác `send/followup/steer/inject/cancel/whenIdle`. |
| **Turn** | Một lần xả đầu vào đã nhận: mở trước khi nhận đầu vào đầu tiên, đóng khi không còn nợ gì (mô hình dừng, không tool nào chờ, không đầu vào lái nào chờ). |
| **Step** | Một lần gọi mô hình + các tool mà câu trả lời đó yêu cầu. Một turn có 0..nhiều step. Turn có thể đóng với **0 step** (đầu vào bị từ chối/trống). |
| **Inbox** | Hai hàng đợi bền vững của agent: `next-turn` (chờ mở turn riêng) và `next-step` (chờ biên step tiếp theo trong turn hiện tại). |
| **Request series** | Một chuỗi các yêu cầu liên tiếp dùng chung "đầu yêu cầu" (header). Một số thay đổi (prompt đổi theo kiểu không tương thích, thay thế bề mặt…) bắt đầu series mới. Xem file 04. |

`status = running` nghĩa là bộ máy đang xả việc (có thể spanning nhiều turn liên tiếp), không có nghĩa một turn cụ thể còn mở. Huỷ agent xong, agent biến khỏi registry — "đã huỷ" không phải một trạng thái quan sát được thứ ba.

---

## 2. Cách đưa đầu vào vào agent

Mọi đầu vào đều là một **tin nhắn người dùng có danh tính** (`id` duy nhất + `source` cho biết ai sản xuất). Có ba cách gửi, khác nhau ở *đích đến* và *có đánh thức hay không*:

| Cách gửi | Đích | Đánh thức agent đang idle? | Dùng khi |
|---|---|---|---|
| `followup(msg)` | `next-turn` | Có | Câu hỏi/yêu cầu mới của người dùng; mỗi followup thành một turn riêng |
| `steer(msg)` | `next-step` | Có (idle thì mở turn) | Lái hướng giữa chừng: agent đang chạy sẽ nhận ở biên step gần nhất |
| `inject(msg)` | `next-step` | **Không** | Ngữ cảnh cho mô hình (file thay đổi, thông báo hệ thống…); nằm chờ tới khi có thứ khác đánh thức |

Hậu quả cần nhớ:

- Chỉ inject mà không có followup/steer thì **không có gì chạy cả** — ngữ cảnh chờ là đúng thiết kế, không phải bug.
- Inject có thể "trượt" khỏi một yêu cầu mà giai đoạn trước-step của nó đã nhận lô xong. Chấp nhận điều này; đừng cố đồng bộ hoá tuyệt đối.
- Mỗi tin nhắn chờ được nhận diện bằng chính `id` của nó xuyên suốt vòng đời: đã chèn → đã nhận (claimed) → hoặc bị loại bỏ. Consumer theo dõi một tin nhắn cụ thể dựa trên ba thông báo này.

Thao tác trên inbox (thay thế, xoá, nối…) đều **ghi vào log** dưới dạng một sự kiện biến đổi hộp thư đã chuẩn hoá, để resume dựng lại đúng hộp thư. Id trùng trong hàng chờ bị từ chối.

---

## 3. Vòng lặp chi tiết

```
TRẠNG THÁI IDLE
  │ có tin nhắn đánh thức
  ▼
turn/start  ──────────────────────────────────────────────── ghi log
  nhận lô đầu vào: toàn bộ next-step + (nếu đầu turn) đúng 1 tin next-turn
  chụp lựa chọn model của session cho step này
  ghép system prompt + tool schemas (file 07)
  ► sự kiện sống agent/pre-step (waterfall) trên lô đã nhận:
      - reject  → đóng turn, không step nào (nếu là lô đầu tiên)
      - enter(messages) → đi tiếp với lô này (được sửa/thêm/bớt;
        tin đã nhận mà bị loại khỏi lô cuối thì coi như đã tiêu thụ)
  ┌─ step/start ────────────────────────────────────────── ghi log
  │   ► sự kiện agent/request (waterfall, điều chỉnh thông số gọi —
  │     không được đổi provider/model/effort đã chụp)
  │   prepareCall(): gắn adapter cụ thể cho provider/model đã chọn
  │     — huỷ trong lúc chuẩn bị hoặc ghép prompt: CHƯA ghi gì cả
  │   ghi system/message (nếu prompt thay đổi) + user/message cho lô
  │   ghi request/header + request/context nếu thay đổi
  │   deriveMessages() từ log → đóng băng yêu cầu
  │   stream qua model gateway (file 05):
  │     chunk sống → UI (agent/assistant-stream: start/chunk*/end)
  │     kết thúc → ghi assistant/message (kèm luồng stream gốc đã nén,
  │                usage) hoặc assistant/attempt nếu là lần thử thất bại
  │   nếu có tool call:
  │     ghi tool/call* → chạy pipeline (file 06) → ghi tool/result*
  │ step/end ───────────────────────────────────────────── ghi log
  └─ còn tool nợ / có đầu vào next-step → step tiếp theo (không mở turn mới)
  ► sự kiện sống agent/turn-stopping (serial, không next):
      listener được chèn việc cuối trước khi turn thật sự đóng
turn/end {reason} ──────────────────────────────────────── ghi log
```

### 3.1. Các đảm bảo quan trọng

1. **Chuẩn bị trước, ghi sau.** Adapter được gắn và prompt được hoà giải *trước khi* system/user message được ghi. Huỷ trong giai đoạn chuẩn bị → log không có vết của step chưa bắt đầu.
2. **Yêu cầu bất biến khi đã phát.** Yêu cầu gửi model là bản đóng băng. Retry không lặp lại việc ghép prompt hay pre-step; nó dùng lại lựa chọn model đã chụp.
3. **Một stream chỉ "chốt" một lần.** Toàn bộ stream của một lần thử được ghi thành đúng một `assistant/message` (thành công, có mặt trong lịch sử) hoặc một `assistant/attempt` (thất bại/huỷ/thử lại — giữ làm bằng chứng, **không** vào lịch sử model). Khung `end` cho UI chỉ phát *sau khi* bản ghi bền vững đã chốt. Mất tiến trình trước khi chốt → lần thử đó không để lại gì trong log.
4. **Step thất bại vẫn đóng step.** Nếu step lỗi mà không có kết quả tool, vòng lặp ghi kết quả tool bị thiếu (thất bại) cho các call còn nợ, để lịch sử không bao giờ có tool call không có kết quả — hầu hết nhà cung cấp model từ chối yêu cầu như vậy.
5. **Tạo agent là giao dịch có rollback.** Tạo/resume agent: vào session store → chạy setup của người tạo → công bố → chạy listener `agent/created` (serial, chờ xong mới cho việc chạy). Bất kỳ bước nào lỗi → tháo ngược toàn bộ, session bị gỡ, đầu vào chờ đã khôi phục giữ nguyên.
6. **Retry là quyết định có chủ.** Sau một step gọi model thất bại, sự kiện `agent/request-error` cho listener cơ hội sửa trạng thái rồi trả `{kind: 'retry'}`. Không ai nhận → lỗi thành lý do kết thúc turn. Vòng lặp tự nó không retry mù.

### 3.2. Lý do kết thúc turn (`turn/end`)

Đóng bởi đúng một lý do có kiểu: `completed` · `aborted` (kèm nguyên nhân huỷ) · `blocked` · `error` (kèm lỗi có cấu trúc) · `max-tokens` (bất kỳ step nào chạm trần token đầu ra thì cả turn mang lý do này, kể cả khi sau đó được tiếp tục) · và hai lý do **chỉ do khôi phục/fork tổng hợp, vòng lặp không bao giờ tự phát**: `interrupted` (crash để lại turn mở), `forked` (fork cắt ngang turn đang mở).

---

## 4. Huỷ (cancellation)

- `cancel(cause)` huỷ turn đang chạy và (trừ khi `keepInbox`) dọn hàng chờ. Cause có kiểu: do người dùng / do agent cha / do hook (kèm lý do) / do bị dispose. **Nguyên nhân đầu tiên thắng** cho hoạt động đó.
- Huỷ khi đang idle là no-op và **không** "gài" cho việc tương lai; đầu vào đánh thức gửi sau một lần huỷ thì xếp hàng cho turn kế tiếp bình thường.
- Huỷ giữa stream: phần text/reasoning đã phát được chốt thành `assistant/message` với cờ `interrupted: true`, tool call chưa phát thì bỏ. Turn không có sự kiện nào như vậy nghĩa là chưa có nội dung nào tới được người dùng.
- Tín hiệu huỷ là tín hiệu hợp tác: code cùng tiến trình không bị giết cứng được. Mọi thao tác dài (gọi model, chạy tool, chạy tiến trình con) phải nhận và tôn trọng signal — xem file 06 và file 13.

---

## 5. Lựa chọn model của agent

- Mỗi agent/session có lựa chọn model bền vững: `{provider, model, reasoningEffort?}`. Nguồn ưu tiên cho step kế tiếp: lựa chọn đã ghi trong log → lựa chọn đã dùng ở yêu cầu gần nhất → tuỳ chọn ban đầu của agent.
- Đổi model = ghi một sự kiện lựa chọn vào log (không vào lịch sử model). Step đang chuẩn bị không bị ảnh hưởng.
- Thiếu provider hoặc model khi chuẩn bị step → lỗi fail loud trước khi ghép prompt.

---

## 6. Bảo trì ngoài turn và `whenIdle`

- `whenIdle()`: chờ tới khi toàn bộ hoạt động của agent (kể cả việc bảo trì và việc bị đánh thức phía sau nó) lắng xuống. Nó **không** hứa về một tin nhắn cụ thể nào.
- Công việc bảo trì (không phải turn) chỉ chạy từ pha idle thật, chiếm pha đó một cách đồng bộ; đầu vào đến trong lúc bảo trì nằm chờ trong inbox.

---

## 7. Những sự kiện sống của agent (tóm tắt hợp đồng)

| Sự kiện | Chế độ | Khi nào | Listener được làm gì |
|---|---|---|---|
| `agent/created` | serial | Agent vừa công bố | Khởi tạo theo agent; lỗi → huỷ việc tạo agent |
| `agent/pre-step` | waterfall | Trước mỗi step, trên lô đầu vào đã nhận | `enter` (được sửa lô) / `reject`; khai báo bắt đầu series mới |
| `agent/request` | waterfall | Sau pre-step, trước prepareCall | Chỉnh thông số gọi; không đổi route model |
| `agent/assistant-stream` | emit | Trong lúc stream | Hiển thị trực tiếp (chunk là tạm thời, không bền) |
| `agent/request-error` | waterfall/bail | Step gọi model thất bại | Sửa trạng thái rồi yêu cầu `retry`, hoặc để lỗi kết thúc turn |
| `agent/turn-stopping` | serial | Turn sắp đóng, không còn tiếp nối | Chèn việc cuối (ví dụ tiếp tục theo mục tiêu) |
| `agent/status` | emit | Mỗi lần idle↔running đổi | Đồng bộ UI |
| `agent/error` | emit | Step/turn lỗi | Báo cáo; kể cả lỗi không có chỗ ghi trong log |
| `agent/disposed` | emit | Agent rời registry | Dọn dẹp phía consumer |

Sự kiện về một agent chỉ tới listener scoped của agent đó và listener global (file 02, mục 4).

---

## 8. Không thuộc phạm vi file này

- Định dạng sự kiện log: file 04. Hợp đồng adapter/stream: file 05. Pipeline tool: file 06.
- Ai được tạo agent con và cách uỷ quyền: file 10 (subagent cũng chỉ là consumer của registry này).
- Vòng lặp **không** biết: UI, nhà cung cấp model cụ thể, chính sách approval, cách lưu trữ.

---

## 9. Tiêu chí nghiệm thu

- [ ] Gửi followup → đúng một turn; trong turn, model gọi tool → có step 2; model dừng → `turn/end {completed}`.
- [ ] Inject khi idle → không có turn nào; sau đó followup → ngữ cảnh đã inject xuất hiện trong yêu cầu của step đầu tiên.
- [ ] Huỷ giữa stream → `assistant/message` có cờ interrupted (nếu đã có nội dung) + `turn/end {aborted}`, không có tool call mồ côi không kết quả.
- [ ] Listener pre-step reject lô đầu tiên → turn đóng với 0 step, không có yêu cầu nào gửi đi.
- [ ] Crash giữa turn rồi resume → turn được đóng bổ sung bằng lý do `interrupted`, lịch sử vẫn hợp lệ để gọi model tiếp (file 11).
- [ ] Đổi model giữa hai turn → step sau dùng model mới, step trước không đổi; replay log tái dựng đúng cả hai yêu cầu.
