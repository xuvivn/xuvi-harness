# Technology — các quyết định công nghệ thay được mỗi lần chạy

Thư mục này chứa **mọi chi tiết công nghệ** của harness: ngôn ngữ, runtime, nhà cung cấp AI, giao diện, hệ điều hành, cách lưu trữ, cách đóng gói. Logic cốt lõi trong [`core/`](../core/) tuyệt đối không phụ thuộc vào các file này — nó chỉ phụ thuộc vào những *giao diện* mà các file này chọn cài đặt.

---

## Cách dùng (mỗi lần bắt đầu một bản dựng / một lần chạy thử)

1. **Sao chép trạng thái quyết định**: mở [quyet-dinh-hien-tai.md](quyet-dinh-hien-tai.md) và điền lại từ đầu (hoặc sửa những dòng thay đổi so với lần trước). Đây là "hồ sơ công nghệ" của lần chạy này — code và agent hỗ trợ bạn đều đọc nó trước.
2. **Mỗi quyết định thuộc đúng một file**: đối chiếu bảng dưới. Một quyết định không được ghi ở hai nơi; `quyet-dinh-hien-tai.md` chỉ *tổng kết* lựa chọn và liên kết tới file chi tiết.
3. **Điền theo mẫu**: mỗi file technology có sẵn (a) các lựa chọn khả thi kèm đánh đổi, (b) một lựa chọn *tham chiếu* — chính là lựa chọn DeepSeek Harness đang dùng, để bạn có điểm bắt đầu — và (c) ô "Quyết định của lần này" để bạn ghi lựa chọn của mình.
4. **Kiểm tra ảnh hưởng**: cuối mỗi file technology có mục "Ảnh hưởng lên core" liệt kê các giao diện bị chạm tới. Ngoài danh sách đó, việc đổi công nghệ **không** được đòi sửa logic core. Nếu bạn thấy mình phải sửa core để đổi một công nghệ, giao diện core đang bị rò rỉ chi tiết công nghệ — sửa giao diện, không sửa logic.

## Việc nào vào file nào

| Bạn muốn quyết định… | File |
|---|---|
| Ngôn ngữ, framework plugin, mô hình tiến trình, đóng gói thư viện | [runtime-va-ngon-ngu.md](runtime-va-ngon-ngu.md) |
| Hỗ trợ nhà cung cấp/model AI nào, khả năng từng model | [ai-models.md](ai-models.md) |
| Model chạy ở đâu (API đám mây / tự host / local), streaming, retry, xác thực, đếm token | [ai-model-runtime.md](ai-model-runtime.md) |
| Bật tool nào trong chat, nối MCP/server ngoài, trình bày tool trên UI | [chat-va-tools-integration.md](chat-va-tools-integration.md) |
| Làm ứng dụng macOS native (công nghệ cửa sổ, IPC, ký số, sandbox của Apple) | [ung-dung-macos-native.md](ung-dung-macos-native.md) |
| Hệ điều hành đích, định dạng lưu trữ trên đĩa, cơ chế sandbox từng OS, đóng gói/phát hành nói chung | [nen-tang-luu-tru-va-dong-goi.md](nen-tang-luu-tru-va-dong-goi.md) |
| Một lần chạy cụ thể dùng tổ hợp nào | [quyet-dinh-hien-tai.md](quyet-dinh-hien-tai.md) |

Xem một bộ quyết định đã điền trọn vẹn: [../examples/vi-du-macos-deepseek-chat.md](../examples/vi-du-macos-deepseek-chat.md).

## Nguyên tắc viết quyết định

- **Ghi cả cái không chọn và vì sao** (một dòng là đủ). Quyết định không có lý do sẽ bị lật lại vô ích ở lần chạy sau.
- **Quyết định phải kiểm chứng được**: mỗi lựa chọn gắn với một tiêu chí nghiệm thu trong file `core/` tương ứng.
- **Không quyết định thay cho core**: file technology chọn *cài đặt* cho giao diện, không định nghĩa lại hành vi. Muốn đổi hành vi → đó là sửa `core/`, một quyết định khác hẳn và ảnh hưởng mọi lần chạy.
