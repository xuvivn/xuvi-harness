# 05 — Model gateway: cổng gọi mô hình trung lập

> Vòng lặp chỉ nói chuyện với **một** giao diện gọi mô hình. Mỗi nhà cung cấp (hoặc model chạy local) là một **adapter** đăng ký vào gateway. File này định nghĩa hợp đồng đó — *không* liệt kê provider/model cụ thể. Danh sách provider, model, xác thực, endpoint nằm ở [technology/ai-models.md](../technology/ai-models.md) và [technology/ai-model-runtime.md](../technology/ai-model-runtime.md).

---

## 1. Từ vựng dùng chung

### 1.1. Tin nhắn và khối nội dung

Một hội thoại là danh sách `Message`; một tin nhắn là danh sách **khối nội dung** có kiểu:

| Khối | Ý nghĩa |
|---|---|
| `text` | Văn bản thường |
| `reasoning` | Suy luận của mô hình (tách khỏi văn bản hiển thị) |
| `image` | Ảnh (tham chiếu tới attachment bền vững, không nhét bytes vào log) |
| `file` | File đính kèm nguyên văn (qua attachment) |
| `tool-call` | `{id, name, arguments}` — `arguments` luôn là **chuỗi JSON thô** |
| `tool-addition` / `tool-removal` | Kích hoạt/gỡ một tool trong lịch sử (cho tuyến hỗ trợ nạp tool trễ) |

Tin nhắn có `role` (`system`/`user`/`assistant`/`tool`/`developer`…) và `source` cho biết ai sản xuất (`user`, `model` kèm provider+model, `tool`, `system-prompt`, và các nguồn do plugin tự khai báo). Nguồn còn mang `form` ngữ nghĩa tuỳ chọn — nội dung này là *chỉ thị*, *danh mục*, *ảnh chụp trạng thái*, *thông báo một lần*, *chuyển tiếp từ agent khác*, hay *trích từ session khác* — để nơi hiển thị quyết định cách trình bày. Hình thức trình bày (màu, icon, thu gọn…) là việc của nơi hiển thị, không được lọt vào từ vựng này.

Thêm một loại khối mới chỉ khi adapter, UI, compaction và đường replay bền vững đều hỗ trợ nó.

### 1.2. Giao thức streaming: `StreamChunk`

Adapter trả về một luồng chunk có kiểu, **đóng** (không mở rộng tuỳ tiện — thêm biến thể là thay đổi phá vỡ mọi consumer):

```
block-start   {index, blockType}
text-delta    {index, text}
reasoning-delta {index, text}
tool-call-delta {index, id, name?, argumentsDelta}
block-end     {index, block}        // khối đã ghép hoàn chỉnh
usage         {usage}
finish        {reason, replayState?}
```

- `index` gắn mỗi delta vào khối của nó — các khối xen kẽ nhau (text, reasoning, nhiều tool call) vẫn tách bạch.
- Consumer dùng một **bộ ghép dùng chung** (assembler) để gấp luồng chunk thành khối hoàn chỉnh + usage + lý do kết thúc + trạng thái replay. Không consumer nào tự ghép lại theo cách riêng.
- Luồng đã ghép kèm dấu thời gian được nén và nhúng vào `assistant/message`/`assistant/attempt` trong log (file 04) mà không làm mất ranh giới delta.

### 1.3. Lý do kết thúc và lỗi

`FinishReason`: `stop` · `tool-calls` · `max-tokens` · `aborted {failure}` · `error {failure}` — mở rộng được cho lý do riêng của provider.

Mọi thất bại chuẩn hoá về **một** kiểu lỗi có thể serialize: `{message, code, status?, providerRetryAfterMs?, requestId?}`. Consumer rẽ nhánh theo `code` máy-đọc-được, **không bao giờ** theo câu chữ của provider. Một số code chuẩn: tràn cửa sổ ngữ cảnh, phản hồi rỗng, timeout, bị huỷ.

`TokenUsage`: các đếm **rời nhau** — input chưa cache, cache đọc, cache ghi báo riêng (input tính tiền = tổng ba phần); reasoning tokens là chi tiết nằm *trong* output, không cộng thêm lần nữa.

---

## 2. Một yêu cầu: `GenerateOptions`

Một lần gọi mô hình là một yêu cầu đã ghép hoàn chỉnh:

```
{
  provider, model, reasoningEffort?,
  messages: [...],        // lịch sử đã suy ra từ log (đường vòng lặp)
  system?,                // chỉ cho caller một-lần-gọi ngoài vòng lặp
  tools?: [ToolSchema],   // {name, description, parameters: JSON Schema}
  temperature?, maxTokens?, stop?,
  signal?,                // huỷ hợp tác
  sessionId?, purpose?    // metadata định tuyến, không hiện với model
}
```

`ToolSchema` thuộc về file này (không thuộc tool registry) vì nó là một phần của yêu cầu mà vòng lặp ghép ở mỗi step. Registry sinh ra nó; gateway chỉ mang nó đi.

---

## 3. Hợp đồng adapter — mọi adapter phải tuân, mọi consumer được quyền dựa vào

1. **`usage` trước `finish`, không gì sau `finish`.**
2. **Arguments của tool call giữ nguyên chuỗi JSON thô từ đầu tới cuối.** Provider nào trả object đã parse thì adapter phải chuỗi hoá lại tại `block-end`.
3. **Hai đường lỗi, một kiểu lỗi.** Adapter hoặc *ném* (lỗi vận chuyển/giao thức), hoặc kết thúc luồng bằng `finish {error|aborted, failure}`. Gateway chuẩn hoá cả hai về cùng một dạng cho consumer.
4. **Một lần gọi adapter = một lần thử ở provider.** Adapter tắt retry của thư viện bên dưới. Việc thử lại do tầng trên quyết định (vòng lặp qua `agent/request-error`, hoặc một module retry chuyên trách) — để mỗi lần thử đều được ghi log và đếm được.
5. **Stall có giới hạn.** Luồng treo không chunk mới quá một timeout cấu hình được → lỗi timeout; huỷ của người gọi luôn thắng và giữ nguyên ý nghĩa "đã huỷ".
6. **Tràn ngữ cảnh có một code chuẩn duy nhất**, dù provider báo bằng HTTP lỗi hay bằng lỗi trong luồng.
7. **Hoàn thành rỗng là lỗi retryable**, không phải thành công im lặng.
8. **Trạng thái replay là của adapter.** Một `finish` thành công có thể mang metadata riêng để phát lại đúng định dạng provider. Gateway chỉ đưa lại trạng thái đó cho *đúng adapter instance đã sản xuất nó* và khi provider đích trùng provider gốc; adapter khác nhận nội dung trung lập. Trạng thái không dùng được → hạ cấp tin nhắn đó xuống dạng trung lập kèm chẩn đoán, không làm hỏng cả yêu cầu.
9. **Định danh ứng dụng tĩnh.** Mọi yêu cầu HTTP tới provider mang định danh ứng dụng công khai (tên sản phẩm/phiên bản) — không chứa bí mật, đường dẫn, id session hay định danh người dùng.

---

## 4. Registry và phân giải model

- `registerAdapter(adapter, providers[]) → handle` — handle gồm disposer + phép thay toàn bộ tuyến (routes) nguyên tử: kiểm chứng toàn bộ tập mới trước, hoán đổi trong một nhịp đồng bộ, không yêu cầu nào thấy trạng thái nửa vời.
- **Danh mục model (catalog) chỉ là tư vấn**: hiển thị cho người dùng chọn. Định tuyến thật vẫn dựa trên provider đã đăng ký.
- **Metadata đúng-tuyến** (adapter sở hữu, phân giải theo đúng cặp provider+model): cửa sổ ngữ cảnh, trần output mặc định, danh sách mức reasoning effort (theo thứ tự tăng dần, kèm mặc định), chế độ cập nhật system prompt, chế độ cập nhật tool. Consumer không tự đoán các giá trị này.
- Reasoning effort là định danh do adapter sở hữu — core chỉ mang nó đi như một nhãn mờ, không liệt kê giá trị.

---

## 5. Chính sách retry (tầng trên adapter)

- Chính sách phân giải **trước** khi đăng ký tuyến thành một giá trị bất biến: chế độ thường `{maxRetries, retryableCodes, initialDelayMs, maxDelayMs, jitterRatio}` hoặc chế độ luôn-thử-lại (không trần số lần, vẫn giữ backoff). Thiếu chính sách → mặc định hữu hạn (tham chiếu: 5 lần).
- Chính sách được *chụp vào lời gọi* lúc adapter được chọn: gỡ/thay tuyến giữa chừng không đổi được chính sách của một thất bại đang bay.
- Delay do provider yêu cầu (`Retry-After`) được tôn trọng như dữ kiện, quyết định có retry hay không vẫn là của chính sách.

---

## 6. Gateway không làm gì

- Không chứa logic hội thoại, không ghi log (vòng lặp ghi).
- Không chọn model thay người dùng (nó chỉ phân giải lựa chọn đã có).
- Không biết tool nào đang được bật — nó chỉ mang `tools` được đưa cho.

---

## 7. Tiêu chí nghiệm thu

- [ ] Đăng ký một adapter giả (phát lại luồng chunk đã ghi): vòng lặp chạy trọn một turn mà không phân biệt được với adapter thật.
- [ ] Adapter ném lỗi giữa stream và adapter kết thúc bằng `finish{error}` cho ra cùng một dạng lỗi ở consumer.
- [ ] Tool call được stream thành nhiều delta rời rạc vẫn ghép thành đúng một khối với `arguments` nguyên văn.
- [ ] Thay tuyến của một adapter đang có yêu cầu bay: yêu cầu đó kết thúc bằng chính sách/adapter đã chụp, không lỗi lạ.
- [ ] Một adapter thứ hai (khác nhà cung cấp) được chọn chỉ bằng đổi `{provider, model}` trong lựa chọn của session — không sửa code vòng lặp.
