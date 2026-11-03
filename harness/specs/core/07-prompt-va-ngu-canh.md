# 07 — System prompt và ngữ cảnh đưa vào mô hình (kỹ thuật ngữ cảnh)

> Mỗi lần gọi mô hình cần ba thứ: system prompt, danh sách tool, và lịch sử. File này quản hai thứ đầu cùng với "ngữ cảnh chạy" (runtime context) — dữ kiện mô hình cần biết về thế giới hiện tại — và kỷ luật chọn *cái gì được vào* mỗi lần gọi. Tất cả đều phải qua log (quy tắc 3.2, file 01) và vừa trong ngân sách (quy tắc 3.8, file 01).

Đọc trước: [01](01-tong-quan-kien-truc.md), [04](04-session-log.md). Liên quan: tool schemas ở [06](06-tools-va-thuc-thi.md), metadata tuyến ở [05](05-model-gateway.md), xử lý việc dài hơn cửa sổ ở [10](10-phoi-hop-va-mo-rong.md).

---

## 1. Nguyên tắc: tập token nhỏ nhất có tín hiệu cao nhất

Ngữ cảnh là tài nguyên hữu hạn có lợi tức giảm dần. Mô hình phải chia "ngân sách chú ý" cho mọi token trong cửa sổ; càng nhiều token, các quan hệ giữa chúng càng bị kéo mỏng và khả năng nhớ đúng một dữ kiện cụ thể càng giảm — hiện tượng thường gọi là *context rot*. Suy giảm này là một dốc, không phải vách cứng: mô hình vẫn làm được việc ở ngữ cảnh dài, nhưng độ chính xác truy hồi và suy luận đường dài kém hơn ở ngữ cảnh ngắn. Hệ quả thực hành:

1. **Mục tiêu của mỗi lần ghép là tập token nhỏ nhất vẫn đủ tín hiệu để đạt kết quả mong muốn.** Thêm dữ kiện "cho chắc" là một chi phí thật, không phải an toàn miễn phí.
2. **Liên quan và mới quan trọng hơn đầy đủ.** Một dữ kiện đúng nhưng đã lỗi thời, hoặc đúng nhưng không dùng tới trong việc hiện tại, đều làm loãng tín hiệu.
3. **Mọi thành phần đều phải trả giá công khai:** system prompt, từng tool schema, lịch sử, ngữ cảnh chạy và kết quả tool đều được đo bằng token và tính vào ngân sách của step (mục 6). Không đo được thì không quản được.
4. **Ưu tiên truy hồi đúng lúc hơn nạp trước** đối với dữ kiện lớn, thay đổi nhanh hoặc ít khi dùng (mục 5). Chỉ nạp trước dữ kiện nhỏ, ổn định và gần như luôn cần.
5. Khi việc dài hơn cửa sổ, đừng phình ngữ cảnh: chuyển sang nén, ghi chú ngoài ngữ cảnh hoặc agent con (mục 9; chi tiết ở file 10).

---

## 2. Prompt gồm các mảnh (sections)

System prompt không phải một chuỗi viết cứng. Nó được ghép từ các **mảnh** do plugin đăng ký:

```
PromptSection = {
  name,            // duy nhất; trùng → lỗi ngay
  order,           // số thứ tự; ghép tăng dần, hoà bằng tên
  text | (ctx) -> text,   // tĩnh hoặc tính theo lần ghép
  interpolate?,    // có thay biến {{...}} hay không (mặc định có)
  complete?,       // mảnh này LÀ toàn bộ prompt (xem dưới)
}
```

- Mỗi lần ghép, các mảnh được phân giải theo **scope** của agent đang gọi: mảnh global + mảnh scoped của agent đó; mảnh scoped trùng tên che mảnh global.
- Có các **khe đặt tên dùng chung** ở hai đầu (prefix/suffix của "persona") để cấu hình triển khai hoặc một agent cụ thể chèn cá tính của mình mà không phải sửa các mảnh giữa.
- Đúng **một** mảnh hiệu lực được đánh dấu `complete` thì nó là toàn bộ prompt (các mảnh khác vẫn được phân giải để lấy tool/ngữ cảnh, nhưng bị thay bằng mảnh này). Hai mảnh `complete` cùng hiệu lực → lần ghép lỗi, fail loud.
- Thứ tự mảnh phải **ổn định và thân thiện với cache**: cùng đầu vào → cùng chuỗi ký tự. Đừng đưa dữ kiện thay đổi liên tục (giờ hiện tại…) vào prompt — chỗ của chúng là runtime context (mục 4).

Prompt đã ghép được ghi vào log thành `system/message` theo cơ chế ở file 04, mục 5.

### 2.1. Độ cao đúng của chỉ thị (altitude)

Mỗi mảnh phải nằm ở "độ cao" vừa phải giữa hai cực đều hỏng:

| Cực hỏng | Triệu chứng |
|---|---|
| Quá thấp: logic cứng kiểu if-else viết bằng lời | Prompt dài, giòn, phải sửa mỗi khi gặp trường hợp mới; mô hình bị trói vào đúng kịch bản đã liệt kê và hụt ở tình huống chưa liệt kê |
| Quá cao: khuyên chung chung | "Hãy làm việc cẩn thận" — không cho mô hình tín hiệu cụ thể nào về kết quả mong muốn, và thường giả định ngữ cảnh chung không tồn tại |

Độ cao đúng: đủ cụ thể để dẫn hành vi (nêu kết quả mong muốn, ràng buộc cứng, và heuristic khi phải tự quyết), đủ mềm để mô hình áp dụng heuristic đó cho tình huống chưa liệt kê. Quy tắc cứng về an toàn/quyền (file 13) không nằm ở đây để "khuyên" — chúng được thực thi bằng cơ chế (approval, pipeline, sandbox); prompt chỉ nhắc mô hình về sự tồn tại của chúng khi điều đó giúp nó chọn hành động.

Thực hành bắt buộc:

- **Bắt đầu tối thiểu.** Viết bản prompt nhỏ nhất nêu đúng hành vi mong muốn, kiểm thử trên việc thật, rồi chỉ thêm chỉ thị khi thấy một kiểu thất bại cụ thể. Mỗi câu trong prompt phải trả lời được "câu này ngăn kiểu thất bại nào?" — không trả lời được thì xoá.
- Tổ chức prompt thành các phần có ranh giới rõ (tiêu đề markdown hoặc thẻ phân đoạn) để mô hình và người đọc đều định vị được từng loại chỉ thị. Định dạng chính xác ít quan trọng hơn việc các phần không trộn lẫn nhau.
- Mảnh nào cũng có chủ sở hữu và được tính token như mọi thành phần khác (mục 6). Một mảnh phình liên tục qua nhiều lần sửa là tín hiệu nên tách việc của nó ra tool, skill hoặc agent con thay vì tiếp tục nhồi chữ.

### 2.2. Ví dụ chuẩn (few-shot) là một loại mảnh

Ví dụ minh hoạ hành vi mong muốn thường đáng giá hơn nhiều đoạn mô tả trừu tượng. Quy tắc:

- Dùng một tập **ít, đa dạng và điển hình (canonical)**: mỗi ví dụ thể hiện một hành vi cốt lõi, các ví dụ khác nhau đủ để mô hình suy ra quy tắc chung.
- Không liệt kê mọi trường hợp biên thành danh sách dài trong prompt. Trường hợp biên hiếm thuộc về tool (kiểm chứng và báo lỗi có hướng dẫn, file 06), skill nạp theo yêu cầu (file 10, mục 8), hoặc dữ kiện truy hồi đúng lúc — không thuộc prompt luôn bật.
- Ví dụ là một phần của prompt đã ghép: chúng đi qua cùng cơ chế ghi log của `system/message`, đổi ví dụ cũng tạo series prompt mới như đổi bất kỳ mảnh nào.

---

## 3. Tool schemas trong lần ghép

Cùng một lần ghép với prompt, bộ ghép thu thập `ToolSchema[]` từ tool registry theo scope hiện tại (file 06, mục 3). Prompt và tool list là hai mặt của **cùng một lần ghép** — chúng phải nhất quán: tool được mô tả trong prompt thì phải có schema, và ngược lại. Kết quả ghép (cấu hình gọi + tool schemas) được ghi vào `request/header` (file 04).

Tool schemas cũng là token: một bộ tool lớn vừa tốn ngân sách ở mọi step vừa tạo điểm quyết định mơ hồ cho mô hình. Việc tuyển chọn bộ tool tối thiểu cho từng scope là trách nhiệm thiết kế ở file 06, mục 2; bộ ghép ở đây chỉ đo và báo tổng chi phí của chúng (mục 6).

---

## 4. Runtime context: dữ kiện động đưa vào như tin nhắn

Những dữ kiện thay đổi trong phiên — chỉ thị trong workspace (file hướng dẫn ở thư mục dự án), thư mục làm việc hiện tại, mốc thời gian, ảnh chụp trạng thái của các hệ con, thông báo việc vừa xảy ra — **không** nhét vào system prompt. Chúng đi qua kênh riêng:

```
PromptContext = { name, order, text | (ctx) -> text, interpolate?, required? }
```

- Mỗi lần chuẩn bị step, vòng lặp tính lại các đóng góp; bản chụp đầy đủ hiện tại chỉ được ghi thành một `user/message` (nguồn: runtime context) **khi nó thay đổi** so với bản đang hiệu lực trong lịch sử, hoặc khi compaction đã che mất bản cũ.
- Vì nó là `user/message` trong log: replay thấy lại đúng, và nó nằm *sau* lịch sử đã giữ — không phá cache tiền tố.
- Bản chụp mới phải tự nêu nó **thay thế** bản chụp cùng loại trước đó (bằng `source.form` và nội dung tự mô tả), để mô hình biết lấy bản mới nhất làm hiệu lực thay vì cộng dồn các bản mâu thuẫn nhau.
- `required: true` giữ một đóng góp vận hành kể cả khi ngữ cảnh tuỳ chọn bị tắt theo cấu hình.
- Thông tin trong context phải tự mô tả nguồn gốc qua `source.form` (file 05, mục 1.1): chỉ thị / danh mục / ảnh chụp / thông báo… để UI trình bày đúng và để model hiểu đây là loại dữ kiện gì.
- Mỗi đóng góp phải tự giới hạn kích thước và chịu trách nhiệm cắt tỉa phần của mình (ví dụ danh mục dài thì chỉ đưa mục lục + cách tra cứu, xem mục 5). Bộ ghép không "cắt hộ" một đóng góp nó không hiểu — nó đo, báo vượt ngân sách (mục 6) và chủ sở hữu của đóng góp quyết định cắt gì.

---

## 5. Truy hồi đúng lúc (just-in-time) và chiến lược lai

Có ba chiến lược đưa dữ kiện vào ngữ cảnh. Chọn theo tính chất của dữ kiện, không theo thói quen:

| Chiến lược | Cách làm | Dùng khi | Đánh đổi |
|---|---|---|---|
| Nạp trước (pre-load) | Đưa toàn văn vào prompt/runtime context từ đầu phiên | Dữ kiện nhỏ, ổn định, gần như step nào cũng cần (chỉ thị workspace ngắn, thư mục làm việc) | Tốn token mọi step; rủi ro lỗi thời nếu dữ kiện đổi mà bản chụp không đổi |
| Đúng lúc (JIT) | Ngữ cảnh chỉ giữ **định danh nhẹ** — đường dẫn file, truy vấn đã lưu, liên kết, id, tên mục — mô hình dùng tool để nạp nội dung thật khi cần | Corpus lớn, dữ kiện thay đổi nhanh, hoặc chỉ một phần nhỏ được dùng trong một việc (file trong repo, hàng trong cơ sở dữ liệu, trang web) | Chậm hơn một nhịp tool; cần tool và heuristic đủ tốt để mô hình không lạc đường |
| Lai (hybrid) | Nạp trước phần khung ổn định, để phần lớn cho JIT | Mặc định cho hầu hết harness làm việc với workspace: chỉ thị dự án nạp trước, nội dung file/tài liệu nạp bằng glob/grep/đọc theo đoạn khi cần | Phải quyết định rõ ranh giới hai phần; ranh giới sai thì hoặc phình ngữ cảnh hoặc thiếu dữ kiện nền |

Nguyên tắc khi dùng JIT:

1. **Định danh phải mang tín hiệu.** Tên file/thư mục, cấu trúc cây, thời gian sửa, kích thước — metadata quanh định danh giúp mô hình đoán đúng chỗ cần mở mà chưa tốn token nội dung. Công cụ liệt kê/tìm kiếm phải trả các metadata này (file 06, mục 2).
2. **Khám phá tăng dần (progressive disclosure).** Mỗi lần truy hồi chỉ mở một lớp: mục lục → mục liên quan → đoạn liên quan. Đừng trả toàn bộ tài liệu khi mô hình mới cần định hướng.
3. **Dữ kiện đã nạp vẫn qua log như thường.** Nội dung JIT vào ngữ cảnh bằng kết quả tool hoặc `inject` — cả hai đều là sự kiện trong log (mục 7). JIT thay đổi *khi nào* dữ kiện vào, không thay đổi quy tắc 3.2.
4. **Cho mô hình heuristic tìm kiếm, không chỉ cho tool.** Mảnh prompt hoặc mô tả tool phải nói rõ: bắt đầu từ đâu, tìm bằng gì trước, khi nào đào sâu, khi nào dừng. Không có heuristic, JIT biến thành lãng phí ngữ cảnh vì mò sai và đọc thừa.
5. **Tránh nạp trước bằng chỉ mục dễ lỗi thời.** Nếu vẫn dùng chỉ mục/embedding dựng sẵn, nó là tầng tăng tốc cho JIT chứ không phải nguồn sự thật: kết quả cuối cùng phải kiểm chứng được từ dữ kiện gốc mà định danh trỏ tới.

---

## 6. Ngân sách token của một step

Trước khi phát yêu cầu, bộ ghép phải biết lần gọi này tốn bao nhiêu và còn bao nhiêu. Đây là dữ kiện vận hành bắt buộc, không phải thống kê tuỳ chọn.

### 6.1. Đo theo loại

Bộ ghép (cùng bộ đo token ở file 10, mục 6.1) tính, cho step sắp phát:

| Loại | Gồm |
|---|---|
| `system` | System prompt đã ghép (mọi mảnh, kể cả ví dụ) |
| `tools` | Toàn bộ `ToolSchema[]` của scope |
| `history` | Lịch sử suy ra từ log trên bề mặt hiện tại |
| `runtime` | Bản chụp runtime context đang hiệu lực |
| `reservedOutput` | Phần dự trữ cho đầu ra, lấy từ trần output của tuyến (file 05, mục 4) |

- Tổng `system + tools + history + runtime + reservedOutput` được so với cửa sổ ngữ cảnh của tuyến đã chụp cho step (metadata đúng-tuyến, file 05, mục 4). Thiếu metadata cửa sổ → không đoán: báo rõ là chưa đo được và coi như chưa có bảo vệ áp lực (fail loud ở tầng cấu hình, không chặn step).
- Bộ đo đếm từ chính dữ kiện sắp gửi (prompt đã render, schema đã thu thập, lịch sử đã suy ra), tính cả chi phí ảnh theo tuyến hiện tại — không đếm ký tự rồi chia ước lượng ở nơi khác.
- Kết quả đo **tính lại được từ log** cho mọi step (cùng log + cùng bộ đo → cùng con số), và bộ ghép ghi một bản chụp của nó vào `request/context` khi vượt một ngưỡng đã khai, để chẩn đoán không phải tính lại (file 13, mục 5). Đo lường không phải dữ kiện cho mô hình thấy, trừ khi một đóng góp runtime context cố ý công bố phần tóm tắt của nó.

### 6.2. Thang xử lý khi áp lực tăng

Ngưỡng là trường cấu hình của triển khai (không viết cứng trong vòng lặp), nhưng thang xử lý thì cố định — rẻ và ít mất mát trước:

1. **Cắt kết quả tool đã hết giá trị** trên bề mặt (file 10, mục 6.1): kết quả thô sâu trong lịch sử mà mô hình không cần đọc lại nguyên văn nữa.
2. **Dựa vào ghi chú có cấu trúc** đã ghi ở các mốc trước (file 10, mục 6.2) để bản tóm tắt nén được ngắn mà không mất quyết định/việc còn mở.
3. **Nén (compaction)** đoạn lịch sử cũ thành tóm tắt thay thế trên bề mặt (file 10, mục 6.1).
4. **Cách ly việc con** sang agent con có ngữ cảnh sạch, chỉ nhận kết quả đã chưng cất (file 10, mục 1) — cho việc còn lại đủ dài để đáng trả chi phí phối hợp.

Vòng lặp tự nó không nén lén: cơ chế tự động ở các bậc trên là consumer đã đăng ký (lệnh người dùng hoặc bộ theo dõi áp lực), và mọi thay đổi bề mặt đều qua sự kiện log như file 04 quy định. Không có provider cho một bậc → bậc đó vắng mặt; chạm trần cửa sổ thật thì đó là lỗi rõ ràng của step (file 05, mục 1.3), không phải hành vi cắt ngầm.

---

## 7. Ba đường đưa dữ kiện vào một agent đang sống

Nhắc lại từ file 03, dưới góc nhìn "ai được viết gì vào đầu model":

| Đường | Ai dùng | Ghi log thành |
|---|---|---|
| Tin nhắn người dùng (`followup`/`steer`) | Người dùng, hệ thống ngoài qua API | `user/message` nguồn user |
| `inject()` | Plugin: thông báo file đổi, skill được nạp, việc nền xong, nhắc nhở định kỳ… | `user/message` nguồn của producer đó |
| Runtime context refresh | Vòng lặp tự làm mỗi step | `user/message` bản chụp (chỉ khi đổi) |

Không tồn tại đường thứ tư "gắn thẳng vào yêu cầu lúc gọi". Nếu một plugin muốn model thấy điều gì, nó phải đi qua một trong ba đường trên. Truy hồi JIT (mục 5) không phải đường thứ tư: nó dùng kết quả tool và `inject` của chính ba đường này.

---

## 8. Biến prompt

- Mảnh/context được phép chứa `{{biến}}`; biến do plugin đăng ký (cũng theo scope, cũng có disposer).
- `interpolate: false` giữ nguyên văn — cần cho nội dung có chuỗi giống cú pháp biến nhưng là dữ kiện thật (ví dụ tên thư mục).
- Biến không tồn tại lúc render → lỗi fail loud hoặc giữ nguyên kèm chẩn đoán, nhưng **không** lặng lẽ thay bằng chuỗi rỗng. Chọn một hành vi, ghi vào spec cài đặt của bạn và test nó.

---

## 9. Việc dài hơn cửa sổ: ba kỹ thuật, một chủ sở hữu ở file 10

File này chỉ chốt ranh giới; cơ chế đầy đủ nằm ở file 10:

| Kỹ thuật | Giữ lại gì | File sở hữu |
|---|---|---|
| Nén (compaction) | Luồng hội thoại liên tục, đổi độ dài lấy bản tóm tắt có độ trung thực cao | 10, mục 6.1 |
| Ghi chú có cấu trúc (structured note-taking) | Tiến độ, quyết định, việc còn mở — nằm *ngoài* cửa sổ, đọc lại khi cần | 10, mục 6.2 |
| Agent con (sub-agent) | Việc con được cách ly trong ngữ cảnh sạch; agent cha chỉ nhận bản chưng cất + tham chiếu | 10, mục 1 |

Bảng chọn giữa chúng theo tính chất việc: file 10, mục 6.3. Điểm chung bắt buộc: cả ba đều là thay đổi *bề mặt gửi model* hoặc *nội dung được nạp lại*, nên tất cả đều phải tái dựng được từ log như mọi dữ kiện model thấy khác.

---

## 10. Ranh giới với các file khác

- Cách prompt trở thành `system/message` và khi nào series mới bắt đầu: file 04, mục 5.
- Lựa chọn model được chụp cho lần ghép: file 03, mục 5.
- Skill (gói chỉ thị nạp theo yêu cầu) là một *producer* của ngữ cảnh qua `inject`, không phải một phần của bộ ghép: file 10, mục 8. Skill là ứng dụng trực tiếp của mục 5: danh mục nhẹ luôn sẵn, toàn văn chỉ nạp khi cần.
- Bộ đo token và cơ chế nén/ghi chú: file 10, mục 6. Cách đánh giá một thay đổi prompt/tool/ngữ cảnh bằng số đo thay vì cảm tính: file 14, mục 2.

---

## 11. Tiêu chí nghiệm thu

- [ ] Đổi một mảnh prompt bằng cấu hình/plugin → step sau phản ánh prompt mới đúng cơ chế series (file 04), step trước replay vẫn ra prompt cũ.
- [ ] Hai plugin đăng ký mảnh trùng tên trong cùng scope → lỗi ngay lúc đăng ký.
- [ ] Runtime context không đổi qua 3 step → chỉ đúng một `user/message` ngữ cảnh trong log; đổi thư mục làm việc → có bản chụp mới và bản mới tự nêu nó thay bản cũ.
- [ ] Một agent scoped có persona riêng → prompt của nó khác agent bên cạnh, trong cùng tiến trình.
- [ ] Với một step bất kỳ, tính lại được từ log số token theo từng loại ở mục 6.1 và tổng so với cửa sổ của tuyến; tăng giả tạo một mảnh prompt lớn → số đo phản ánh đúng ở step kế tiếp, và khi vượt ngưỡng đã khai thì log có bản chụp số đo trong `request/context`.
- [ ] Một dữ kiện lớn khai báo theo chiến lược JIT: step đầu chỉ chứa định danh của nó; sau khi mô hình gọi tool truy hồi, nội dung xuất hiện trong log dưới dạng kết quả tool và replay tái dựng đúng.
- [ ] Ép áp lực ngữ cảnh tăng dần trên một session dài: các bậc ở mục 6.2 được áp theo đúng thứ tự đã cấu hình, và khi không có provider nén nào thì step chạm trần báo lỗi rõ ràng thay vì bị cắt ngầm.
