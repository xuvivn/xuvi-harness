# Cấu hình META_API_KEY trên UI — 2026-11-03

## Phạm vi và quyết định

Người dùng yêu cầu chỗ nhập khóa trong app. Một điểm nghẽn B3: cấu hình qua
Terminal/cwd không đáp ứng cách mở từ Finder. Giả thuyết: SecureField + lưu khóa
qua Keychain riêng của app giúp cấu hình tại UI, không cần sửa .env và không
đưa khóa vào hội thoại/log/file. Spec trước code:
`harness/specs/technology/cau-hinh-meta-tren-ui.md`; runtime spec làm rõ ưu tiên
khóa nhập UI cho app, giữ CLI cũ khi không inject store.

**Phán quyết: bản triển khai dùng thử sẵn; ĐỖ nghiệm thu Keychain/UI thật và
benchmark toàn hệ.** Không tuyên bố đã xác thực khóa với Meta hoặc đã chuyển
nhà tự vận hành. Không chấm lại điểm 62.5 từ báo cáo bootstrap. Đây là lần đo
phạm vi cấu hình UI theo yêu cầu trực tiếp, không hoàn tất loop tiến hoá tổng thể.

## Trước/sau cùng checklist

| Tiêu chí | Trước | Sau | Bằng chứng và giới hạn |
|---|---|---|---|
| Nhập khóa được che trong Cài đặt | Không có control | SecureField, Lưu, Xóa, trạng thái | Rà code + build; chưa thao tác UI thực |
| Khóa lưu từ UI có hiệu lực ngay | Không có đường lưu | Provider ưu tiên store được inject; UI cập nhật trạng thái | Test store giả, key hư cấu |
| Dùng lại sau khi tạo provider mới | Không có store | Dùng lại store, override env giả | Test tự động; không suy ra Keychain persistence thật |
| Lỗi save không mất khóa cũ, read lỗi không đổi nguồn | Không có contract này | Test đạt | Store giả có fault injection |
| Xóa + fallback | Không có action | Chỉ xóa item của app; env/.env vẫn tương thích | Test store/env giả |
| Không tự gọi mạng khi lưu | Không có đường UI | Save/delete chỉ gọi credential store | Rà code; không gọi Meta trong lần này |
| Regression kỹ thuật | 44/44 | 54/54 | Cùng custom runner thực chạy tại repo |
| Build bản giao | Chưa có bản mới | Release build + bundle ad-hoc verify đạt | Không phải UI acceptance hoặc notarization |

Lệnh thực chạy:

- `swift run --package-path app xuvi-tests`: trước **44 passed, 0 failed**;
  sau **54 passed, 0 failed**. Mức `tự động`; 10 kiểm tra mới dùng in-memory
  credential store và environment closure giả. Không test qua Keychain thật.
- `swift build --package-path app --product XuViApp -c release`: đạt sau khi
  xử lý lỗi toolchain thiếu macro `SwiftUIMacros.StateMacro`; dùng StateObject
  với ObservableObject cho state field, cùng cơ chế app hiện có.
- `codesign --force --sign - app/dist/XuVi-KeySettings.app` và
  `codesign --verify --strict app/dist/XuVi-KeySettings.app`: exit 0.
- Build còn warning toolchain/framework của code cũ; không gọi là sạch warning.

## Bằng chứng toàn thể và cổng

| Mục | Trạng thái lần này |
|---|---|
| G1 | Rà thay đổi không tác động giáo dục/trẻ; chưa nghiệm thu toàn sản phẩm |
| G2 | UI nói đã cấu hình, chưa kiểm kết nối; không gọi lưu là xác thực. Mức mock và chưa kiểm chứng khai rõ |
| G3 | Test không dùng key thật, không đọc .env/Keychain thật; adapter Keychain chỉ query item service `vn.xuvi.harness.meta-api-key`, account `META_API_KEY`; nghiệm thu runtime thật còn đỗ |
| B1 | Không đổi hành trình Waldorf; chưa đo lại |
| B2 | Không đổi nội dung/nguồn giáo dục; chưa đo lại |
| B3 | Có controls mới theo yêu cầu; kiểm tĩnh/build, chưa UI walkthrough |
| B4 | Không đổi core/hợp đồng worker; checks cũ vẫn đạt; chưa đo toàn bộ |
| B5 | 54 checks, release build, ký bundle đạt; không kiểm lại persistence Waldorf/UI |
| B6 | Không gọi Meta live; khóa mới chưa được xác thực |
| B7 | Không quét nguồn/model; quyết định dựa trên yêu cầu người dùng và code repo |
| B8 | Specs trước, kết quả mới, INDEX, chỉ file whitelist trong status; loop toàn hệ chưa hoàn tất |
| Tổng/delta | Không tính; bảng này không thay benchmark toàn thể |

## Bản giao và giới hạn

`app/dist/XuVi-KeySettings.app` chứa executable release vừa build, Info.plist
cùng bundle id `vn.xuvi.harness`, version 0.1.1/build 20261103.1, ký ad-hoc.
Không đóng app đang chạy, không thay executable của bundle cũ. Người dùng tự
đóng bản cũ rồi mở bản mới để tránh macOS kích hoạt lại process cũ cùng bundle id.

Chưa thao tác nhập/lưu/xóa qua UI thực và chưa xác nhận OS cấp quyền Keychain.
Nếu macOS hỏi quyền, người dùng tự quyết trên hộp thoại của hệ điều hành. Agent
không biết/đọc key, không tự cấp quyền, không dùng khóa thật trong kiểm thử.
UserDefaults/.env/clipboard/log không được dùng để lưu giá trị nhập.

Không commit/push, không sửa Đạo hoặc kết quả cũ. Thay đổi được giới hạn ở
credential provider, Settings UI, test liên quan và specs/kết quả; không can thiệp
workload evolution đang chạy ở chat chính. Code/bundle sinh ra vẫn bị ignore.

Bước nghiệm thu tiếp: người dùng mở bản mới, nhập khóa trực tiếp trong Cài đặt,
lưu rồi mở lại app để xác nhận persistence; nếu muốn kiểm AI thật, tự gửi một câu
minh họa trong Chat. Chỉ sau thao tác thật mới nâng mức bằng chứng tương ứng.
