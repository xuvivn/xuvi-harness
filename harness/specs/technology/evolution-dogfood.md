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
trạng thái chuyển nhà.

---

## Hợp nhất vào runtime gốc và cơ chế Tự cập nhật 3 tầng (2026-11-03, Loop Self-Update)

Để công cụ (`xuvi` CLI và ứng dụng macOS `XuVi — Vườn học`) thực sự **tự cập nhật
chính mình** một cách trung thực theo Đạo và `cron.md`, hệ thống hợp nhất
`Evolution.swift` vào runtime gốc `app/Sources/XuviCore/Evolution.swift` cùng
bộ điều khiển phán quyết máy kiểm được và cơ chế tự cập nhật 3 tầng:

### 1. Cổng phán quyết máy kiểm được (`EvolutionGateJudge`)

- Đọc báo cáo nháp của orchestrator cùng kết quả `[EvolutionCheck]` từ `EvolutionVerifier`.
- Trích xuất trạng thái từng cổng `G1`, `G2`, `G3` (`ĐẠT` / `CHƯA ĐẠT` / `CHƯA ĐO`)
  và phán quyết nháp của model (`KEEP`/`GIỮ`, `REVERT`/`HUỶ`, `PARK`/`ĐỖ`).
- **Quy tắc bất biến (Cổng 2 trong `cron.md`):**
  - Nếu có bất kỳ check kỹ thuật nào trong `EvolutionVerifier` thất bại ⇒
    `controllerVerdict = .revert` (`HUỶ`), `canPromote = false`.
  - Nếu mọi check kỹ thuật đạt nhưng bất kỳ cổng nào trong `G1`, `G2`, `G3` là
    `CHƯA ĐO`, `CHƯA ĐẠT`, hoặc không khai báo tường minh `ĐẠT` ⇒
    `controllerVerdict = .park` (`ĐỖ`), `canPromote = false`, `overrodeModelKeep = (modelDraft == .keep)`.
    Model tự viết `Giữ (KEEP)` trong nháp **bị bác tự động** ở tầng controller.
  - Chỉ khi toàn bộ check kỹ thuật đạt **và** cả 3 cổng `G1`, `G2`, `G3` đều
    được xác nhận `ĐẠT` (không có chuỗi `CHƯA ĐO`/`CHƯA ĐẠT` trên cùng dòng cổng)
    và model không yêu cầu `HUỶ`/`ĐỖ` ⇒ `controllerVerdict = .keep` (`GIỮ`), `canPromote = true`.

### 2. Ba tầng tự cập nhật trung thực của XuVi (`SelfUpdate`)

Khi người dùng hỏi *"Bạn có thể tự cập nhật chính mình không?"* hoặc yêu cầu công
cụ tự cập nhật, XuVi thực thi và giải thích trung thực theo 3 tầng có thật:

1. **Tầng 1 — Tự cập nhật dữ liệu & trí nhớ bền vững trong ứng dụng (`WaldorfStore` + `SessionLog` + `SelfUpdateStore`)**:
   - Ở chế độ thường (không phải `--ui-preview`), `XuViApp` lưu bền dữ liệu Vườn học
     tại `~/Library/Application Support/XuViHarness/waldorf.json` (ghi nguyên tử qua
     candidate trong `WaldorfStore.update`, tự tạo `.bak` và khôi phục khi file chính hỏng),
     lưu bền nhật ký hội thoại tại `~/Library/Application Support/XuViHarness/sessions/chat-session.jsonl`,
     và lưu trạng thái tự tiến hoá tại `~/Library/Application Support/XuViHarness/self-update-state.json`.
   - Qua công cụ `xuvi_garden_update` và `xuvi_self_status` trong mục `Trò chuyện`,
     orchestrator (`muse-wonder-1.0`) có thể tự đọc trạng thái hiện tại, thêm hoặc
     cập nhật truyện/cảnh/mục tiêu/chủ đề năm học trong `WaldorfStore`, và ghi nhận
     đề xuất tiến hoá mới vào `SelfUpdateStore` — thay đổi phản ánh ngay trên UI và
     bền vững qua các lần khởi động lại.
2. **Tầng 2 — Tự tiến hoá specs → code → kiểm thử trong bản sao cô lập (`xuvi evolution-live`)**:
   - Orchestrator (`muse-wonder-1.0`) tự đọc `ĐẠO.md`, `README.md`, `cron.md`,
     `specs` (JIT), lập `todo`, sửa spec nhiệm vụ trước, uỷ quyền (`delegate` đủ 4
     thành phần) cho worker (`muse-spark-1.3-contributor`) viết test regression,
     review artefakt con, sửa `Sources/`, chạy `run_tests` dưới `sandbox-exec`
     (fail-closed, không mạng), và được kiểm định bởi `EvolutionVerifier` +
     `EvolutionGateJudge`.
3. **Tầng 3 — Tự kiểm tra, biên dịch & cập nhật gói ứng dụng cục bộ (`SelfUpdater` / `xuvi self-update` / `xuvi_self_update_run`)**:
   - Chạy bộ kiểm thử tự động `xuvi-tests` dưới `sandbox-exec` (hoặc runner cố định)
     để xác nhận toàn bộ bất biến hệ thống (`passed > 0, failed == 0`).
   - Khi có yêu cầu rebuild/self-update trên workspace nguồn hợp lệ (có `Package.swift`,
     `ĐẠO.md` nguyên vẹn) và toàn bộ kiểm thử đạt: biên dịch binary mới, sao lưu
     bundle `.app` cũ sang `.app.bak`, đóng gói và ký ad-hoc (`codesign --force --sign -`
     + `codesign --verify --strict`) bundle mới, ghi `SelfUpdateManifest` (phiên bản,
     thời điểm, SHA256 binary, số test PASS, phán quyết cổng). Nếu bất kỳ bước nào
     thất bại, tự động khôi phục từ `.app.bak` và báo lỗi trung thực.

### 3. Ranh giới trung thực (Cổng G1/G2/G3)

- **Không nói quá khả năng (G2):** XuVi nói rõ mình tự cập nhật dữ liệu bền vững,
  ghi nhận đề xuất tiến hoá, chạy loop `evolution` sửa specs/code trong workspace
  cô lập và tự biên dịch/đóng gói lại bản `.app` cục bộ qua cổng kiểm thử; XuVi
  **không** tự sửa trọng số mô hình AI trên máy chủ Meta và **không** tự ý vượt
  cổng `EvolutionGateJudge`.
- **Bảo vệ Đạo & riêng tư (G1/G3):** Mọi tool tự cập nhật từ chối tuyệt đối việc
  ghi/sửa `ĐẠO.md` và chặn đọc/ghi `.env`, khoá bí mật, symlink hay đường dẫn
  ngoài phạm vi. Chế độ `--ui-preview` giữ cô lập hoàn toàn trong bộ nhớ.
