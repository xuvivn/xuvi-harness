# 10 — Phối hợp và mở rộng: subagent, việc nền, workflow, mục tiêu, nén, skills

> File này gom các khả năng xây **trên** vòng lặp cốt lõi. Điểm chung: không khả năng nào sửa vòng lặp — tất cả cắm vào qua registry agent, inbox, tool registry và session log. Đây là bằng chứng sống của quy tắc 3.1 ở file 01: nếu bạn thấy mình phải sửa agent loop để thêm một mục dưới đây, hãy dừng lại và tìm điểm cắm.

Đọc trước: [03](03-agent-loop.md), [04](04-session-log.md), [06](06-tools-va-thuc-thi.md), [08](08-capability-seams.md).

---

## 1. Subagent: uỷ quyền cho một agent khác

### 1.1. Ý tưởng

Một agent (cha) giao một việc con cho một agent khác (con) hoặc một hệ agent ngoài, rồi nhận kết quả. Tất cả qua một seam:

```
Service Definition: registry các backend uỷ quyền + quản lý vòng đời lần kích hoạt
Provider (backend):   - tạo agent con mới trong cùng tiến trình
                      - fork từ lịch sử hiện tại của cha trong cùng tiến trình
                      - qua SDK (tiến trình harness khác)
                      - qua giao thức agent ngoài (hệ khác hẳn)
Consumer:             tool uỷ quyền cho model, workflow, cơ chế vòng lặp tươi (mục 7)
```

### 1.2. Hợp đồng

- Mỗi backend khai báo **năng lực** (cờ: hỗ trợ tuỳ chọn agent? chọn preset riêng? ràng buộc schema đầu ra? giới hạn độ sâu?) — yêu cầu dùng tuỳ chọn nào thì backend đó phải có cờ tương ứng, kiểm chứng *trước khi* bắt đầu.
- Một lần kích hoạt có: định danh ổn định của con, nhãn việc, cách **giao kết quả** (`parent`: thông báo cho model của cha qua inject · `caller`: chỉ trả cho nơi gọi đang chờ), và một thao tác dispose dừng đúng lần kích hoạt đó cùng hậu duệ của nó.
- **Fork trong tiến trình**: con bắt đầu từ tiền tố đã hoàn tất của log cha (điểm cắt có quy tắc riêng: được cắt tại biên hoàn tất gần nhất ngay trong turn đang mở — khác fork cho người dùng ở file 04), giữ nguyên thư mục làm việc hiệu lực của cha.
- **Lineage là dữ liệu**: session cha, độ sâu uỷ quyền bền vững, độ sâu lúc chạy — mang theo dạng trường dữ liệu, không ảnh hưởng quyền nhìn thấy theo scope (file 02).
- Độ sâu uỷ quyền có trần cấu hình được; vượt trần thì từ chối bắt đầu.
- Con lịch sử mà backend không còn mô tả được vẫn *liệt kê* được với năng lực "không rõ" — liệt kê được không có nghĩa tiếp tục được.
- **Cách ly ngữ cảnh là mục đích chính, không phải chi tiết phụ.** Con làm việc trong cửa sổ riêng của nó: phần khám phá chi tiết (truy vấn, kết quả tool thô, ngõ cụt) ở lại log của con, không chảy vào lịch sử của cha. Cha chỉ giữ kế hoạch, các lần uỷ quyền và kết quả đã chưng cất (mục 1.3). Một backend "con" mà sao chép toàn bộ transcript của con về cha thì không đạt mục đích này, dù cơ chế uỷ quyền chạy đúng.
- **Mô tả việc giao cho con phải tự chứa đủ bốn thứ:** mục tiêu cụ thể, định dạng kết quả mong muốn, hướng dẫn dùng tool/nguồn nào, và ranh giới (phần việc con *không* làm, để các con không giẫm nhau và không bỏ sót khoảng giữa). Giao việc một câu mơ hồ ("nghiên cứu X") là lỗi của bên giao, không phải của con.
- **Quy mô uỷ quyền theo độ phức tạp của việc:** việc đơn giản thì không sinh con nào (hoặc một con với ngân sách tool nhỏ); việc gồm nhiều hướng độc lập mới sinh nhiều con song song, mỗi con một ranh giới rõ. Số con, ngân sách tool và ngân sách token của mỗi lần kích hoạt là cấu hình của consumer giao việc — cơ chế nhiều agent nhân chi phí token lên nhiều lần so với một agent, nên chỉ dùng khi tính song song hoặc lượng thông tin vượt một cửa sổ đáng giá hơn chi phí đó.

### 1.3. Giao kết quả cho cha

Kết quả của con (hoàn thành/lỗi/bị dừng) trở thành một thông báo inject vào cha (kèm định danh con và việc). Cha đang bận thì thông báo chờ ở inbox; cha idle và thông báo thuộc loại đánh thức thì cha mở turn mới để xử lý. Xem "exchange" ở mục 8.

Thông báo kết quả đó bị giới hạn kích thước theo cấu hình và chỉ gồm: **bản chưng cất** (kết luận, dữ kiện quyết định, độ tin cậy/giới hạn nếu có) cộng **tham chiếu** để cha đào sâu khi cần — định danh session của con, đường dẫn artifact con đã ghi ra kho phụ hoặc workspace, id của các nguồn chính. Kết quả thô dài (toàn văn tìm kiếm, log chạy, bảng dữ liệu lớn) được con ghi ra nơi lưu trữ ngoài ngữ cảnh rồi trỏ tới bằng tham chiếu, không nhét qua thông báo: cha muốn xem chi tiết thì truy hồi đúng lúc bằng tham chiếu đó (file 07, mục 5). Nhờ vậy chi phí ngữ cảnh của cha cho mỗi con là một bản tóm tắt ngắn, dù con đã tiêu hàng chục nghìn token để khám phá.

---

## 2. Việc nền (background jobs)

Một registry việc dài hạn độc lập với turn:

```
Job = { id (có nhãn, do producer đặt tiền tố loại), ownerSession, kind,
        label, status: running|stopping|completed|killed|failed,
        progress?, detail?, startedAt, finishedAt? }
```

- **Chủ sở hữu là một session.** Việc ghi hai sự kiện log-only vào session của chủ: `job/registered` lúc bắt đầu, `job/settled` lúc kết thúc. Thiếu sự kiện kết thúc = việc chưa rõ kết cục (kể cả khi tiến trình chết giữa chừng) — consumer không được bịa thời điểm kết thúc.
- Turn kết thúc **không** có nghĩa việc của turn đó đã dừng. Các tool kiểm soát việc (xem output, dừng) đọc registry sống, không đọc log.
- Loại việc (kind) mở rộng được; registry đối xử với kind như không gian tên mờ.
- Huỷ việc là yêu cầu hợp tác tới producer; trạng thái `stopping` tồn tại vì việc không dừng tức thì được.

---

## 3. Workflow: điều phối nhiều agent bằng kịch bản

Một seam động cơ workflow: nhận một kịch bản điều phối (fan-out nhiều agent con song song, pipeline nhiều giai đoạn, tổng hợp kết quả) và chạy nó như **một tool** từ góc nhìn của agent gọi.

- Kịch bản thuần điều phối: nó không có filesystem/mạng riêng; mọi việc thật vẫn qua agent con và tool của chúng.
- Lỗi một nhánh → nhánh đó cho kết quả rỗng/lỗi, các nhánh khác tiếp tục; tầng tổng hợp quyết định có dùng kết quả thiếu hay không. Lỗi cú pháp kịch bản là lỗi của lần gọi, không phải của từng nhánh.
- Giới hạn đồng thời, ngân sách agent con và timeout là cấu hình của động cơ — không viết cứng trong kịch bản.

## 4. Lịch trình (schedule)

Một dịch vụ hẹn giờ thuộc về host: tới hạn thì gửi một tin nhắn đánh thức vào một session đã chỉ định (tạo thành turn như bình thường).

- Lịch là dữ liệu bền của host (không phải của session): một mục gồm thời điểm/quy tắc lặp, session đích, nội dung tin nhắn.
- Trễ do tiến trình tắt: khi dậy lại, một lịch lặp chỉ bù lần lỡ gần nhất, không bù dồn.
- Người dùng quản lý lịch bằng lệnh/tool chuyên dụng; bản thân việc gửi tin nhắn vẫn qua inbox (file 03) — schedule không có đường tắt vào vòng lặp.

## 5. Mục tiêu (goal)

Một mục tiêu hoàn thành gắn với một session đang có:

```
Goal = { objective, phase: active|paused|blocked|complete,
         revision, goalRoundCap, blockedReason? }
```

- Mục tiêu là **trạng thái**, không phải bộ hẹn giờ hay cuộc hội thoại riêng. Nguồn sự thật vẫn là log của session.
- Khi mục tiêu `active` và được *kích hoạt* (armed), một consumer tiếp nối tự động mở các **vòng mục tiêu** (goal round): mỗi vòng là một turn sinh ra từ mục tiêu. Turn của người dùng xen giữa không tính vào trần số vòng.
- Kích hoạt là quyền *trong tiến trình*, cố ý không bền: resume/fork xong thì mục tiêu ở trạng thái chưa kích hoạt, phải có người (lệnh `/goal` hoặc tool) cho phép lại mới tự chạy tiếp. Đây là chốt an toàn chống "session cũ tự sống dậy chạy việc".
- `blocked` phải kèm mã lý do có cấu trúc và lời giải thích; `complete` chỉ do chủ thể có thẩm quyền tuyên bố theo tiêu chí của mục tiêu.

## 6. Ngữ cảnh dài hạn: nén, ghi chú và cách chọn

Việc dài hơn cửa sổ ngữ cảnh có ba kỹ thuật bổ trợ nhau (bối cảnh và thang áp lực: file 07, mục 6.2). Mục này định nghĩa cơ chế; mục 6.3 chốt cách chọn.

### 6.1. Nén ngữ cảnh (compaction)

Khi lịch sử dài tới gần cửa sổ ngữ cảnh, một seam nén thay đoạn cũ bằng tóm tắt:

```
Service Definition: thao tác nén trên một session, kết quả và sự kiện của nó
Provider:           cơ chế tóm tắt cụ thể (gọi model tóm tắt / cắt tỉa không cần model / bỏ ảnh…)
Consumer:           lệnh người dùng, và cơ chế tự động khi áp lực ngữ cảnh cao
```

Hợp đồng:

1. Ba sự kiện log-only ôm trọn thao tác: `compaction/start` (khoá) → `compaction/summary` (tóm tắt + đoạn bị che + các seq bị che + số token + envelope của lần gọi tóm tắt) → `compaction/end` (nhả khoá, kèm lỗi nếu thất bại). Crash giữa chừng để lại khoá mồ côi **phát hiện được** — thà thế còn hơn một `end` giả vờ đã xong.
2. Phần duy nhất chạm bề mặt là *một* `user/message` tóm tắt mang `surfaceOp: replace` trên đúng đoạn đã chọn (file 04, mục 4).
3. Đo áp lực ngữ cảnh bằng một bộ đo token riêng (dựng lại từ log, tính cả chi phí ảnh theo tuyến hiện tại); quyết định nén dựa trên số đo đó, không dựa trên đếm ký tự. Đây cũng là bộ đo dùng cho ngân sách của từng step (file 07, mục 6.1) — một bộ đo duy nhất, không hai con số lệch nhau.
4. Trước khi tóm tắt bằng model, được phép *tỉa rẻ trước*: thay các kết quả tool quá lớn hiện tại bằng bản rút gọn qua thay thế bề mặt đơn điểm, và *dỡ ảnh cũ* ra khỏi lịch sử bằng projection (file 04, mục 7) khi provider tính phí ảnh cao. Cắt riêng kết quả tool đã hết giá trị (**tool-result clearing**) là bậc nhẹ nhất của thang áp lực: một kết quả thô sâu trong lịch sử hiếm khi cần đọc lại nguyên văn, và cắt nó không cần gọi model tóm tắt.
5. **Giữ gì và bỏ gì là hợp đồng của bản tóm tắt, phải khai trước:** giữ quyết định kiến trúc/thiết kế đã chốt, việc chưa xong và lỗi chưa giải quyết, ràng buộc người dùng đã nêu, định danh/tham chiếu tới artifact quan trọng, và trạng thái cần để bước tiếp theo đúng; bỏ kết quả tool thô đã tiêu hoá xong, vòng lặp thử-sai đã kết luận, và lời chào hỏi/xã giao. Sau khi nén, các artifact vừa được dùng gần nhất (ví dụ vài file vừa đọc/sửa) được nạp lại qua runtime context hoặc truy hồi đúng lúc thay vì kỳ vọng bản tóm tắt chứa chúng.
6. **Tinh chỉnh prompt tóm tắt theo thứ tự recall → precision:** đầu tiên chỉnh cho không sót dữ kiện quan trọng (recall), kiểm trên các trace dài thật; sau đó mới cắt phần thừa (precision). Nén quá tay làm mất dữ kiện mà tầm quan trọng chỉ lộ ra nhiều step sau, và mất mát đó không khôi phục được từ bề mặt — chỉ còn đường đọc lại đoạn gốc trong log (transcript vẫn giữ nguyên, file 04, mục 4).
7. Nén là khả năng **tuỳ chọn**: vòng lặp chạy bình thường khi không có provider nén nào; lúc đó chạm trần ngữ cảnh là một lỗi rõ ràng, không phải hành vi nén lén.

### 6.2. Ghi chú có cấu trúc (structured note-taking)

Agent ghi tiến độ ra **bộ nhớ ngoài cửa sổ ngữ cảnh** — một file ghi chú trong workspace, một kho key-value của seam storage, hoặc trạng thái todo suy ra từ log — rồi đọc lại khi cần. Đây là kỹ thuật rẻ nhất trong ba kỹ thuật: không cần lần gọi model tóm tắt riêng, và agent tự quyết định cái gì đáng nhớ ngay lúc nó còn biết rõ nhất.

Hợp đồng:

1. **Cấu trúc ghi chú tối thiểu:** mục tiêu hiện tại · đã xong gì (theo mốc) · quyết định đã chốt kèm lý do một dòng · việc còn mở và bước tiếp theo · vướng mắc/lỗi chưa giải quyết · tham chiếu (đường dẫn, id, liên kết) tới dữ kiện gốc thay vì sao chép nội dung. Đếm/tiến độ định lượng (đã xử lý bao nhiêu mục, còn bao nhiêu) ghi bằng số, không ghi cảm tính.
2. **Ghi ở các mốc, không ghi liên tục:** sau khi hoàn thành một pha, trước một lần nén dự kiến, sau một chuỗi tool dài, và trước khi dừng việc dài. Đọc lại ở đầu phiên resume, sau khi nén che mất đoạn chứa tiến độ, và khi bắt đầu một pha mới.
3. **Ghi chú không phải đường tắt qua log.** Viết ghi chú bằng tool thì lần viết là `tool/call`/`tool/result` trong log như thường; nội dung ghi chú chỉ vào ngữ cảnh khi được nạp lại qua kết quả tool hoặc `inject` — cả hai đều là sự kiện log (file 07, mục 7). Ghi chú lưu ngoài log được (đó là bản chất của kỹ thuật), nhưng *việc mô hình đã đọc bản nào* phải tái dựng được từ log.
4. Ghi chú là **dữ kiện của phiên, không phải chỉ thị hệ thống**: nó nằm ở mức tin nhắn thường khi được nạp lại, không leo lên system prompt, và nội dung sao chép từ nguồn ngoài (file người khác, trang web) vẫn là dữ kiện không tin cậy như mọi nội dung tool khác (file 13, mục 1).
5. Cơ chế sẵn có đã đủ để cài đặt: todo của model (mục 9) là một dạng ghi chú; một tool đọc/ghi file ghi chú chuyên dụng là dạng khác. Không thêm khái niệm vào vòng lặp cho kỹ thuật này.

### 6.3. Chọn kỹ thuật nào

| Tính chất việc | Kỹ thuật chính | Vì sao |
|---|---|---|
| Hội thoại qua lại dày, cần giữ mạch trao đổi liên tục | Nén (6.1) | Giữ luồng hội thoại trong cùng một session; ghi chú rời rạc không thay được mạch đó |
| Việc lặp theo mốc rõ ràng (làm từng pha, từng đợt, có tiến độ đếm được) | Ghi chú (6.2), nén làm phụ | Tiến độ sống ngoài ngữ cảnh nên nén/reset không làm mất; chi phí gần như bằng không |
| Nghiên cứu/phân tích gồm nhiều hướng độc lập, mỗi hướng ngốn nhiều ngữ cảnh | Agent con (mục 1) | Cách ly khám phá; cha chỉ trả giá cho bản chưng cất |
| Lịch sử phình chủ yếu vì vài kết quả tool khổng lồ | Cắt kết quả tool (6.1, điều 4) trước mọi kỹ thuật khác | Rẻ nhất, không mất dữ kiện nào khác |

Ba kỹ thuật kết hợp được và thường phải kết hợp: ghi chú tốt làm bản tóm tắt nén ngắn và chính xác hơn; agent con ở vòng "agent tươi" (mục 7) thực chất là ghi chú + uỷ quyền. Điều cấm là dùng nén như lựa chọn duy nhất theo mặc định cho mọi việc dài mà chưa hỏi việc đó thuộc hàng nào trong bảng trên.

## 7. Vòng lặp "agent tươi" (Ralph-style)

Một *chính sách* ở tầng tool, không phải chế độ của vòng lặp: chạy nhiều vòng, mỗi vòng là một **agent con hoàn toàn mới** hướng tới cùng một mục tiêu bất biến.

- Con của vòng sau không thừa hưởng hội thoại của vòng trước; thứ mang qua là workspace dùng chung + một **bản bàn giao** có cấu trúc và bị giới hạn kích thước (trạng thái, tóm tắt, bằng chứng, bước tiếp theo, vướng mắc) — đây là ghi chú có cấu trúc ở mục 6.2 dùng làm đầu vào duy nhất của vòng sau.
- Ghép từ hai nguyên thuỷ đã có: workflow (mục 3) và subagent (mục 1). Không thêm khái niệm vào vòng lặp.

## 8. Skills: gói chỉ thị nạp theo yêu cầu

Skill là một gói chỉ thị (văn bản + tệp kèm) mà model *tự phát hiện và nạp khi cần*, thay vì nhét tất cả vào prompt từ đầu.

```
Service Definition: registry hợp nhất các provider skill
Provider:           thư mục trên đĩa · gói đóng sẵn · registry từ xa…
Consumer:           tool `skill` cho model + một danh mục skill theo ngữ cảnh
```

- Registry phân tầng global/scoped như tool registry (file 02): trùng tên thì tầng gần hơn thắng tuyệt đối; trong cùng tầng phân định bằng thứ hạng provider rồi thứ tự.
- Consumer công bố **danh mục** (tên + mô tả ngắn) vào ngữ cảnh của model; model gọi tool để nạp **toàn văn** một skill qua `inject`. Nội dung skill vì thế nằm trong log và tuân quy tắc model-visible ⟺ logged. Đây là khám phá tăng dần ba lớp (file 07, mục 5): danh mục luôn sẵn nhưng rẻ, toàn văn chỉ tốn token khi skill thật sự liên quan, và tệp kèm theo skill chỉ được đọc tiếp khi toàn văn trỏ tới chúng — không lớp nào tự động kéo lớp sau vào ngữ cảnh.
- Provider liệt kê ứng viên (có thể báo "chưa đầy đủ" khi khám phá dở dang — ứng viên vẫn nạp trực tiếp được nhưng không cache như kết quả đầy đủ); nạp toàn văn là thao tác riêng.
- Skill là chỉ thị tuỳ chọn: không có skill nào thì harness vẫn chạy.

## 9. Các mảnh ghép nhỏ thường đi kèm

| Khả năng | Cơ chế (không sửa vòng lặp) |
|---|---|
| Todo của model | Một tool ghi sự kiện log-only; trạng thái suy ra bằng projection |
| Chặn vòng lặp vô ích | Guard nhắc khi model gọi lặp một tool (qua pipeline file 06) + guard deadline cho cả lần thực thi |
| Tự sửa harness lúc chạy | Module cho model xem/gắn/gỡ plugin trong phạm vi cho phép, qua chính module system (file 02) — quyền này theo scope và bị approval kiểm soát |
| Webhook | Một rule tin cậy được đăng ký → sự kiện ngoài tạo/đánh thức session; adapter của từng nguồn webhook là plugin riêng |
| Agent teams (thử nghiệm) | Một dịch vụ phối hợp: danh sách thành viên + hộp thư trực tiếp giữa các subagent tiếp tục được — xây hoàn toàn trên mục 1 |

## 10. Projection: trạng thái suy ra từ log

Nhiều khả năng trên cần "trạng thái hiện tại" (hộp thư, lựa chọn model, mục tiêu, todo…). Mẫu chung:

- Một **đơn vị projection** đăng ký một hàm gấp (fold) thuần: `(trạng thái, sự kiện) → trạng thái`.
- Session phát sự kiện đã chốt → các projection gấp theo; nơi đọc dùng `snapshot()` lấy bản chụp.
- Dịch vụ miền giữ handle riêng của projection và phơi bày phương thức nghiệp vụ; không nơi nào tự quét log tay nếu đã có projection cho việc đó.
- Trạng thái suy ra này **không bao giờ** là nguồn sự thật thứ hai — xoá đi dựng lại từ log phải ra hệt như cũ.

## 11. Tiêu chí nghiệm thu

- [ ] Uỷ quyền một việc con: cha nhận kết quả qua inject, con có session riêng xem lại được; dispose cha thì con bị dừng theo.
- [ ] Một việc nền chạy qua thời điểm turn kết thúc; chủ session xem/dừng được nó bằng tool; log của chủ có đủ cặp registered/settled.
- [ ] Đặt mục tiêu, tắt tiến trình, mở lại: mục tiêu còn đó nhưng không tự chạy cho tới khi người dùng kích hoạt lại.
- [ ] Ép nén trên một session dài: lịch sử sau nén ngắn hơn, đoạn bị che vẫn đọc được ở transcript, step kế tiếp chạy được với lịch sử đã nén.
- [ ] Một quyết định đã chốt và một lỗi chưa giải quyết nằm trong đoạn bị nén: cả hai xuất hiện trong bản tóm tắt trên bề mặt sau nén; một kết quả tool thô đã tiêu hoá trong cùng đoạn thì không.
- [ ] Ghi một ghi chú tiến độ, ép nén (hoặc mở phiên mới từ cùng workspace), rồi tiếp tục: agent đọc lại ghi chú và nêu đúng bước tiếp theo cùng việc còn mở mà không cần đoạn lịch sử đã bị che.
- [ ] Uỷ quyền một việc con ngốn nhiều kết quả tool: lịch sử của cha chỉ tăng thêm bản chưng cất và tham chiếu, không tăng theo kích thước transcript của con; cha dùng tham chiếu để mở đúng một chi tiết trong kết quả của con khi cần.
- [ ] Xoá toàn bộ trạng thái projection và dựng lại từ log: mọi snapshot giống hệt trước khi xoá.
