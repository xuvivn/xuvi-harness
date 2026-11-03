# 01 — Benchmark tổng thể của sản phẩm

> Bộ tiêu chí này đo **cả sản phẩm như một thể sống**: giá trị cho người học và
> người dạy, chất lượng giáo dục, trải nghiệm, kiến trúc harness, độ tin cậy kỹ
> thuật, tính thật của AI, chất lượng học hỏi (research) và sức khoẻ của chính
> quá trình tiến hoá. Một sản phẩm chỉ nhanh, hoặc chỉ nhiều tính năng, hoặc chỉ
> đẹp kiến trúc — đều không đạt benchmark này.
>
> Đứng trên bộ tiêu chí này là [ĐẠO.md](../../ĐẠO.md). Mâu thuẫn thì Đạo thắng.

---

## 1. Cấu trúc: 3 cổng veto + 8 chiều chấm điểm

Điểm số không cứu được một cổng bị phá. Thứ tự đánh giá luôn là:
**cổng trước, điểm sau.**

### 1.1. Ba cổng (ĐẠT / CHƯA ĐẠT / CHƯA ĐO — không chấm số)

| Cổng | ĐẠT khi | CHƯA ĐẠT khi (ví dụ) |
|---|---|---|
| **G1. Đạo & an toàn sự sống** | Không tính năng nào đánh đổi sức khoẻ, sự tự do, nhịp phát triển tự nhiên của trẻ hay người dùng để lấy điểm, tốc độ, hay sự hấp dẫn | Thiết kế gây nghiện/gây áp lực lên trẻ; khuyên nhủ y tế/giáo dục vượt quá căn cứ; nội dung làm người dùng lệ thuộc thay vì tự chủ hơn |
| **G2. Trung thực bằng chứng** | Mọi tuyên bố đúng mức đã kiểm chứng: AI thật được gọi là AI thật, thuật toán cục bộ được gắn nhãn cục bộ, trích dẫn nguồn đúng trang, không tuyên bố đạt chuẩn/chứng nhận chính thức khi chưa có | Gắn nhãn "AI" cho fallback cục bộ; trích dẫn MOET sai trang hoặc bịa; hiển thị tranh mẫu như thể là sản phẩm học sinh; kế thừa nhãn "đã kiểm chứng" từ nơi khác mà không chạy lại |
| **G3. Riêng tư & bí mật** | Không dữ liệu học sinh thật trong seed/demo/benchmark; dữ liệu gửi ra ngoài có đồng ý tường minh cho lần gửi đó; `.env`, khoá, đường dẫn riêng tư không vào specs, log, file kết quả | Seed lấy từ hồ sơ thật; khoá API xuất hiện trong log/frontend/bản sao lưu; gửi toàn bộ repo/nội dung riêng tư cho provider khi chỉ cần một đoạn |

`CHƯA ĐO` nghĩa là chưa đủ bằng chứng để nói ĐẠT — và được đối xử như chưa đạt
khi quyết định giữ thay đổi (xem cổng benchmark trong [cron.md](../../cron.md)).

### 1.2. Tám chiều (thang 0–4, có trọng số)

| # | Chiều | Trọng số | Chiều này hỏi điều gì |
|---|---|---|---|
| B1 | **Giá trị & độ trọn vẹn của sản phẩm** | 20% | Một phụ huynh/giáo viên có hoàn thành được các hành trình cốt lõi đầu-cuối không (với Waldorf: luyện kể XuVi Story; tạo đủ sáu loại hồ sơ từ dữ liệu thật của họ)? |
| B2 | **Chất lượng giáo dục & văn hoá** | 15% | Nội dung có đúng tinh thần Waldorf-inspired, hợp bối cảnh Việt Nam, đối chiếu GDPT có provenance đúng trang, và không tự nhận chứng nhận chính thức? |
| B3 | **Trải nghiệm người dùng** | 10% | Giao diện tiếng Việt rõ, dùng được bằng bàn phím/điện thoại, trạng thái lỗi/rỗng/tải xử lý được, một người mới demo theo kịch bản mà không cần người xây đứng cạnh giải thích? |
| B4 | **Toàn vẹn kiến trúc harness** | 15% | Các bất biến của harness còn giữ: everything-is-a-plugin, model-visible ⟺ logged, đăng ký thu hồi được, seam đủ 3 vai, ngân sách ngữ cảnh đo được (xem `harness/specs/core/01`)? |
| B5 | **Độ tin cậy kỹ thuật & bảo mật cục bộ** | 15% | Build/test/typecheck đạt, dữ liệu bền qua khởi động lại, sao lưu/khôi phục không mất dữ liệu khi lỗi, server chỉ loopback, upload được kiểm soát? |
| B6 | **Tích hợp AI thật & chọn model** | 10% | Các adapter AI (hiện tại: Meta — Wonder/Spark/Image/Voice Transcribe) gọi thật đúng contract, có timeout/redact, tắt mạng vẫn dùng thủ công được, lựa chọn model có lý do trong research? |
| B7 | **Chất lượng nghiên cứu & học hỏi** | 5% | Nguồn mới được kiểm tra có ngày tháng, quyết định adopt/watch/reject được ghi, và nghiên cứu thực sự làm specs tốt lên (không phải danh sách link để đó)? |
| B8 | **Sức khoẻ tiến hoá & vệ sinh specs** | 10% | Thay đổi truy vết được specs → code → kết quả đo; `git status` chỉ gồm diện được commit; kết quả benchmark được lưu đều; loop sau đọc lại hiểu được loop trước? |
| | **Tổng** | **100%** | |

Thang điểm dùng chung cho mọi chiều:

| Điểm | Nghĩa |
|---|---|
| 0 | Chưa có, hoặc có tuyên bố nhưng không có gì chạy được |
| 1 | Có ở mức specs/bản nháp, hoặc chạy được một phần hẹp |
| 2 | Chạy được hành trình chính, bằng chứng còn một phần thủ công/mock |
| 3 | Chạy được đầy đủ, có bằng chứng tự động hoặc live cho phần cốt lõi |
| 4 | Như 3, cộng: chịu được lỗi (offline, provider chết, dữ liệu hỏng) mà không nói dối người dùng, và người ngoài nhóm xây tự dùng được |

**Điểm tổng** = Σ (điểm chiều ÷ 4 × trọng số), thang 100. Chỉ dùng để nhìn
xu hướng giữa các lần đo. Quyết định giữ/huỷ không bao giờ dựa trên tổng một mình.

---

## 2. Mức bằng chứng — và trần điểm nó áp đặt

Mọi bằng chứng trong một lần đo phải tự khai thuộc một trong năm mức:

| Mức | Nghĩa | Trần điểm cho chiều dựa chủ yếu vào mức này |
|---|---|---|
| `live` | Gọi dịch vụ/model thật, hoặc người dùng thật thao tác thật | 4 |
| `tự động` | Test/integration chạy thật trong repo này, lần đo này | 4 (riêng B6 tối đa 3 nếu chưa từng live) |
| `thủ công` | Người/agent mở sản phẩm, thao tác và ghi lại kết quả quan sát | 3 |
| `mock` | Provider giả lập để kiểm contract | 2 |
| `chưa kiểm chứng` | Có tài liệu/tuyên bố, chưa chạy lại ở đây | 1 |

Quy tắc cứng: **tài liệu của một đợt build khác, repo khác, hay thời điểm khác
chỉ là `chưa kiểm chứng` cho lần đo hiện tại**, cho tới khi được chạy lại.
Đây là điểm benchmark này cố tình khắt khe: nó bảo vệ hệ khỏi việc tự kể cho
mình nghe một câu chuyện tiến bộ không có thật — điều Đạo gọi là không trung thực.

---

## 3. Cách đo từng chiều (gợi ý tối thiểu, được phép đo sâu hơn)

| Chiều | Cách đo tối thiểu mỗi lần |
|---|---|
| B1 | Chạy kịch bản hành trình trong `plugins/waldorf/specs/05_acceptance_matrix.md` (hoặc specs tương đương của plugin đang xét) trên dữ liệu minh họa, ghi mục nào qua/không |
| B2 | Lấy mẫu ngẫu nhiên ít nhất 5 trích dẫn/đối chiếu, mở đúng trang nguồn kiểm tra; kiểm không có câu chữ nào tuyên bố chứng nhận chính thức |
| B3 | Đi hết một hành trình bằng giao diện thật (hoặc bản build), gồm một màn hình hẹp; ghi điểm nghẽn cần người xây giải thích (mỗi điểm nghẽn trừ chất lượng) |
| B4 | Kiểm các câu nghiệm thu cuối `harness/specs/core/01` và `02`: cắm/gỡ một plugin thử bằng cấu hình, huỷ agent có dọn sạch đăng ký không — hoặc ghi rõ chưa có cài đặt để kiểm |
| B5 | Chạy typecheck/test/build đã khai trong specs của sản phẩm; thử khởi động lại giữ dữ liệu; thử khôi phục bằng bản sao hỏng và xác nhận dữ liệu cũ còn nguyên |
| B6 | Với mỗi khả năng AI: một lần gọi thật (nếu có khoá và đồng ý gửi dữ liệu mẫu công khai/tổng hợp) hoặc ghi rõ chưa live; một lần rút mạng/key và xác nhận luồng thủ công vẫn trọn |
| B7 | Mở sổ research của đợt: có ngày kiểm tra, nguồn, quyết định adopt/watch/reject không; quyết định đó có vết trong specs không |
| B8 | `git status` + đọc lại file kết quả của loop trước: người đo có hiểu loop trước đã giữ/huỷ gì mà không cần hỏi tác giả không |

---

## 4. Khi nào bộ tiêu chí này tự nó cần sửa?

Benchmark cũng là specs, nên nó cũng tiến hoá — nhưng chậm và có lý do:

- Sửa khi: một chiều không còn phân biệt được tốt/xấu (mọi lần đo đều 4, hoặc đều 0),
  một rủi ro mới xuất hiện mà 3 cổng chưa bao phủ, hoặc sản phẩm thêm plugin mới
  mà B1–B3 chưa nói được ngôn ngữ của nó.
- Mỗi lần sửa phải ghi trong file kết quả của loop đó: sửa gì, vì sao, và từ lần
  đo nào áp dụng. **Không áp dụng ngược** để làm đẹp các kết quả cũ.
- Không sửa benchmark trong cùng một loop với thay đổi sản phẩm mà nó sẽ chấm,
  trừ khi sửa đó là để *khắt khe hơn* — và vẫn phải ghi rõ.
