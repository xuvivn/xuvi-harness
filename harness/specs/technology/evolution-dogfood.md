# Evolution — cổng tự vận hành có bằng chứng (2026-11-03)

Đứng dưới Đạo, core/13 và Waldorf spec 11 §5. Một điểm nghẽn: chưa chứng minh
được loop tự tiến hoá; bài toán 42 chỉ là smoke test, không được nâng thành dogfood.

**Trạng thái sau lần thử 2026-11-03:** candidate, chưa kích hoạt thay runtime
gốc. Chuỗi live và 13 kiểm tra kỹ thuật đã đạt, nhưng draft của Wonder kết luận
GIỮ khi chính nó ghi G1–G3 CHƯA ĐO. Người điều phối veto, ĐỖ chuyển nhà; không
coi verifier kỹ thuật xanh là cổng benchmark xanh. Xem kết quả evolution tương ứng.

## Contract tối thiểu

- CLI `evolution-live <clean-workspace> --confirm-remote` chạy Wonder qua chính
  adapter XuVi; xác nhận danh mục có Wonder và Spark Contributor trước khi chạy.
  Từ chối thiếu consent hoặc workspace không có marker `.evolution-workspace`.
- Workspace do người điều phối chuẩn bị theo allowlist: Đạo/README/cron/specs,
  Package.swift và Sources Swift; không copy `.env`, .git, database, log cũ,
  symlink hay nội dung riêng tư. Không cho agent đọc workspace gốc.
  Prompt phải đưa manifest đường dẫn tối thiểu: package ở `app/`, domain store
  `app/Sources/XuviWaldorf/Waldorf.swift`, runner `app/Sources/XuviTests/main.swift`.
  Không bắt model đoán cấu trúc thư mục khi chưa cấp tool liệt kê.
- Tools read/write/edit/todo/delegate và `run_tests` cố định. Không cấp shell
  tuỳ ý; `run_tests` không nhận command/path từ model, chỉ build/run custom runner
  `xuvi-tests` của bản sao. Chính Wonder phải gọi tool, không tính lệnh người điều
  phối chạy thay. Model/worker steps hữu hạn; timeout cho runner, huỷ dọn process.
- Mọi file tool của profile này chặn `.env`, symlink, traversal, Đạo ghi/sửa,
  và chỉ cho ghi Sources hoặc một file spec nhiệm vụ và báo cáo đã chọn trước.
  Chốt ở chính tool, kể cả gọi trực tiếp. Agent không được sửa marker/Package.swift.
- Runner chạy qua sandbox-exec với profile fail-closed: read chỉ workspace,
  toolchain/system; write chỉ workspace và thư mục scratch riêng; mạng bị cấm.
  Environment mới theo allowlist, không thừa kế API key; HOME/TMPDIR/cache đều
  trong workspace. Thiếu sandbox hay preflight bị từ chối thì dừng, không bỏ sandbox.
  Thu stdout đồng thời để tránh đầy pipe, timeout hữu hạn, cap output.
  Khi SwiftPM dùng cache/temp ngoài quyền đã cấp, runner được biên dịch trực tiếp
  bằng `swiftc` cố định cho XuviCore/Meta/Waldorf và executable XuviTests, cùng
  sources và checks. Không chạy Package.swift ngoài sandbox để lách quyền.
  Test fixtures nhận `XUVI_TEST_TMPDIR` là scratch do runner cấp. Kiểm confinement
  bằng sandbox thật chạy ở tầng điều phối; không yêu cầu sandbox lồng trong runner.
  macOS Foundation atomic-write dùng `TemporaryItems/NSIRD_<executable>_*` ngoài
  TMPDIR: runner đặt tên executable riêng mỗi lượt và chỉ cấp read/write cho
  prefix replacement của chính lượt đó; không cấp cả thư mục tạm người dùng.
  Có thể đọc metadata dung lượng filesystem cho atomic-write, không cấp đọc
  preferences hay file cá nhân. Đây là ngoại lệ OS cụ thể, phải kiểm sentinel lại.
- Logs cha/con ghi bền riêng trong workspace. Log cấu hình host api.meta.ai,
  provider meta, model theo vai; không ghi credentials. Bản báo cáo nháp phải do
  Wonder ghi qua tool, gồm giả thuyết, trước/sau, giới hạn B1–B8/G1–G3, phán quyết.
- Sau run, verifier kiểm bằng sự kiện và artefakt: đọc Đạo/README/cron/spec nhiệm vụ,
  todo; delegate đủ 4 thành phần; cha đọc một artefakt con; test trước và sau exit 0;
  có thay đổi Sources và spec theo nhiệm vụ; báo cáo do cha ghi; đúng model/vai.
  Không chỉ kiểm câu trả lời cuối. Verifier không tự nâng thành nghiệm thu toàn hệ.

## Nhiệm vụ thật để nghiệm thu

Chỉ trong bản sao: cải thiện lỗi mất trạng thái khi `WaldorfStore.update` ghi
thất bại. Đọc contract rồi nêu giả thuyết; sửa spec nhiệm vụ trước khi sửa code;
uỷ quyền Spark viết test regression; Wonder review, tích hợp implementation và
chạy trước/sau. Mutation phải trên candidate, chỉ công bố `data` sau save thành
công; lỗi giữ data cũ. Dùng fixture tổng hợp với đích ghi là directory để gây lỗi.
Không sửa dữ liệu/ghi âm/UI. Đây là workload thật dùng để đo cổng tự vận hành;
thay đổi trong bản sao chưa tự động được đưa về sản phẩm gốc.

## Test và quyết định

`swift run --package-path app xuvi-tests` là runner chính thức hiện tại, thay
cho chữ `swift test` ở spec 11 §5/§7 trong môi trường chưa có XCTest. Giữ toàn bộ
phạm vi hành vi bắt buộc; không hồi tố để cho bài toán 42 đạt cổng cũ.

Phủ định bắt buộc: symlink đọc/ghi, traversal, ghi Đạo, đọc file bí mật giả,
workspace không marker, runner timeout/nonzero, không có sandbox/không có consent.
Không dùng secret thật trong test. Chạy cùng checklist trước/sau; lưu kết quả mới.
Đạt bốn khoản spec 11 §5 là cần; cổng benchmark cron.md vẫn quyết giữ/huỷ/đỗ.
G1/G3 chưa nghiệm thu toàn sản phẩm thì ghi CHƯA ĐO, không tự đổi sang ĐẠT.

## Chốt phán quyết bắt buộc trước lần chạy tiếp

Sự cố quan sát: draft tự nhận KEEP sau khi liệt kê mọi cổng CHƯA ĐO. Không
chấp nhận lập luận "đã đúng ở phạm vi unit test nên giữ dù chưa qua cổng".
Lần sinh tiếp phải có phán quyết bằng dữ liệu máy kiểm được ở tầng điều phối:
thiếu bất kỳ cổng bắt buộc hoặc chưa có phép đo trước/sau tổng thể ⇒ ĐỖ,
model không được override. Báo cáo model là nháp; không tự làm nguồn xác nhận
cho chính nó. Verifier phải có test phủ định KEEP + CHƯA ĐO và từ chối nâng
trạng thái chuyển nhà. Dừng đợt hiện tại để ghi bài học, chưa tiếp tục nâng điểm.
