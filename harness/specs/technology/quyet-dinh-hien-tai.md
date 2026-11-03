# Quyết định công nghệ — lần chạy hiện tại

> **Đây là file bạn điền trước mỗi lần xây/chạy.** Sao chép nó, điền mọi ô, và giữ nó ở đầu ngữ cảnh làm việc. Các file chi tiết trong thư mục này giải thích từng lựa chọn; file này chỉ tổng kết.
>
> Để trống một ô = chưa quyết định = **không được viết code phụ thuộc vào nó**.

| # | Quyết định | Lựa chọn của lần này | File chi tiết |
|---|---|---|---|
| 1 | Ngôn ngữ + runtime chính | _(ví dụ: TypeScript trên Node ≥ 24)_ | [runtime-va-ngon-ngu.md](runtime-va-ngon-ngu.md) |
| 2 | Cơ chế module/plugin (framework hay tự viết) | | [runtime-va-ngon-ngu.md](runtime-va-ngon-ngu.md) |
| 3 | Mô hình tiến trình (một tiến trình / host+client / worker) | | [runtime-va-ngon-ngu.md](runtime-va-ngon-ngu.md) |
| 4 | Nhà cung cấp + model mặc định | | [ai-models.md](ai-models.md) |
| 5 | Nhà cung cấp + model phụ (nếu có) | | [ai-models.md](ai-models.md) |
| 6 | Model chạy ở đâu (API cloud / tự host / local) | | [ai-model-runtime.md](ai-model-runtime.md) |
| 7 | Cách lấy bí mật API (biến môi trường / kho bí mật OS / OAuth) | | [ai-model-runtime.md](ai-model-runtime.md) |
| 8 | Mặt tiếp xúc chính (headless / web chat / macOS native / nhúng SDK) | | [chat-va-tools-integration.md](chat-va-tools-integration.md), [ung-dung-macos-native.md](ung-dung-macos-native.md) |
| 9 | Bộ tool bật mặc định trong chat | | [chat-va-tools-integration.md](chat-va-tools-integration.md) |
| 10 | Nguồn tool ngoài (MCP server nào, nếu có) | | [chat-va-tools-integration.md](chat-va-tools-integration.md) |
| 11 | Chính sách approval mặc định (`ask` / `never` / preset) | | [chat-va-tools-integration.md](chat-va-tools-integration.md) |
| 12 | Định dạng lưu session trên đĩa | | [nen-tang-luu-tru-va-dong-goi.md](nen-tang-luu-tru-va-dong-goi.md) |
| 13 | Cơ chế sandbox cho thực thi của model, theo OS | | [nen-tang-luu-tru-va-dong-goi.md](nen-tang-luu-tru-va-dong-goi.md) |
| 14 | Hệ điều hành đích + cách đóng gói/phát hành | | [nen-tang-luu-tru-va-dong-goi.md](nen-tang-luu-tru-va-dong-goi.md), [ung-dung-macos-native.md](ung-dung-macos-native.md) |
| 15 | Chiến lược ngữ cảnh: bộ đếm token, ngưỡng áp lực và bậc xử lý bật (cắt kết quả tool / nén / agent con), model tóm tắt nén, nơi lưu ghi chú, trần token của kết quả tool | | [ai-model-runtime.md](ai-model-runtime.md), [chat-va-tools-integration.md](chat-va-tools-integration.md) (nguyên tắc: [core/07](../core/07-prompt-va-ngu-canh.md), [core/10](../core/10-phoi-hop-va-mo-rong.md)) |

## Ghi chú riêng của lần chạy này

_(Viết tự do: ràng buộc riêng, thứ cố tình bỏ qua, hạn chót phải giữ… Ví dụ: "Bản này chỉ cần chạy trên máy cá nhân, bỏ qua đa người dùng và telemetry.")_

-

## Lịch sử thay đổi quyết định

| Ngày | Ô # | Từ → Thành | Lý do một dòng |
|---|---|---|---|
| | | | |

> Khi một quyết định đổi giữa chừng: cập nhật bảng trên, rồi kiểm tra mục "Ảnh hưởng lên core" trong file chi tiết của quyết định đó trước khi sửa code.
