# 04 — Session log: nhật ký sự kiện là nguồn sự thật

> Một session là một **danh sách sự kiện chỉ-nối-thêm (append-only)**. Lịch sử gửi cho mô hình được *suy ra* từ danh sách này mỗi lần cần, không lưu riêng ở đâu khác. Đây là quyết định kiến trúc quan trọng nhất của harness — nó cho bạn resume, fork, replay và kiểm thử gần như miễn phí.

Đọc trước: [01](01-tong-quan-kien-truc.md). Liên quan: vòng lặp ghi log ở [03](03-agent-loop.md), lưu bền ở [11](11-luu-tru-va-khoi-phuc.md).

---

## 1. Một sự kiện trông thế nào

```
SessionEvent = {
  seq:  số nguyên, vị trí trong log (bắt đầu từ 0, liên tục không nhảy)
  time: thời điểm (epoch ms)
  type: nhãn phân biệt, ví dụ 'user/message'
  data: payload theo type, bắt buộc là JSON không mất dữ liệu (lossless JSON)
  ignorable?: true      // chỉ khi sự kiện thuần thông tin, mất cũng không sao
  surfaceOp / sourceEventSeqs   // chỉ với sự kiện sinh tin nhắn — xem mục 4
}
```

- `type` là nhãn phân biệt duy nhất quyết định kiểu của `data` (discriminated union): đọc code phải `switch` theo `type`, không nối chuỗi `if`.
- Payload không JSON hoá được (hàm, chu trình, BigInt, `undefined` lọt vào…) → **từ chối ngay tại điểm ghi**, không đợi tới lúc lưu trữ mới nổ.
- Sự kiện mới lạ mà reader không biết: mặc định **từ chối dựng lại session**, trừ khi sự kiện tự khai `ignorable: true`. Thà từ chối nhầm còn hơn resume một session đã bị cắt mất phần quan trọng mà không ai biết.

---

## 2. Từ vựng sự kiện cốt lõi

Đây là bộ tối thiểu. Plugin được *thêm* loại sự kiện của riêng mình (mục 7) nhưng không sửa bộ này.

| Type | Vào lịch sử model? | Nội dung |
|---|---|---|
| `turn/start` `{turn}` | Không | Mở turn |
| `turn/end` `{turn, reason}` | Không | Đóng turn kèm lý do (file 03, mục 3.2) |
| `step/start` `{turn, step}` | Không | Mở step |
| `step/end` `{turn, step}` | Không | Đóng step |
| `user/message` | **Có** (role user) | Tin nhắn vào: từ người dùng, từ inject, từ tiếp tục mục tiêu… `source` phân biệt nguồn; nội dung vào lịch sử nguyên văn |
| `system/message` | **Có** (role system) | System prompt đã render — xem mục 5 |
| `assistant/message` | **Có** (role assistant) | Câu trả lời hoàn chỉnh của một step + luồng stream gốc đã nén + usage + cờ interrupted nếu bị cắt |
| `assistant/attempt` | Không | Một lần gọi model thất bại/huỷ/đã retry — giữ stream làm bằng chứng, không bịa lịch sử |
| `tool/call` `{callId, name, arguments}` | Không* | Model yêu cầu gọi tool; `arguments` là **chuỗi JSON thô nguyên bản** model sản xuất |
| `tool/result` | **Có** (role tool) | Kết quả cho model + thông tin lỗi cho người dùng + `meta` riêng của tool để vẽ lại UI |
| `request/header` | Không | Ảnh chụp "đầu yêu cầu" đang hiệu lực — xem mục 5 |
| `request/context` | Không | Metadata tuyến (route) đã phân giải: provider/model, cửa sổ ngữ cảnh… chỉ ghi khi thay đổi |
| `session/end-seed` | Không | Mốc tách phần lịch sử kế thừa (fork) khỏi phần việc sống — xem mục 6 |

\* `tool/call` không tự thành tin nhắn; nó được lịch sử mang theo trong khối tool-call của `assistant/message`. `tool/result` thì là tin nhắn hạng nhất vì mọi nhà cung cấp đều yêu cầu kết quả đi kèm call tương ứng.

---

## 3. Suy ra lịch sử: `deriveMessages()`

Hàm thuần: `log → Message[]` cho mô hình.

- `user/message` → tin user, nội dung nguyên văn, `source` chỉ là metadata hiển thị/log, không đổi nội dung.
- `assistant/message` → tin assistant kèm provider/model đã sản xuất và dữ liệu replay riêng của adapter (nếu có). Sự kiện có nội dung rỗng (step bị cắt bởi trần token trước khi có chữ nào) thì **bỏ qua** khi dựng lịch sử — vẫn giữ trong log làm bằng chứng usage/stream.
- `tool/result` → tin role tool mang `toolCallId` tương ứng.
- Mọi loại khác → không sinh tin nhắn.

Tính chất bắt buộc:

1. **Thuần và xác định**: cùng một log → cùng một lịch sử, mãi mãi. Muốn một plugin thay đổi nội dung tin nhắn đã có thì phải qua cơ chế *projection* thuần đã đăng ký (mục 7), không được sửa sự kiện cũ.
2. **Đóng băng**: kết quả trả ra là dữ liệu bất biến; consumer không sửa được log thông qua lịch sử đã suy ra.
3. **Tăng dần được**: lần suy ra sau chỉ tốn công trên sự kiện mới, trừ khi có thay thế bề mặt (mục 4) buộc dựng lại.

> **Kiểm chứng nhanh của quy tắc 3.2 (file 01):** yêu cầu gửi model ở step bất kỳ = `deriveMessages(log tới thời điểm đó)` + header hiệu lực. Nếu không tái dựng được, có dữ liệu ngoài log đã lọt vào yêu cầu.

---

## 4. Bề mặt (surface) và thay thế

Không phải sự kiện sinh tin nhắn nào cũng còn nằm trong lịch sử hiện tại: nén ngữ cảnh (compaction, file 10) thay một đoạn dài bằng một bản tóm tắt. Cơ chế:

- Mỗi sự kiện sinh tin nhắn khai báo `surfaceOp`: `append` (nối vào cuối — đường thường) hoặc `{op: 'replace', startSeq, endSeq}` (thay thế đoạn bề mặt từ sự kiện này tới sự kiện kia bằng chính sự kiện này).
- Sự kiện thay thế phải trích dẫn (`sourceEventSeqs`) đầy đủ các sự kiện bề mặt nó che đi.
- Lịch sử hiện tại =fold các `surfaceOp` theo thứ tự log. UI dạng transcript (cho người đọc) thì đọc các sự kiện gốc theo thứ tự ghi, không đọc bề mặt đã thay — vì người dùng cần thấy những gì đã bị tóm tắt.

---

## 5. System prompt và header: hai thứ nằm ngoài "nội dung"

**System prompt là một tin nhắn trong lịch sử**, không phải trường riêng của yêu cầu: nó được ghi thành `system/message` ở nút đầu tiên của bề mặt. Khi prompt thay đổi:

- nếu tuyến model hỗ trợ "prompt cập nhật trong lịch sử" → nối một `system/message` mới phía sau (mô hình đọc bản mới nhất);
- nếu không → thay thế nút system đầu tiên qua đường ghi có kiểm soát;
- prompt rỗng → xoá hiệu lực mọi nút system (không để chỉ thị cũ sống sót).

**`request/header`** là ảnh chụp phần *không phải nội dung* của yêu cầu: cấu hình gọi (provider, model, effort, thông số sinh) + danh sách tool schema đã ghép. Ghi khi: bắt đầu session/resume, thay đổi, hoặc bắt đầu series mới. Header mới nhất hiệu lực cho mọi yêu cầu sau đó, kể cả retry.

Tách hai thứ này ra khỏi nhau là có chủ đích: đổi prompt không làm header đổi (không phá cache tiền tố của nhà cung cấp), và mọi yêu cầu đều tái dựng được từ `log = header + lịch sử`.

---

## 6. Fork, seed và mốc kế thừa

- **Fork** = tạo session mới mà tiền tố là bản sao sâu của log nguồn tới một ranh giới ổn định (mặc định: sự kiện cuối hiện tại; không được cắt giữa một turn đang mở — từ chối thay vì cắt lén, trừ cơ chế fork của subagent có quy tắc riêng ở file 10).
- `session/end-seed {inherited: true}` được ghi đúng tại điểm cắt. Mốc cuối cùng mang cờ này cho biết session con kế thừa bao nhiêu sự kiện.
- Sự kiện seed không phát thông báo sống. Fork giữ nguyên thư mục làm việc (cwd) của nguồn nhưng không mang quan hệ "session cha" bền vững — lineage là dữ liệu riêng (file 10).

---

## 7. Plugin mở rộng log

Plugin thêm loại sự kiện bằng cách *khai báo mở rộng* bản đồ sự kiện (declaration merging hoặc đăng ký tương đương tuỳ ngôn ngữ). Hai loại:

1. **Log-only**: chỉ ghi nhận (ví dụ `compaction/start`, `job/registered`, `approval/asked`…). Không vào bề mặt, không sinh tin nhắn.
2. **Sửa nội dung tin nhắn đã có**: đăng ký một *message projection* thuần `(sự kiện, lịch sử trước đó) → tin nhắn đã sửa`. Session kiểm chứng và áp dụng trước khi chốt sự kiện. Thiếu projection đã đăng ký khi đọc lại → từ chối dựng session. Projection phải thuần: không I/O, không sửa đầu vào.

Một nhóm sự kiện cùng vẽ thành một nút giao diện (ví dụ một "thẻ" trong chat) phải mang cùng một **id nghiệp vụ ổn định** do producer tự suy ra từ payload của chính nó — client không bao giờ đoán "gắn vào cái gần nhất chưa xong".

---

## 8. Bất biến của log

1. `seq` liên tục, không tái sử dụng, không sửa sự kiện đã ghi. Sai thì sửa bằng sự kiện mới.
2. Mọi `tool/call` cuối cùng đều có `tool/result` tương ứng (kể cả kết quả lỗi tổng hợp khi step chết giữa chừng).
3. Turn mở phải được đóng — bởi vòng lặp, hoặc bởi khôi phục sau crash (file 11). Không để turn mở treo trong một session đã resume xong.
4. Sự kiện nằm ngoài turn vẫn hợp lệ (plugin ghi log-only giữa hai turn); đừng bịa turn tổng hợp chỉ để "bao" chúng.

---

## 9. Tiêu chí nghiệm thu

- [ ] Từ một log đã lưu, `deriveMessages()` cho ra đúng từng yêu cầu đã gửi (so với bản ghi lúc chạy) ở mọi step.
- [ ] Ghi payload không JSON hoá được → lỗi ngay tại điểm ghi, log không đổi.
- [ ] Reader gặp loại sự kiện lạ không có `ignorable` → từ chối session kèm lý do rõ; có `ignorable` → bỏ qua an toàn.
- [ ] Fork tại một `turn/end` → session con replay ra lịch sử giống hệt nguồn tới điểm cắt, và tiếp tục độc lập.
- [ ] Thay thế bề mặt (tóm tắt thử) → lịch sử rút gọn đúng; transcript gốc vẫn đọc được đầy đủ.
