# Kiểm tra nghiệm thu

Các mục dưới đây là hợp đồng kiểm thử, chưa phải báo cáo đã đạt. Kết quả thực tế lưu trong `04_verification.md`.

| Nhóm | Thao tác cần kiểm | Kết quả mong đợi |
| --- | --- | --- |
| Khởi chạy | Cài dependencies, chạy bằng một lệnh | Backend/frontend sẵn sàng, có URL local, không cần khóa để mở dữ liệu |
| Lưu trữ | Sửa chương trình, tải lại trang/khởi động server | Dữ liệu đã sửa còn nguyên |
| Độ tuổi/lớp | Chọn mầm non, lớp 3, lớp 9, lớp 12 | Mục tiêu và nội dung thay theo nguồn tương ứng |
| Nguồn | Tìm văn bản Bộ theo từ và mở trang | Xem được vị trí nguồn thật; mã nội bộ không giả làm mã Bộ |
| Đối chiếu | Thêm yêu cầu, nối hoạt động, đổi mức độ | Ma trận cập nhật, phần chưa đối chiếu vẫn hiện rõ |
| Năm học | Thay ngày/tuần/block | Bảng phân phối và xuất hồ sơ phản ánh thay đổi |
| Lịch tuần | Sửa buổi, người dạy, mục tiêu và phút thực tế | 5 ngày, 08:00–15:00; cảnh báo trùng; tách phút học/sinh hoạt |
| Nhật ký | Thêm trẻ ví dụ, điểm danh, quan sát, minh chứng | Phân biệt kế hoạch với đã dạy; báo cáo cá nhân đúng dữ liệu |
| Hồ sơ | Tạo cả sáu tài liệu | Đủ trường master prompt, nguồn dữ liệu/phiên bản rõ, xuất và in được |
| Nhập truyện | TXT/MD/HTML/DOCX/PDF/DOC, văn bản lỗi hoặc quá lớn | Duyệt văn bản trước lưu; lỗi rõ; không chạy HTML/script |
| Cảnh nhớ | Chia cảnh, sửa, thêm/xóa, lưu và mở lại | Giữ diễn biến, nhân vật, chi tiết và sửa tay |
| Tranh | Chọn phong cách/tuổi, tạo hoặc tải tranh lên | Tranh gắn đúng cảnh; lỗi provider không giả thành công |
| Luyện kể | Có hình, ẩn dần, tự do; trợ giúp chữ riêng | Điều kiện được lưu đúng với từng lượt |
| Âm thanh | Ghi/tải, dừng, nghe, phiên âm và sửa | Media lưu thật; từ chối mic có hướng dẫn; không tự tạo transcript giả |
| Góp ý | So bản kể với phiên bản nguồn đã lưu | Có đoạn dẫn chứng, phân biệt bỏ sót/chuyển thể; không điểm tổng; không suy giọng từ chữ |
| Lịch sử | Sửa truyện sau một lần luyện | Lần luyện cũ còn snapshot đúng, so sánh điều kiện công bằng |
| Nhịp kể | Mở bảy ngày, đánh dấu và quay lại | Gợi ý tùy chọn, không khóa chức năng theo ngày |
| Quyền dữ liệu | Gọi AI, xuất, xóa, sao lưu/khôi phục | Thông báo dữ liệu gửi đi; bản sao đầy đủ gồm media; xác nhận xóa |
| Khôi phục lỗi | Nhập backup hỏng và thử tệp sai kiểu | Từ chối rõ, không mất dữ liệu hiện có |
| Giao diện | Desktop và màn hình hẹp, bàn phím | Không tràn/che nút; nhãn rõ; trạng thái tải/lỗi có thể xử lý |

## Quy tắc dữ liệu giáo dục

- Dữ liệu ví dụ không phải quan sát học sinh thật. Không lấy hồ sơ cá nhân trong `minhxuvi` để seed.
- Nguồn Waldorf tham khảo không đồng nghĩa chứng nhận của tổ chức Waldorf. Văn bản Bộ được nhập theo bản cung cấp, không tự tuyên bố đã hợp nhất mọi sửa đổi mới.
- Truyện dài, tiến trình môn và việc thực hành là ba mạch có thể nối; nghe truyện không tự là minh chứng đạt năng lực.
- Khi thay bài hoặc thời lượng, hiển thị ảnh hưởng lên kế hoạch và đối chiếu; không sửa lịch sử quan sát đã có.
- Học liệu chuẩn bị đầy đủ và thẻ dạy gọn là hai cách xem cùng dữ liệu.
