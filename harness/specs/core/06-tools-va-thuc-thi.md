# 06 — Tools và pipeline thực thi

> Tool là cách mô hình tác động lên thế giới. File này định nghĩa: một tool khai báo những gì, registry quản lý ra sao, và một lần gọi tool đi qua những chốt kiểm soát nào trước khi thực sự chạy. Danh sách tool cụ thể bật trong từng mặt tiếp xúc nằm ở [technology/chat-va-tools-integration.md](../technology/chat-va-tools-integration.md).

Đọc trước: [02](02-module-va-plugin.md), [04](04-session-log.md). Chốt approval: [09](09-tuong-tac-nguoi.md).

---

## 1. Một tool khai báo những gì

```
ToolDefinition = {
  // Phần mô hình thấy (sinh ToolSchema):
  name, description, parameters (JSON Schema)

  // Phần chỉ harness thấy:
  output:  { schema, render(args, value) -> ContentBlock[],
             presentationMeta?(args, value) }
  execute(args, exec) -> canonical JSON value
  projectContent?(exec, result) -> ContentBlock[] | undefined
  finalizeContent?(exec, result) -> ContentBlock[] | undefined
  timeoutMs?
  isConcurrencySafe?(args) -> boolean
  presentCall?(args) -> view | undefined
  presentResult?(args, result) -> view | undefined
}
```

Quy tắc:

1. **Schema gửi model được sinh theo allowlist tường minh** — chỉ `name/description/parameters` (và cờ nạp-trễ nếu có). Các trường thực thi (`output`, `execute`, timeout, presenter…) **không bao giờ** lọt vào yêu cầu model.
2. **`execute` chỉ trả về giá trị JSON chính tắc** đã khai trong `output.schema`. Tầng pipeline kiểm chứng giá trị theo schema rồi mới dùng `render` biến nó thành nội dung cho model. Tool không tự chế nội dung model tùy tiện.
3. **Tham số vào là bản chụp đóng băng** đã validate (tool dùng một DSL khai báo schema có kiểu để khỏi tự validate tay; tool viết tay thì tự validate và chịu trách nhiệm).
4. **`exec` mang**: danh tính lần gọi (agent, callId…), tín hiệu huỷ, và cơ chế hoãn ngữ cảnh bổ sung. Mọi việc bất đồng bộ trong `execute` phải tôn trọng signal và chỉ kết thúc khi việc con đã lắng (quiesce) — pipeline không "giết cứng" được code cùng tiến trình, nên đây là nghĩa vụ của tool.
5. **Presenter là hàm thuần**: `presentCall/presentResult` chỉ phụ thuộc vào args (và kết quả đã bền vững) — vì UI gọi chúng cả lúc stream sống lẫn lúc replay log. Presenter có side-effect là bug.
6. `finalizeContent` là chốt cuối thuần, đồng bộ, **không được ném**: nó chạy cho mọi kết quả đã chuẩn hoá kể cả lỗi pipeline, để tool thực thi bất biến cuối cùng về nội dung (ví dụ cắt nội dung quá dài) mà không đổi được trạng thái lỗi hay metadata.

---

## 2. Thiết kế tool cho agent (trước khi nghĩ tới pipeline)

Tool là hợp đồng giữa một hệ xác định (code của bạn) và một tác nhân không xác định (mô hình). Một tool đúng cơ chế ở mục 1 nhưng khó chọn, khó gọi đúng hoặc trả về quá nhiều vẫn là một tool hỏng. Các quy tắc dưới đây áp cho mọi tool mô hình thấy được, kể cả tool đến từ MCP (file 08, mục 2.9).

### 2.1. Chọn ít tool, đúng việc

- Xây **vài tool đánh trúng các luồng việc có giá trị cao** thay vì bọc mọi endpoint API sẵn có thành tool. Tool nên gom các thao tác con thường đi cùng nhau thành một lần gọi có ý nghĩa theo cách con người chia việc (ví dụ một tool "tìm các dòng log liên quan kèm ngữ cảnh quanh chúng" thay vì "đọc toàn bộ log"; một tool "lên lịch sự kiện" kiểm tra trùng lịch luôn thay vì ba tool liệt kê/tạo rời).
- **Kiểm tra chồng chéo:** nếu một kỹ sư không nói chắc chắn được trong tình huống X phải dùng tool nào, mô hình cũng không. Hai tool không phân định được bằng một câu mô tả tình huống → gộp, tách lại theo ranh giới khác, hoặc bỏ một.
- Bộ tool của một scope là **bộ tối thiểu khả thi** cho việc của scope đó: mỗi tool thêm vào vừa tốn token schema ở mọi step (file 07, mục 6) vừa thêm một điểm quyết định có thể chọn sai. Tool ít dùng thuộc về scope hẹp hơn hoặc nạp theo yêu cầu, không nằm trong bộ mặc định của mọi agent.

### 2.2. Tên, mô tả và tham số là một phần của prompt

- Nhóm tool theo **không gian tên (namespace)** bằng tiền tố dịch vụ/nguồn lực nhất quán (ví dụ `web_search`, `docs_search`) để ranh giới chức năng đọc được ngay từ tên. Chọn một quy ước tiền tố/hậu tố cho toàn bộ tool và giữ nhất quán; đổi quy ước là đổi prompt.
- Mô tả tool viết như hướng dẫn cho người mới vào nhóm: dùng khi nào, **không** dùng khi nào, định dạng truy vấn đặc biệt, thuật ngữ riêng, và quan hệ với các tài nguyên liên quan. Kiến thức ngầm mà người viết tool "tự hiểu" phải được viết ra.
- Tên tham số không mơ hồ: `user_id` thay vì `user`; quy tắc về mặc định, khoảng giá trị và tham số đi cặp thuộc về mô tả của chính tham số đó hoặc schema của nó — không nằm rải trong mô tả tool khác hay trong system prompt.
- Mỗi dữ kiện chỉ nói **một lần**: quy tắc của một tool nằm trong định nghĩa của tool đó, không lặp lại trong system prompt (file 07) và không lặp trong mô tả của tool kề bên.

### 2.3. Kết quả trả về: tín hiệu cao, token thấp

- Chỉ trả dữ kiện giúp mô hình quyết định bước tiếp theo. Ưu tiên tên/ngữ nghĩa con người đọc được (`name`, `file_type`, định danh ngắn ổn định) hơn định danh kỹ thuật thô (UUID dài, URL nội bộ, mime type) — trừ định danh cần cho lần gọi tiếp theo, thứ đó phải có mặt và nói rõ dùng vào đâu.
- Tool có thể trả cả bản dài lẫn bản ngắn thì khai một tham số chọn mức chi tiết (kiểu `response_format: concise | detailed`) do mô hình điều khiển, mặc định là bản ngắn.
- Mọi kết quả có thể phình đều phải có **phân trang / lọc / chọn khoảng / cắt** với mặc định tiết kiệm. Bản đã cắt phải tự nêu đã cắt ở đâu và chỉ cách lấy tiếp hoặc thu hẹp truy vấn — một thông báo cắt câm lặng khiến mô hình hoặc bỏ cuộc hoặc gọi lại rộng hơn.
- Lỗi tham số phải **hành động được**: nêu trường nào sai, vì sao, và dạng đúng trông thế nào (một ví dụ ngắn). Mã lỗi mờ hoặc nguyên văn stack trace là kết quả hỏng, vì mô hình chỉ có kết quả đó để tự sửa.
- Định dạng kết quả (JSON, XML, Markdown…) chọn theo việc và kiểm chứng bằng đánh giá (mục 2.4); không có định dạng đúng cho mọi tool.

### 2.4. Đánh giá tool bằng số đo

Mô tả và tham số của tool được tinh chỉnh như prompt: bằng đánh giá trên việc thật (file 14, mục 2), không bằng cảm tính. Theo dõi ít nhất: tỉ lệ chọn sai tool, số lần gọi thừa cho cùng một kết quả, tỉ lệ lỗi tham số, token kết quả trung bình, và thời gian hoàn thành việc. Nhiều lần gọi thừa thường là dấu hiệu cần gom thao tác hoặc sửa mặc định phân trang; nhiều lỗi tham số cùng một kiểu là dấu hiệu mô tả/schema chưa rõ — sửa ở định nghĩa tool trước khi thêm chỉ thị vào system prompt.

---

## 3. Registry

- `register(tool) → disposer`. Trùng tên trong cùng scope → lỗi ngay.
- Tool có hai tầng: **global** và **scoped theo agent** (file 02, mục 4), gồm cả cơ chế che (scoped che global cùng tên) và hạn chế (restriction lọc bỏ tool global khỏi một scope — biến mất khỏi cả prompt lẫn thực thi).
- `schemas(scope)` → danh sách `ToolSchema` cho lần ghép prompt này.
- Việc một tool *xuất hiện trong prompt* và việc nó *được phép chạy* là cùng một quyết định: tool không có trong tập đã ghép của scope thì lời gọi tới nó bị từ chối như tool không tồn tại.

---

## 4. Pipeline thực thi một batch

Mô hình có thể yêu cầu nhiều tool trong một step. Executor được chọn (native trực tiếp, hoặc qua một runtime thực thi code — file 08/10) nhận **batch thô**, tự phân tích, lên lịch và ghi log:

```
assistant/message chứa tool-call
  → ghi tool/call (trước khi chạy)
  → hiện thẻ "đang chờ" trên UI (presentCall)
  → tools/pre-execute (waterfall): hooks, chính sách quyền, sandbox…
       ├─ deny ─────────────────────────────┐
       ├─ ask → approval (file 09)           │
       │        ├─ allowed-once → tiếp tục  │
       │        └─ rejected/cancelled/unavailable → deny (fail-closed)
       └─ allow
  → guards đơn điệu (monotonic guards): chỉ deny hoặc đứng ngoài,
    không biến đổi được danh tính lần gọi
  → tools/execute (waterfall bọc ngoài): timeout, retry, metrics
       → thân tool execute()
            (tool ghi file phải qua sự kiện ý định ghi của seam fs — file 08)
  → projectContent của tool (nội dung đã chuẩn bị)
  → tools/post-execute (waterfall): chấp nhận / chặn / thay / thêm ngữ cảnh
  → finalizeContent (chốt cuối của tool)
  → tools/result (thông báo đồng bộ, quan sát kết quả đã đóng băng)
  → ghi tool/result (đúng một kết quả cho model cho mỗi call)
  → hiện thẻ hoàn thành (presentResult)
batch lắng xuống → ngữ cảnh bổ sung (additionalContexts) được inject
  thành user/message ngay sau các tool/result đã ghi
```

Bất biến của pipeline:

1. **Mọi đường đi đều kết thúc bằng đúng một `tool/result`.** Deny, approval từ chối, guard ném lỗi, thân tool ném lỗi, pipeline ném lỗi — tất cả chuẩn hoá thành kết quả `isError` cho model thấy. Không có tool call nào bị treo không kết quả (xem thêm bất biến log ở file 04).
2. **Thứ tự chốt**: approval chạy *trước* guards đơn điệu; chính sách của chủ sở hữu không được phép bị sắp xếp lại thì phải đăng ký dạng guard, không phải waterfall thường.
3. **Nội dung thêm từ hook không phá tính kề nhau**: ngữ cảnh bổ sung của cả batch được phát *sau khi* mọi `tool/result` của batch đã ghi, để cặp call/result luôn kề nhau trong lịch sử.
4. **Kết quả quá lớn phải bị chặn ở nơi biết toàn bộ kết quả** (kể cả phần bọc và metadata), không chặn theo từng mảnh. Vượt giới hạn thì tràn ra bộ lưu trữ phụ và thay bằng một tham chiếu + phần đầu (cơ chế spill, file 11) — quyết định này ghi vào log để replay giống hệt.

### 4.1. Song song và timeout

- Nhiều call trong một batch chỉ chạy song song khi **từng call** tự khai `isConcurrencySafe(args) === true`. Mặc định là độc quyền. Tool chạy song song không được sửa trạng thái của cha; trạng thái dùng chung phải chịu được chạy chồng hoặc fail-closed khi đua.
- `timeoutMs` là ngân sách hợp tác do một wrapper trong `tools/execute` thực thi; khai báo timeout là cam kết tool chuyển tiếp signal tới cài đặt có thể lắng xuống khi bị huỷ.

---

## 5. Executor thay thế được

Vòng lặp không gọi tool trực tiếp; nó chọn một **tool executor**:

| Executor | Ý nghĩa |
|---|---|
| Native | Chạy batch trực tiếp qua pipeline trên |
| Qua runtime thực thi code (PTC-style) | Model viết mã gọi tool như hàm; các lời gọi con vẫn đi qua đúng pipeline này, mang token của call cha, và kết quả của chúng giữ tính kề call/result |

Đổi executor không đổi tool, không đổi pipeline, không đổi log.

---

## 6. Lỗi và trình bày

- Kết quả cho model (`content`, `isError`) tách khỏi lý do cho người dùng (`error {name, code, reason}` ngoài nội dung model) và khỏi `meta` (payload riêng của tool để vẽ lại thẻ UI — bắt buộc JSON hoá được để replay vẽ lại y hệt).
- Tool thất bại vì lý do hạ tầng (không tìm thấy provider, mất quyền…) vẫn là một `tool/result {isError}` — model được thấy và tự quyết định tiếp tục hay dừng.

---

## 7. Tiêu chí nghiệm thu

- [ ] Một tool bị chính sách deny: thân tool không chạy, model nhận đúng một kết quả lỗi, UI hiển thị thẻ bị từ chối.
- [ ] Approval không có ai trả lời (không UI) → kết quả là deny, không treo, không mặc định cho phép.
- [ ] Hai tool call trong một batch, một khai concurrency-safe một không → chỉ phần an toàn được chồng lên nhau; thứ tự ghi `tool/result` vẫn xác định.
- [ ] Tool trả kết quả vượt giới hạn kích thước → log chứa bản đã tràn-có-tham-chiếu; replay cho model thấy đúng bản đó.
- [ ] Gỡ plugin của một tool giữa phiên → tool biến mất khỏi `schemas()` của step sau và lời gọi tới nó bị từ chối như không tồn tại.
- [ ] Một tool có kết quả dài: gọi với tham số mặc định → nhận bản ngắn có chỉ cách lấy tiếp/thu hẹp; gọi bản chi tiết → nhận đủ trường, kể cả định danh cần cho lần gọi tiếp theo.
- [ ] Gọi tool với một tham số sai dạng điển hình → kết quả lỗi nêu đúng trường sai và dạng đúng; mô hình sửa được trong lần gọi kế tiếp mà không cần chỉ thị thêm từ người dùng.
