# 12 — Vườn học: giao diện native dễ bắt đầu

Ngày 2026-11-03. Yêu cầu trực tiếp: cải thiện UI/UX, tham khảo `xuvi-waldorf`.
Đứng dưới Đạo và spec 11; không thay stack SwiftUI/AppKit hoặc core/provider.

## Điểm nghẽn và giả thuyết

B3: app mở vào thông tin kỹ thuật agent, không có điểm bắt đầu theo việc của
phụ huynh/giáo viên; truyện, lịch và hồ sơ trình bày như bản debug.
Nếu mở bằng một vườn học có hành động rõ và áp dụng cùng một hệ thị giác cho
các màn hình, người mới tìm được hành trình luyện kể mà không cần giải thích.
Một loop chỉ giải quyết điểm nghẽn này; không bổ sung nghiệp vụ giáo dục mới.

Tham khảo có giới hạn: mã UI `client/src/App.tsx`, `client/src/styles.css` của
`xuvi-waldorf` được người dùng cho phép đọc. Học màu giấy/xanh rừng, phân cấp
serif, điều hướng theo việc và trang tổng quan. Không đọc dữ liệu, secrets,
media hoặc mang tuyên bố hoàn thành/nguồn MOET từ repo đó sang đây.

Bối cảnh thử XuVi Story theo yêu cầu trực tiếp của người dùng: lấy kế hoạch
block học kế tiếp sau kỳ nghỉ Thu (02–15/11/2026) từ `minhxuvi` — **Khối 11:
Tiền tệ (3 tuần, 16/11–06/12/2026, Lớp 3, 8–9 tuổi)** — để dựng truyện thử
nghiệm ngay trên giao diện mới, gồm:
- Truyện bài học chính Khối 11 (*"Vỏ sò và đồng xu bên bến Hội An — Khối 11 Tiền tệ"*):
  từ chia sẻ/đổi hàng trực tiếp (nông sản, gốm, đan tre) → tình huống lệch giá
  trị khi đổi đồ tốn nhiều ngày công → dùng vỏ sò rồi đồng xu làm vật trao đổi
  chung dựa trên niềm tin → phiên chợ lớp học và những giá trị (lòng tin, tình
  bạn, món quà tự tay làm) không đồng xu nào mua được.
- Truyện mạch đọc/kể xuyên Khối 10–11 (*"Abraham và bầu trời sao — Hành trình về miền đất hứa"*):
  rời thành Ur lên đường, nhường đất chăn cừu cho Lot để giữ hòa khí, giữ niềm
  tin qua thử thách.
- Giữ thêm truyện mẫu *"Hạt giống của Mây (minh họa)"* để kiểm chuyển đổi qua
  lại nhiều truyện. Tuyệt đối không đưa dữ liệu học sinh thật hay thông tin cá
  nhân từ `minhxuvi` vào seed, specs hay báo cáo.

## Hệ thị giác và điều hướng

- Nền giấy #FDF8EE, thẻ #FFFDF7, mực #2B3A2E, xanh rừng #2F5D3A,
  xanh đậm #1E4028, đường viền #E7DCC4, đất nung chỉ làm điểm nhấn.
  Chữ phụ phải đủ đậm trên nền giấy. Bảng màu giấy chủ đích dùng light scheme
  nhất quán, kể cả khi OS dark; controls native vẫn rõ và đủ tương phản.
- Tiêu đề serif 28–34 pt; nội dung hệ thống 14–16 pt, khoảng cách 8/12/16/24/32,
  thẻ bo góc 14–18. SF Symbols có nhãn; không emoji thay icon chính.
- Sidebar xanh đậm 220–240 pt, thương hiệu XuVi, nhãn Waldorf-inspired,
  chia nhóm “Vườn học” (Tổng quan, Luyện kể, Lịch tuần, Hồ sơ) và
  “Trợ giúp” (Trò chuyện, Cài đặt). Mục đang chọn rõ, có tên truy cập.
- Mở mặc định Tổng quan. Các CTA điều hướng phải đổi cả nội dung lẫn selection.
  Nội dung co giãn, cuộn dọc; cửa sổ tối thiểu 900×600, kiểm ở 900×600 và
  khoảng 1280×800. Bố cục nhiều cột thu về một khi thiếu chỗ; không cắt nút.
- Mỗi trang có tiêu đề, mô tả ngắn, hành động chính. Dữ liệu minh họa hư cấu
  phải nhận biết ngay ở shell; không hứa đã lưu bền khi chỉ lưu trong phiên.

## Các màn hình

1. **Tổng quan — Vườn học hôm nay:** lời dẫn ngắn; thẻ truyện đang luyện với CTA
   “Bắt đầu luyện kể”; thẻ nhịp tuần với CTA mở lịch; số liệu lấy từ state thật
   (truyện, lượt luyện trong phiên, loại hồ sơ). Hình lá trang trí bằng SwiftUI
   hoặc SF Symbols, không cần ảnh ngoài. Không bịa “tiết tiếp theo” theo giờ hiện
   tại: chỉ gọi là một tiết trong lịch minh họa nếu chưa tính thời gian thật.
2. **Luyện kể:** chọn truyện (mặc định mở vào truyện Khối 11 Tiền tệ sau kỳ nghỉ
   Thu để thử ngay, kèm truyện mạch *Hành trình về miền đất hứa* và truyện mẫu);
   tiêu đề, tuổi, nguồn; bước 1 đọc truyện/bản đồ cảnh, bước 2 chọn chế độ luyện
   (`prompt`: hiện đầy đủ tóm tắt + từ khóa; `fading`: mờ dần chỉ giữ tiêu đề +
   từ khóa; `free`: tự kể ẩn thẻ cảnh) và viết bản kể, bước 3 xem góp ý. Cảnh
   thành thẻ đánh số, có tóm tắt và từ khóa; nội dung nguồn đọc được. TextEditor
   có label ngoài, placeholder hỗ trợ; không phân tích bản kể rỗng. Góp ý vẫn
   ghi “phân tích cục bộ — không phải AI”, không điểm tổng. Đổi truyện phải
   tránh gán transcript/góp ý truyện cũ sang truyện mới (giữ draft và chế độ
   luyện theo truyện hoặc reset có thông báo rõ).
3. **Lịch tuần:** nhóm thứ 2–6, khoảng giờ bắt đầu–kết thúc, tên, giáo viên,
   mục tiêu; phân biệt Học/Sinh hoạt bằng chữ và màu. Cửa sổ hẹp vẫn đọc được.
4. **Hồ sơ:** sáu lựa chọn dễ hiểu, xem trước văn bản có phân cấp; copy Markdown
   bằng nút thật và phản hồi đã sao chép. Không tuyên bố có Print/PDF/DOCX nếu
   chưa có action thực thi. Nêu dữ liệu minh họa, không chứng nhận chính thức.
5. **Trò chuyện:** lời chào theo công việc, gợi ý bắt đầu, phân biệt Bạn/XuVi,
   composer rõ, trạng thái đang chạy/không cấu hình. Thông tin model/ủy quyền
   nằm trong vùng chi tiết gập mặc định. Ngay cạnh Gửi nêu nội dung sẽ được gửi
   đến Meta; không tự gửi khi vào trang. Gợi ý không ngụ ý trợ lý đã đọc truyện
   hoặc lịch trong app: hiện chỉ gửi đoạn người dùng nhập. Tin nhắn mới phải
   cuộn tới được trong lịch sử. Không đổi agent/core.
6. **Cài đặt:** giữ SecureField, Lưu/Xóa và contract spec cấu hình Meta; tách
   kết nối, dữ liệu minh họa và thông tin kỹ thuật; không hiển thị giá trị khóa.

## Kiểm tra cô lập

App hỗ trợ `--ui-preview`: chỉ dùng seed hư cấu và SecretProvider không store,
không envFile, environmentKey trả nil. Không đọc Keychain/.env hoặc gọi Meta
trong chế độ này. Các hành động khóa bị vô hiệu và có giải thích. Bản QA có
bundle ID riêng, không đóng/ghi đè app người dùng. Không đổi chế độ bình thường.
Bundle QA có thể bật cùng chế độ qua boolean `XuViUIPreview` trong Info.plist,
để mở qua Finder/CUA bằng cùng executable mà không cần truyền cờ Terminal.

## Checklist cố định trước–sau (B3)

1. Mở có tổng quan theo việc + CTA luyện kể, không bắt đầu bằng tên model.
2. Điều hướng có selection, icon có nhãn; CTA vào đúng trang.
3. Nền giấy/xanh rừng, tiêu đề và nhóm nội dung rõ, trạng thái minh họa hiện ngay.
4. Đọc được truyện/cảnh → nhập bản kể → góp ý cục bộ; rỗng không gửi.
5. Lịch nhóm ngày, hiển thị khoảng giờ và loại hoạt động bằng chữ.
6. Chọn đủ sáu hồ sơ, copy hoạt động, không hướng dẫn action không tồn tại.
7. Chat nêu gửi Meta ngay tại composer, không key vẫn dùng Waldorf được.
8. 900×600 không cắt nội dung quan trọng; Tab/focus và cuộn dùng được.

Chấm đạt/chưa đạt từng mục trên cùng build/seed. Ghi mức `thủ công` cho
quan sát UI và `tự động` cho custom runner; không gọi là người dùng độc lập.
Build release + chạy lại `xuvi-tests`; kết quả mới + INDEX, không sửa kết quả
cũ. Các chiều ngoài phạm vi chưa đo phải khai rõ; không suy ra tổng toàn hệ.
