# Cấu hình Meta trực tiếp trong app (2026-11-03)

Một điểm nghẽn B3: mở app từ Finder không thể nhập META_API_KEY. Người dùng yêu
cầu cấu hình ngay trên UI; không bắt dùng Terminal hoặc .env.

## Hành vi và bí mật

- Cài đặt & bằng chứng có SecureField trống `META_API_KEY`, Lưu khóa, Xóa khóa
  đã lưu, trạng thái và thông báo kết quả. Không nạp giá trị cũ lên field.
- Lưu khóa vào đúng generic-password item của app trong macOS Keychain, service
  `vn.xuvi.harness.meta-api-key`, account `META_API_KEY`; không đọc/ghi item khác.
  Không đồng bộ iCloud; không UserDefaults, .env, log, prompt hay clipboard.
- Lưu thay thế dùng update hoặc add (không xóa khóa cũ trước). Bỏ whitespace đầu
  cuối, từ chối rỗng; clear field chỉ khi lưu thành công. Lỗi OS chỉ báo mã số và
  hướng dẫn chung; không đưa key/response body vào lỗi. Hủy lưu giữ bản cũ.
- Riêng app ưu tiên khóa người dùng lưu trên UI rồi mới env/.env cho tương thích;
  CLI giữ thứ tự cũ khi không inject credential store. Lỗi đọc Keychain không
  âm thầm dùng khóa nguồn khác. Xóa chỉ item của app, không sửa env/.env.
- App cập nhật trạng thái ngay, không cần restart; lần gửi tiếp dùng khóa mới.
  Lưu không gọi mạng. `Đã cấu hình` không có nghĩa `Đã xác thực với Meta`.
- Trạng thái không hiển thị một phần key. Lỗi lưu/xóa hiện ngay, field rời màn
  hình bị bỏ. Nút lưu/xóa tắt khi chat đang chạy để không đổi key giữa một lượt.

## Kiểm chứng trước/sau

- Cùng checklist: nhập được bằng SecureField, lưu/delete/update/error, không lộ
  bí mật, không cần cwd khi có khóa UI, không tự gọi mạng, chat dùng cùng provider.
- Test credential contract qua store giả/in-memory và environment giả, key hư cấu;
  không dùng test/tool đọc Keychain thật hay .env thật. Test restart provider với
  cùng store, override env, delete fallback, lỗi đọc fail-closed, lỗi save giữ cũ.
- Build/test/debug/release rồi đóng gói lại `app/dist/XuVi.app` để người dùng mở
  đúng bản. Không tự thoát app đang mở hoặc ghi đè executable đang chạy; dùng
  staging bundle và thay tên bundle nguyên tử khi an toàn.
  Khi bản cũ còn chạy, được giao bundle dùng thử riêng `XuVi-KeySettings.app`
  để không can thiệp phiên hiện tại; người dùng tự đóng/mở bản mới.
- Ghi file kết quả mới và INDEX; từng B1–B8/G1–G3 có bằng chứng hoặc chưa đo;
  không suy điểm tổng từ số unit checks. Keychain thật/UI cần thao tác thực mới
  được nhận mức thủ công/live. Không tự gửi nội dung người dùng cho Meta.
