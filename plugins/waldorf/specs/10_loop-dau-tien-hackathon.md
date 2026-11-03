# 10 — Loop đầu tiên: sản phẩm hackathon, hôm nay và ngày mai

> File này chốt phạm vi của **loop/đợt đầu tiên** trong hệ tự tiến hoá
> (định nghĩa loop: `cron.md` ở gốc repo). Nó đứng dưới `00_master_prompt.md`
> và dưới Đạo. Các file `01`–`09` trong thư mục này là tài liệu của **đợt build
> trước ở repo khác** — chúng là đầu vào tham khảo rất giá trị, nhưng trạng thái
> "đã triển khai / đã kiểm chứng" trong đó **không phải trạng thái của repo này**
> cho tới khi được chạy lại tại đây (baseline: `benchmark/results/2026-11-03_baseline-truoc-loop-1.md`).

## 1. Mốc của đợt (definition of done)

Đợt kết thúc thành công khi có đủ ba điều, theo thứ tự ưu tiên này:

1. **Một hành trình trọn vẹn chạy được tại repo này, demo được cho người lạ:**
   phụ huynh/giáo viên mở sản phẩm → chọn/nhập một truyện dân gian → có bản đồ
   cảnh → luyện kể (ít nhất chế độ có gợi ý và tự do) → nhận góp ý có dẫn chứng,
   không điểm tổng → xem lại lịch sử luyện. Sáu loại hồ sơ trong master prompt
   tạo được từ dữ liệu hiện hành và xuất/in được.
2. **Một kết quả benchmark mới** trong `benchmark/results/` đo đúng sản phẩm vừa
   chạy, với mức bằng chứng khai thật, và `INDEX.md` thêm một dòng có delta
   so với baseline 32.5.
3. **Specs phản ánh đúng điều đã xây:** nếu cách xây thực tế khác `01`–`03`,
   sửa specs cho khớp — không để sản phẩm và tài liệu kể hai câu chuyện khác nhau.

Thiếu điều 1 thì đợt chưa đạt dù benchmark có đẹp. Có điều 1 mà thiếu điều 2–3
thì đợt chỉ là một lần build, chưa phải một vòng tiến hoá.

## 2. Thứ tự làm trong hai ngày (mỗi mục là một loop theo `cron.md`)

| Loop | Điểm nghẽn chính (một) | Đầu ra kỳ vọng |
|---|---|---|
| Loop 1 | Sản phẩm chưa chạy được tại đây (B1/B5 ở trần bằng chứng) | Dựng lại sản phẩm từ specs `01`–`03` trong vùng sinh (bị ignore), một lệnh chạy được, dữ liệu minh họa hư cấu, luồng thủ công trọn vẹn không cần mạng/key |
| Loop 2 | AI thật và trung thực bằng chứng (B6, cổng G2) | Adapter Meta chạy qua sản phẩm này (text/image/voice) với dữ liệu mẫu công khai/tổng hợp và đồng ý gửi tường minh; tắt key/mạng vẫn hoàn thành loop 1; lấy mẫu kiểm provenance MOET (B2) |
| Loop 3 (nếu còn ngân sách) | Demo và trải nghiệm (B3) | Đi hết kịch bản demo bằng giao diện thật kể cả màn hình hẹp, sửa điểm nghẽn cần người hướng dẫn; đo lại toàn diện và ghi kết quả cuối đợt |

Mỗi loop vẫn qua đủ: sửa specs trước nếu ý định đổi → sinh → đo → giữ/huỷ → ghi
kết quả. Hết hai ngày hoặc hết ngân sách trước khi xong loop 3 là dừng bình
thường theo điều kiện dừng của `cron.md` — ghi lại trạng thái, không chạy cố.

## 2b. Điều chỉnh ngày 2026-11-03 (theo yêu cầu người dùng — spec 11)

Người dùng chốt lại: sản phẩm là **macOS native thuần Swift**, chỉ dùng
`META_API_KEY`, với `muse-wonder-1.0` làm orchestrator và
`muse-spark-1.3-contributor` làm worker. Lượt đầu chạy trên DeepSeek Harness
sinh **cả xuvi-harness lẫn Waldorf tích hợp**; tests đạt và qua cổng dogfood
thì chuyển hẳn sang xuvi-harness, không dùng DeepSeek Harness nữa.

Theo đó, spec `11_macos-native-swift-va-tich-hop-harness.md` **thay thế** thứ
tự trong mục 2 và điều "không xây harness generic" trong mục 3 dưới đây cho
đợt này. Mốc của đợt ở mục 1 và kịch bản demo ở mục 4 giữ nguyên. Các mục còn
lại của file này giữ vai trò lịch sử/bối cảnh cho tới khi spec 11 được thực
thi và đo lại.

## 3. Điều cố tình không làm trong đợt này

- **Không xây lại harness generic** (`harness/specs/core/*`). Harness trong đợt
  này là *agent đang chạy loop* (kèm specs của nó), không phải một sản phẩm code
  thứ hai. Chỉ sửa harness specs khi một điểm nghẽn của Waldorf/benchmark thật
  sự đòi sửa — và ghi rõ vết đó trong kết quả loop.
- **Không điền lại toàn bộ `harness/specs/technology/quyet-dinh-hien-tai.md`**
  như một dự án riêng. Stack của sản phẩm Waldorf đã chốt trong
  `03_research_and_architecture.md` (React/TypeScript + Node + SQLite, local,
  loopback) là quyết định công nghệ *của plugin này*; quyết định của harness
  để dành cho đợt mà điểm nghẽn thật sự nằm ở harness.
- **Không mở rộng phạm vi** sang plugin thứ hai, deploy/hosting, tài khoản
  người dùng, hay dữ liệu học sinh thật. Demo bằng dữ liệu minh họa hư cấu,
  gắn nhãn minh họa.
- **Không theo đuổi đủ tên sản phẩm Meta.** Giữ lựa chọn có lý do trong `03`:
  dùng Wonder/Spark/Image/Voice Transcribe ở nơi chúng phục vụ hành trình;
  SAM, Spark Open, Glimmer, Baseten/Modal/Fireworks chỉ đưa vào nếu một điểm
  nghẽn benchmark cụ thể đòi chúng — kèm lý do mới ghi vào specs.

## 4. Kịch bản demo mục tiêu (dùng để đo B1/B3)

1. Mở sản phẩm local, thấy tổng quan hôm nay và truyện đang luyện.
2. Chọn một truyện trong thư viện (hoặc tải lên một truyện ngắn công khai),
   xem văn bản đã duyệt, độ tuổi và mục tiêu luyện.
3. Xem/sửa bản đồ cảnh; một cảnh có tranh (AI thật hoặc ảnh tải lên — nhãn rõ).
4. Luyện kể một lượt: thu âm hoặc tải âm thanh, sửa transcript, nhận góp ý có
   trích dẫn, phân biệt bỏ sót với chuyển thể có chủ ý, không có điểm tổng.
5. Mở một hồ sơ (ví dụ lịch tuần hoặc chương trình theo lớp) được sinh từ cùng
   dữ liệu, xem trước và in/xuất thử.
6. Toàn bộ kịch bản không cần người xây đứng cạnh, không có nút chết, và mọi
   chỗ AI chưa cấu hình đều nói thật là chưa cấu hình thay vì giả thành công.

## 5. Ngân sách token của đợt

Theo phân bổ trong `cron.md` mục 6. Token còn nhiều là *trần ngân sách*, không
phải nghĩa vụ phải tiêu hết: đạt mốc ở loop 2 thì loop 3 là lựa chọn, và phần
tiết kiệm được ghi vào kết quả cuối đợt như một kết quả tốt. Điều bị cấm là
đốt token vào việc sinh thêm tính năng sau khi đã cắt ngân sách của đo và ghi
trí nhớ — hai pha đó là phần làm đợt này thành tiến hoá thay vì một lần build.
