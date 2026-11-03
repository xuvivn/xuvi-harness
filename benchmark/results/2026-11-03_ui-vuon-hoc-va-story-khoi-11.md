# Giao diện "Vườn học" (Spec 12) và kiểm thử XuVi Story với truyện Khối 11 Tiền tệ — 2026-11-03

## Phạm vi và quyết định

Hoàn tất phiên làm việc dang dở gồm hai phần gắn kết trong một điểm nghẽn B3/B1:
1. **Làm mới UI/UX macOS native "Vườn học" (`plugins/waldorf/specs/12_ui-vuon-hoc-native.md`)**: đưa hành trình chuẩn bị bài kể của giáo viên (`Tổng quan → Luyện kể → Góp ý`) lên vị trí mặc định thay vì rơi thẳng vào màn chat kỹ thuật; áp dụng bảng màu ấm (`#FDF8EE` nền giấy, `#1E4028` xanh lá thẫm sidebar, `#2F5D3A` nút chính, `#C45C4A` đất nung), chữ serif cho tiêu đề và truyện đọc; ẩn log uỷ quyền kỹ thuật sau `DisclosureGroup("Chi tiết kỹ thuật")`; giữ ranh giới thành thật (`DemoBanner`, nhãn `Phân tích cục bộ — không phải AI`, chỉ có nút `Sao chép Markdown`, không giả xuất PDF/DOCX).
2. **Kiểm thử XuVi Story bằng câu chuyện của block kế tiếp sau kỳ nghỉ Thu (02–15/11/2026)**: đối chiếu lịch năm học Lớp 3 (2026–2027), block ngay sau nghỉ Thu là **Khối 11: Tiền tệ (3 tuần, 16/11–06/12/2026)** cùng mạch truyện Cựu Ước xuyên Khối 10–11 (*Hành trình về miền đất hứa* — Jakob Streit). Bổ sung hai truyện minh họa không chứa dữ liệu học sinh thật (`"Vỏ sò và đồng xu bên bến Hội An (Khối 11 — Tiền tệ)"` làm truyện mặc định khi mở app, và `"Abraham và bầu trời sao (Mạch Khối 10–11)"`) cùng 3 chế độ luyện kể (`Gợi ý đầy đủ` / `Mờ dần` / `Tự kể`) lưu riêng theo từng truyện.

Điều phối thực thi qua `muse exec` theo phân vai của hệ:
- `muse-wonder-1.0` (`app/ui-refresh/wonder-run.log`, `wonder-block11.log`, `wonder-layout-fix.log`): sinh bộ giao diện 6 màn hình, tích hợp 2 truyện Khối 11 + 3 chế độ luyện kể, và sửa lỗi bố cục ở kích thước `900×600` (bỏ `.windowResizability(.contentMinSize)` và `.fixedSize(horizontal: false, vertical: true)` ngoài `ScrollView` từng làm `NavigationSplitView` bị cắt đầu/đuôi khi vào trang `Trò chuyện`).
- `muse-spark-1.3 --contributor-mode on` (`app/ui-refresh/spark-review.log`, `spark-block11.log`): rà soát độc lập spec/code và viết 26 kiểm thử tự động mới cho dữ liệu truyện Khối 11, tách 4 cảnh, phân tích cục bộ `covered`/`omissions` và bảo toàn snapshot.

**Phán quyết: GIỮ thay đổi phạm vi B3/B1 (giao diện "Vườn học" + dữ liệu & luồng luyện kể Khối 11 Tiền tệ); ĐỖ kết luận benchmark toàn hệ.** Không tự chấm lại tổng điểm `/100` toàn hệ vì các chiều chưa chạy lại trong lượt này (`B6` live Meta API, `B7` radar tự học) giữ nguyên trạng thái `CHƯA ĐO`.

## Trước/sau trên bộ tiêu chí B3 (Spec 12 §6) và XuVi Story Khối 11

| Tiêu chí | Trước (`before-*.png`) | Sau (`after-*.png`) | Mức bằng chứng & giới hạn |
|---|---|---|---|
| 1. Ngữ cảnh mặc định khi mở app | Rơi thẳng vào màn Chat kỹ thuật, lộ tên model và dòng uỷ quyền | Mở vào `Tổng quan` ("Vườn học hôm nay") với thẻ `Truyện đang luyện: Vỏ sò và đồng xu bên bến Hội An (Khối 11 — Tiền tệ)` và CTA `Bắt đầu luyện kể` | `thủ công` (`after-dashboard.png` ở `900×600`) |
| 2. Độ tương phản & phân cấp chữ | Nền xám hệ thống, chữ đều nhau, thẻ thiếu phân cấp | Nền giấy `#FDF8EE`, sidebar `#1E4028`, chữ chính `#2B3A2E`, chữ phụ `#51624F`, tiêu đề và truyện gốc dùng font serif có khoảng cách dòng | `thủ công` (đối chiếu 8 ảnh `after-*.png`) |
| 3. Luồng `Luyện kể` có dẫn hướng & 3 chế độ luyện | Một cột phẳng, không có chế độ ẩn gợi ý, đổi truyện giữ nguyên `transcript` cũ dễ gán nhầm | Thanh 3 bước (`1 Đọc truyện & cảnh → 2 Chọn chế độ & viết bản kể → 3 Xem góp ý`), `Picker` 3 chế độ (`Gợi ý đầy đủ` / `Mờ dần` / `Tự kể`) có nút thoát `Xem gợi ý`, tách biệt `draftsByStory` / `modesByStory` / `feedbackByStory`, chắn bản kể rỗng | `thủ công` (`after-story.png`, `after-story-scenes.png`, `after-story-writing.png`) + `tự động` (`xuvi-tests`) |
| 4. Thử nghiệm truyện Khối 11 Tiền tệ (sau nghỉ Thu) | Chỉ có 1 truyện hạt giống lớp 2 minh họa | 3 truyện trong vườn: (1) *Vỏ sò và đồng xu bên bến Hội An (Khối 11 — Tiền tệ)* (4 cảnh: đổi hàng → ghế gỗ 3 ngày công → vỏ sò/đồng xu làm tin → phiên chợ lớp học & món quà không mua được bằng tiền), (2) *Abraham và bầu trời sao (Mạch Khối 10–11)* (4 cảnh), (3) *Hạt giống của Mây (minh họa)* | `tự động` (26 checks `waldorf-block11` trong `xuvi-tests`) + `thủ công` trên UI |
| 5. Phân biệt tiết `Học` và `Sinh hoạt` trên `Lịch tuần` | Bảng phẳng khó quét | Nhãn chữ + màu tách biệt (`● Học` xanh lá, `● Sinh hoạt` màu cát), cột giờ `08:00 – 10:00` (`120 phút`), banner kiểm tra `2.100 / 2.100 phút` | `thủ công` (`after-schedule.png`) + `tự động` |
| 6. Xem trước `Hồ sơ` & hành động thật | Dump chuỗi Markdown thô trong khối моно | `MarkdownPreview` phân cấp tiêu đề/mục/bảng, nút duy nhất `Sao chép Markdown` kèm phản hồi `Đã sao chép`, ghi rõ chưa có xuất PDF/DOCX | `thủ công` (`after-documents.png`) + `tự động` (6 loại hồ sơ đủ mục bắt buộc) |
| 7. Tách chi tiết kỹ thuật khỏi luồng chính | Dòng `Uỷ quyền gần nhất...` luôn lộ trên màn chính | Thu gọn vào `DisclosureGroup("Chi tiết kỹ thuật")` mặc định đóng ở đáy `Trò chuyện` và thẻ riêng cuối `Cài đặt` | `thủ công` (`after-chat.png`, `after-settings.png`) |
| 8. Trung thực về trạng thái & dữ liệu | Chưa có dải báo chế độ xem trước và giới hạn lưu | `DemoBanner` ở đầu mọi trang; nhãn `Phân tích cục bộ — không phải AI` ở thẻ góp ý; chế độ `--ui-preview` (`XuViUIPreview`) không đọc Keychain/`.env`/env và khoá nút Lưu/Xóa trong `Cài đặt` | `thủ công` + `tự động` (`xuvi-tests`) |
| 9. Kiểm tra bố cục thực tế ở `900×600` và `1280×880` | Khi chuyển sang `Trò chuyện` ở `900×600`, `.windowResizability(.contentMinSize)` kết hợp `.fixedSize` ngoài `ScrollView` làm `NavigationSplitView` bị cắt đầu/đuôi (phát hiện khi chụp UI thật) | Đã gỡ `.windowResizability(.contentMinSize)`, bỏ `ScrollView` thừa ở `SidebarView` và `.fixedSize` ngoài `ScrollView` ở `composerBar`; cả 6 màn hình hiển thị trọn vẹn ở `900×600` và `1280×880` | `thủ công` (đối chiếu ảnh chụp cửa sổ thật trước và sau khi sửa lỗi bố cục) |

## Lệnh thực chạy và kết quả kiểm chứng

- `swift run --package-path app xuvi-tests`: trước **54 passed, 0 failed**; sau **80 passed, 0 failed** (mức `tự động`). Giữ nguyên toàn bộ 54 kiểm thử cũ, bổ sung 26 kiểm thử `waldorf-block11` kiểm tra:
  - Cấu trúc 3 truyện seed, tiêu đề và nhãn nguồn gốc Khối 11 Tiền tệ (16/11–06/12/2026, sau nghỉ Thu), không chứa PII hay tên học sinh thật.
  - `StorySplitter.split` tách đúng 4 cảnh cho cả 3 truyện.
  - `LocalFeedbackAnalyzer.analyze` với bản kể một phần (chỉ nhắc cảnh 1–2 của truyện Tiền tệ: lúa, gốm Thanh Hà, rổ tre, bác thợ mộc Kim Bồng đóng ghế gỗ ba ngày công) trả về đúng `covered.count == 2` (cảnh 1–2) và `omissions.count == 2` (cảnh 3–4: vỏ sò/đồng xu Hội An và phiên chợ lớp học), nhãn `"Phân tích cục bộ — không phải AI"`, JSON encode không chứa `overallScore`.
  - Bản kể đầy đủ 4 cảnh trả về `covered.count == 4`, `omissions.isEmpty == true`.
  - `PracticeAttempt` lưu đúng `mode` (`"fading"`) và đóng băng `storySnapshot` khi truyện gốc bị sửa sau đó.
- `swift build --package-path app -c release --product XuViApp` và `python3 app/ui-refresh/package-ui.py`: build release đạt, đóng gói và ký ad-hoc hai bundle `app/dist/XuVi-Garden.app` và `app/dist/XuVi-Garden-Preview.app` (`codesign --verify --strict` exit 0).
- Mở trực tiếp `app/dist/XuVi-Garden-Preview.app` bằng `open -n` với từng màn (`dashboard`, `story`, `story-fading`, `story-feedback`, `timetable`, `documents`, `chat`, `settings`) và chụp cửa sổ thật (`screencapture -l <windowId>`) tại `app/ui-refresh/after-*.png` (mức `thủ công`).

## Bằng chứng toàn thể và trạng thái các cổng (theo `01-benchmark-tong-the.md`)

| Mục | Trạng thái lần này |
|---|---|
| G1 (Sự sống & trẻ em) | **ĐẠT** trong phạm vi thay đổi: truyện Khối 11 Tiền tệ nuôi dưỡng lòng biết ơn công sức lao động, sự minh bạch và những giá trị không mua được bằng tiền; góp ý luyện kể khích lệ, không chấm điểm tổng |
| G2 (Trung thực) | **ĐẠT** trong phạm vi thay đổi: nhãn nguồn gốc truyện ghi rõ là chuyện dựng minh họa theo khung chương trình, không tuyên bố đạt chuẩn GDPT/Waldorf chính thức; phân tích cục bộ ghi rõ không phải AI; các chiều chưa đo bên dưới giữ nguyên `CHƯA ĐO` |
| G3 (Riêng tư) | **ĐẠT**: chỉ đọc kế hoạch khối học và mạch truyện từ `minhxuvi`, không trích bất kỳ dữ liệu học sinh thật hay bí mật nào (được kiểm tra tự động trong `waldorf-block11`); bản `XuVi-Garden-Preview.app` cô lập hoàn toàn khỏi Keychain và `.env` |
| B1 (Giá trị sản phẩm Waldorf) | Cải thiện rõ ở luồng `XuVi Story` (2 truyện Khối 10–11 mới + 3 chế độ `prompt`/`fading`/`free` + tách bản nháp theo truyện); mức `tự động` (80/80 checks) + `thủ công` trên app |
| B2 (Phù hợp triết lý giáo dục) | Truyện Khối 11 bám sát tiến trình Lớp 3 (từ trao đổi vật phẩm sang vật làm tin chung rồi phiên chợ lớp học) và giữ nhịp kể giàu hình ảnh; mức `thủ công` + `tự động` |
| B3 (UI/UX & khả năng tiếp cận) | Hoàn thành 8/8 mục kiểm tra của Spec 12 §6 ở cả `900×600` và `1280×880`, bắt và sửa được lỗi cắt khung `ChatView`/`SidebarView` nhờ chạy và chụp cửa sổ thật; mức `thủ công` |
| B4 (Kiến trúc harness) | Không đổi core; việc sinh code và kiểm thử trong loop được điều phối qua `muse-wonder-1.0` và `muse-spark-1.3-contributor` |
| B5 (Độ tin cậy kỹ thuật) | `80/80` checks trong `xuvi-tests` PASS (`tự động`), release build + ad-hoc codesign PASS |
| B6 (Thực thi model Meta) | `CHƯA ĐO` lại trên live API của app trong lượt này (chế độ preview tắt mạng theo thiết kế; CLI `muse` dùng model Meta trực tiếp trong quá trình điều phối) |
| B7 (Radar nghiên cứu) | Cập nhật `research/specs/resources.md` với mẫu tham chiếu câu chuyện Khối 11 Tiền tệ Lớp 3; không chấm điểm lại toàn mục |
| B8 (Kỷ luật tiến hoá) | **ĐẠT**: cập nhật specs (`plugins/waldorf/specs/12_ui-vuon-hoc-native.md`, `research/specs/resources.md`) trước khi sinh code, lưu kết quả đo mới, cập nhật `INDEX.md`, không chạm `ĐẠO.md`, không để file sinh ra lọt vào `git status` |
| Tổng (/100) | `—` (đo theo phạm vi B3/B1; không nội suy thành điểm tổng 100 khi B6 live app chưa đo lại trong lượt này) |
