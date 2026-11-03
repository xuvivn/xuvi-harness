# Chẩn đoán sau bootstrap — 2026-11-03

- **Thời điểm:** 2026-11-03, Asia/Saigon.
- **Phạm vi:** nghiên cứu repo và đề xuất bước tiếp theo theo yêu cầu người dùng;
  không thực hiện loop thay đổi sản phẩm, không sửa specs hay code.
- **Người điều phối:** Codex. Phản biện qua Muse CLI: `muse-wonder-1.0`
  và `muse-spark-1.3-contributor`, hai phiên riêng, đều kết thúc exit 0.
- **Mốc đối chiếu:** `2026-11-03_bootstrap-xuvi-native-swift.md` ghi 62.5/100.
  Đây là điểm lịch sử, **không phải điểm được xác nhận lại trong lần này**.
- **Phán quyết chẩn đoán:** **ĐỖ kết luận đã chuyển nhà**. Bằng chứng được ghi
  trong báo cáo bootstrap và code hiện tại chưa thoả spec 11 §5. Không kết luận
  toàn bộ công sức bootstrap vô hiệu, không sửa file kết quả cũ.

## 1. Dự án đang ở đâu?

XuVi là hệ agent tự tiến hoá qua specs, benchmark và các cổng của Đạo;
Waldorf là plugin sản phẩm đầu tiên. Hướng hiện hành là macOS native Swift,
Meta làm provider, Wonder điều phối và Spark Contributor làm worker.
Spec Waldorf 11 supersede hướng React/Node/SQLite và phạm vi bootstrap cũ.

Code sinh ra có thật trong `app/`, gồm core, adapter, domain Waldorf, CLI,
SwiftUI và test runner. App và logic nền đã build được; khoảng cách còn lại
không chỉ là thiếu tài liệu mà nằm ở khả năng tự vận hành và nối hành trình UI.

## 2. Bằng chứng thực chạy trong lần này

| Kiểm tra | Kết quả | Mức và giới hạn |
|---|---|---|
| `swift run --package-path app xuvi-tests` | exit 0; **44 passed, 0 failed** | `tự động`; bộ custom assertion runner; các ca provider dùng replay/mock, không chứng minh AI live |
| `swift build --package-path app --product XuViApp` | exit 0 | `tự động`; build debug, không phải nghiệm thu giao diện hoặc bản release |
| Đọc code SwiftUI, CLI, store và spec 11 | Các khoảng cách bên dưới | Rà tĩnh thủ công; không phải mức `thủ công` theo nghĩa thao tác sản phẩm của benchmark |
| Hai phản biện Meta qua Muse CLI | Có trả lời ở cả hai model | `live` cho việc tham vấn qua Muse, **không phải** live qua adapter XuVi; không nâng B6 của sản phẩm |
| `git diff --quiet -- ĐẠO.md` | exit 0 | Đạo không bị sửa |

Build/test có warning về API lock/enumeration trong async sẽ thành lỗi ở Swift 6
language mode và đường dẫn framework/library của Command Line Tools. Không gọi
đây là build sạch warning. Không sửa vì ngoài phạm vi chẩn đoán.

Không chạy UI, không gọi live adapter XuVi, không mở nguồn MOET, không kiểm lại
toàn bộ G1/G3. Không đọc `.env`, Keychain, repo khác hoặc dữ liệu học sinh thật.
Prompt gửi Meta chỉ gồm các specs/báo cáo/code liên quan đã đọc; không gửi toàn
repo. Muse dùng preset `empty`, tắt shell/write/web/reminders/user-input,
không nạp foreign personal context, không lưu session log, tối đa một model step.

## 3. Sai lệch quyết định: smoke test chưa phải cổng chuyển nhà

Đối chiếu `plugins/waldorf/specs/11_macos-native-swift-va-tich-hop-harness.md`
§5 với `app/Sources/XuviCLI/main.swift`:

| Điều kiện §5 | Điều hiện có | Kết luận trong phạm vi rà soát |
|---|---|---|
| XuVi tự đọc Đạo/README/cron/specs, lập todo và giả thuyết | `prepareDogfoodWorkspace()` chỉ tạo `notes.txt` chứa 17 + 25; prompt hard-code bài toán | Chưa được chứng minh |
| Việc tiến hoá thật: uỷ quyền, kiểm chứng, tích hợp, tự chạy test | Live composition chỉ có read/write/delegate; không đăng ký shell để chạy test; pass khi `result.txt == 42` | Chứng minh một lát cắt uỷ quyền, chưa chứng minh loop tiến hoá |
| Đúng host/model/vai, log cha/con tái dựng được | Báo cáo cũ khai đã kiểm; lần này không chạy lại hay đọc log live cũ | Chưa kiểm chứng trong lần này |
| Orchestrator XuVi tự soạn nháp benchmark | Báo cáo bootstrap ghi nháp do phiên bootstrap soạn, đẩy yêu cầu này sang loop sau | Không thoả chính điều kiện đã đặt |

Báo cáo cũ vẫn kết luận “ĐẠT” cả cổng và chuyển nhà. Đó là kết luận vượt quá
bằng chứng được trình bày. Phải tách **cơ chế uỷ quyền chạy được** khỏi **hệ tự
thực hiện một loop tiến hoá được**. Điểm tổng 62.5 và 44 checks không thay thế §5.

Đồng thời, spec còn yêu cầu `swift test` trong khi package dùng executable
`xuvi-tests`, không khai `testTarget`. Cách chạy hiện tại có bằng chứng tự động
thật; cần ghi đúng contract kiểm thử trong specs **trước lần nghiệm thu tới**,
giữ các hành vi bắt buộc, không đổi nghĩa benchmark để hợp thức kết quả cũ.

## 4. Khoảng cách sản phẩm đã xác định từ code

1. **Lưu bền UI chưa nối:** `AppState.waldorf` khởi tạo seed trong bộ nhớ;
   `analyzeLocal()` chỉ append lượt luyện. Không gọi `WaldorfStore` từ UI.
   Test store sống qua restart không chứng minh dữ liệu giao diện sống qua restart.
2. **Lịch sử chưa xem lại được:** có snapshot trong model và bộ đếm lượt luyện,
   chưa có màn hình mở lại lượt cũ. Không đánh đồng bộ đếm với hành trình lịch sử.
3. **Story còn thiếu tương tác:** UI hiện hiển thị cảnh và nhập transcript;
   chưa nối ghi âm/tải audio/phiên âm, chỉnh truyện/cảnh, ba chế độ luyện.
4. **Xuất/in chưa có handler trong code UI đã đọc:** có preview Markdown và
   dòng hướng dẫn File → Print; chưa thấy hành động export hay print nối vào hồ sơ.
   Chức năng in qua giao diện hệ thống chưa được thao tác kiểm chứng.
5. **Log chat UI chỉ ở bộ nhớ:** `SessionLog()` không file URL và child log URL
   trả `nil`; không dùng log CLI để suy ra khả năng truy vết bền của chat UI.

Không chấm lại B1/B3 từ rà tĩnh. Không tuyên bố UI đạt hoặc không đạt qua thao tác
khi chưa mở ứng dụng. Spark còn nêu thiếu disclosure/confirmRemote trong chat;
điểm này cần kiểm riêng theo contract gửi dữ liệu trước khi dùng nội dung thực,
không suy diễn rằng đã có rò rỉ dữ liệu trong phiên này.

## 5. Cổng và tám chiều của lần chẩn đoán

| Cổng | Trạng thái | Phạm vi |
|---|---|---|
| G1 | CHƯA ĐO | Chưa nghiệm thu lại toàn sản phẩm |
| G2 | CHƯA ĐẠT đối với kết luận chuyển nhà cũ | Kết luận vượt bằng chứng §5; bản này ghi đính chính phạm vi, không cáo buộc có chủ ý làm giả |
| G3 | CHƯA ĐO toàn sản phẩm | Phiên nghiên cứu tuân ranh giới dữ liệu; chưa đủ để xác nhận sản phẩm |

| Chiều | Kết quả trong lần này |
|---|---|
| B1 | Rà code thấy hành trình UI chưa khép kín; chưa chấm lại |
| B2 | Không rà nguồn/chất lượng giáo dục lại; chưa kiểm chứng |
| B3 | Không thao tác UI; chưa kiểm chứng |
| B4 | Một phần hành vi core/replay được test lại; chưa nghiệm thu toàn bộ bất biến |
| B5 | Build debug và 44 checks đạt; chưa kiểm mọi đường lỗi và UI persistence |
| B6 | Không live qua XuVi trong lần này; chưa kiểm chứng |
| B7 | Có phản biện và quyết định từ nguồn nội bộ; chưa quét research bên ngoài |
| B8 | Đã phát hiện sai lệch, ghi bản mới và dòng INDEX; chưa chứng minh loop tự chủ |
| Tổng/delta | **Không tính** — đây là chẩn đoán có kiểm tra một phần, không phải lần đo tổng thể |

## 6. Bước tiếp theo được chọn

**Một điểm nghẽn chính: cổng chứng minh XuVi tự thực hiện loop tiến hoá chưa đạt.**

Giả thuyết cho đợt tới: nếu hoàn thiện đường chạy `evolution` tối thiểu từ specs,
với tools bị giới hạn và bằng chứng kiểm được, thì chính Wonder của XuVi có thể
đọc specs, giao Spark một việc thật, kiểm kết quả, tự chạy test và tự viết nháp
benchmark trong workspace dùng-một-lần; khi đó mới có căn cứ xét chuyển nhà.

Thứ tự thực hiện:

1. **Specs trước:** làm rõ entrypoint/profile `evolution`, contract custom test
   runner, phạm vi tools, timeout/hủy và thu thập bằng chứng cho đủ bốn khoản §5.
   Giữ nguyên độ chặt của cổng; không biến bài toán 42 thành tiêu chí tự chủ.
2. **Sinh phần tối thiểu:** nối composition/CLI thực thi contract trên. Chỉ mở
   năng lực cần cho nhiệm vụ; bản sao dùng-một-lần chỉ chứa file trong allowlist,
   không chứa `.env`, dữ liệu thật hay symlink dẫn ra vùng riêng tư. Kiểm ranh giới
   trước khi cho agent thực thi shell; lỗi sandbox phải dừng, không fallback rộng.
3. **Dogfood thật:** chính XuVi đọc tài liệu, nêu một giả thuyết nhỏ, uỷ quyền
   Spark đúng bốn thành phần, cha kiểm một tham chiếu, tích hợp, tự gọi test runner
   và tự viết nháp kết quả. Người điều phối kiểm lại artefakt/log, không thay phần
   việc mà spec yêu cầu XuVi tự làm rồi tính là đạt.
4. **Nghiệm thu:** đo trước/sau cùng tiêu chí; kiểm đủ bốn khoản §5, G1–G3,
   mọi chiều B1–B8 có bằng chứng hoặc nhãn chưa kiểm chứng. Chỉ giữ khi thoả
   cron.md; thiếu điều kiện thì đỗ, lưu file mới và INDEX. Không nâng điểm từ
   chính lời tự nhận của model.

Ngân sách đề xuất: một loop trước, checkpoint sau specs và kiểm ranh giới;
timebox 60–90 phút là ước lượng lập kế hoạch, không phải cam kết hoàn thành.
Nếu hết ngân sách hoặc bằng chứng chưa đủ, dừng và ghi điều kiện thiếu.

Sau khi cổng trên qua, **ưu tiên sản phẩm kế tiếp là lưu bền lượt luyện và mở lại
lịch sử trong UI**, kiểm đóng/mở và lỗi ghi bằng dữ liệu minh họa. Không gộp thêm
audio, tranh, DOCX, đa nền tảng hay nâng cấp kiến trúc tổng quát vào loop đó.

## 7. Vì sao chọn như vậy theo Đạo?

- Sự sống: tránh mất công sức của người dùng; lưu bền UI là ưu tiên sản phẩm
  ngay sau việc khôi phục cơ sở vận hành đáng tin.
- Trung thực: cổng bằng chứng có quyền veto; không xây tiếp dựa trên tuyên bố
  tự chủ chưa đạt, không phủ nhận bằng chứng hẹp đã có.
- Riêng tư: nghiên cứu chỉ dùng đầu vào đã chọn; chạy tự chủ trong bản sao sạch,
  không cấp toàn repo có bí mật cho agent.
- Học tiếp: tiêu chí rõ và báo cáo mới giúp người sau phân biệt smoke test,
  dogfood thật và nghiệm thu sản phẩm; giữ đường truy vết qua specs.

Wonder đề nghị đóng cổng dogfood trước. Spark đề nghị lát cắt lưu bền UI;
đề xuất của Spark kèm xuất/in sẽ rộng hơn một điểm nghẽn nên không nhận nguyên
gói. Người điều phối quyết định thứ tự trên sau khi đối chiếu code/test thực tế.
Ý kiến model là phản biện, không thay bằng chứng thực thi.

## 8. Vệ sinh và giới hạn

- Không sửa Đạo, specs, code hay kết quả benchmark cũ; không commit/push/deploy.
- Chỉ thêm bản chẩn đoán này và nối một dòng INDEX, giữ các thay đổi có sẵn.
- Code/build/prompt và output Muse là artefakt sinh, không đưa vào diện commit.
- Không có quyết định adopt model mới; không cần quét nguồn bên ngoài để kết
  luận sai lệch nội bộ này. Không dùng ý kiến Meta như tài liệu chính thức.
- Dừng ở nghiên cứu/chẩn đoán đúng phạm vi yêu cầu; chưa thực hiện đợt tiến hoá
  được đề xuất và chưa tuyên bố cổng G2 của toàn sản phẩm đã được sửa xong.
