# Evolution candidate — 2026-11-03 — live trọn chuỗi, đỗ phán quyết tự chủ

- **Người điều phối:** Codex; sinh code qua Muse CLI Wonder, phản biện Spark
  Contributor; workload live chạy bằng Wonder → Spark qua adapter của XuVi.
- **Baseline:** bản chẩn đoán sau bootstrap; điểm 62.5 trước đó chỉ là lịch sử.
- **Điểm nghẽn duy nhất:** chưa chứng minh đường tự thực hiện loop tiến hoá.
- **Giả thuyết:** bổ sung profile evolution tối thiểu và tools bị giới hạn sẽ
  cho XuVi đọc specs, uỷ quyền một việc thật, kiểm lại, chạy test và tự viết nháp.
- **Phán quyết của người điều phối:** **ĐỖ chuyển nhà / ĐỖ đưa candidate vào
  runtime gốc.** Có tiến bộ kỹ thuật kiểm được, nhưng draft của model kết luận
  KEEP dù tự ghi G1–G3 CHƯA ĐO. Theo cron.md, điều đó không đủ để giữ thay đổi.
  Không công nhận phán quyết của model là nguồn xác nhận cho chính nó.
- **Phạm vi giữ lại:** specs candidate, bằng chứng mới, bài học. Code thử nghiệm
  nằm riêng trong vùng ignore; `app/Sources` gốc chưa thay đổi trong đợt này.

## 1. Specs trước, sản phẩm sinh sau

- Thêm `harness/specs/technology/evolution-dogfood.md`: workspace allowlist,
  consent, file policy tại chính tool, runner cố định, sandbox fail-closed,
  manifest đường dẫn, log/verification, workload và điều kiện giữ/đỗ.
- Làm rõ custom test runner ở Waldorf spec 11 §5; không hồi tố bài toán 42.
- Sau khi thấy draft KEEP sai cổng, bổ sung yêu cầu phán quyết máy kiểm được
  vào spec evolution **trước lần chạy tiếp**. Chưa sinh chốt mới; dừng đợt để ghi
  sự cố và điều kiện thiếu, không lặng lẽ sửa báo cáo của model thành đúng.

## 2. Kết quả trước/sau, cùng checklist

| Phép đo | Trước | Sau | Mức bằng chứng |
|---|---|---|---|
| CLI evolution và fixed runner | Chưa có | Có trong candidate | tự động: build + gọi thật |
| Checks hiện có ở repo gốc | 44/44 | Không thay runtime gốc | tự động baseline |
| Candidate trước workload | — | 73/73 ngoài sandbox; 69/69 trong sandbox + 4 SKIP có nhãn | tự động; 4 confinement checks chạy riêng ngoài sandbox lồng |
| Workload sau thêm regression, trước implementation | Store không giữ data khi save lỗi | Hai assertions giữ data cũ đỏ, runner exit 1 | tự động, do chính Wonder gọi |
| Workload sau implementation | — | Runner exit 0 | tự động, do chính Wonder gọi |
| Kiểm lại độc lập candidate cuối | — | **79 passed, 0 failed**, exit 0 | tự động, người điều phối chạy |
| Chuỗi đọc docs/todo/delegate/đọc artefakt/test/spec/source/report | Chưa có đủ bằng chứng | **13/13 checks kỹ thuật đạt** | live qua adapter XuVi + kiểm log/artefakt |
| Phán quyết tôn trọng cổng benchmark | Chưa chứng minh | **Chưa đạt**: draft KEEP + G1–G3 CHƯA ĐO | đọc draft thật; điều phối veto |

Không tính điểm tổng mới. Không đủ phép đo để chuyển 62.5 thành một điểm mới
hoặc khẳng định mọi chiều không regress. Các counts thêm vào không phải điểm
benchmark tổng thể và không tự nâng B4/B5/B8 lên 4.

## 3. Workload thật và audit độc lập

Workload chỉ trong bản sao: lỗi `WaldorfStore.update` mutate `data` trước save,
để lại data chưa lưu khi save ném lỗi. Worker Spark thêm sáu checks bằng dữ
liệu minh họa: ghi vào directory phải ném lỗi và giữ toàn bộ snapshot; đường
thành công phải publish đúng candidate, load lại bằng data đã publish.

Wonder đọc lại artefakt của worker, tự gọi runner để thấy hai assertions đỏ,
thêm helper nội bộ `save(_ value:)`, rồi đổi `update` thành candidate → mutate
candidate → save candidate → gán data. Không sửa các checks cũ, không đổi API
công khai, định dạng JSON, UI hay hành vi load/backup trong workload.

Audit log cha: sửa spec ở seq 38; đọc artefakt con sau delegate ở seq 47;
sửa implementation ở seq 56. Có runner baseline exit 0, regression exit 1,
và sau sửa exit 0. Report do cha viết bằng tool. Tất cả assistant events đã
kiểm có provider `meta`, cha Wonder, con Spark Contributor đúng vai.
Đạo của bản sao khớp bản gốc; bản gốc không diff.

Host `api.meta.ai` được cấu hình trong composition và endpoint adapter được
dùng thật. Đây không phải packet capture hay chứng minh mọi redirect ở tầng
mạng; không nâng audit cấu hình thành bằng chứng mạng toàn diện.

## 4. Sandbox: vấn đề thật, quyền cần thiết và phép đo

Không yêu cầu người dùng mở Full Disk Access hoặc gỡ hạn chế Codex/macOS.
Vướng mắc chính là profile sandbox của candidate và sandbox lồng của Muse:

- Muse không áp được thêm sandbox bên trong sandbox của nó; điều phối thực
  hiện các probes trực tiếp. Không coi lỗi sandbox lồng là quyền người dùng thiếu.
- Foundation có thể trả `/var/...` sau resolve, trong khi Seatbelt cần
  `/private/var/...`; profile phải chuẩn hoá đường dẫn vật lý.
- SwiftPM dùng temp/cache ngoài phạm vi. Runner chuyển sang pipeline `swiftc`
  cố định, cùng sources/checks, SDK/target/cache tường minh, từng process vẫn
  chạy trong sandbox. Không mở shell tuỳ ý cho model.
- Fixture dùng scratch qua `XUVI_TEST_TMPDIR`. Foundation atomic-write còn dùng
  `TemporaryItems/NSIRD_<executable>_*`; executable có tên riêng mỗi lượt, chỉ
  cấp prefix replacement của lượt đó. Không cấp cả thư mục tạm người dùng.
- Cho phép signal trong cùng sandbox để timeout hoạt động. Environment con
  theo allowlist, không thừa kế credentials.

Probe cuối trên profile thực: ghi trong workspace thành công; đọc sentinel giả
ngoài workspace thất bại và không trả nội dung; ghi ngoài thất bại; kết nối TCP
tới cổng localhost **đang mở** thất bại trong sandbox nhưng thành công ở control
ngoài sandbox. Cả năm kiểm tra đều true; không dùng secret thật làm sentinel.

## 5. Các lần thử và giới hạn

1. Lượt đầu: baseline test thành công nhưng CLI không cung cấp manifest `app/`,
   model dò sai đường dẫn. Điều phối dừng, bổ sung contract manifest rồi sinh lại.
2. Lượt mới: đọc đúng nguồn, baseline thành công; API Meta ngắt mạng với
   NSURLError -1005 trước delegate. Không diễn giải thành lỗi thiếu quyền.
3. Thử lại trên cùng workspace sạch, nạp lại log: tiếp tục spec → delegate →
   red/green → draft; CLI exit 0, verifier kỹ thuật 13/13. Đây là tiếp tục cùng
   nhiệm vụ sau lỗi mạng, không phải ba loop benchmark độc lập đã hoàn tất.

Giới hạn còn lại: toolchain/SDK/target cố định cho máy này; timeout per-process,
chưa chứng minh huỷ cả cây tiến trình; snapshot/hash và verifier không phải một
hệ chống giả mạo tổng quát; verifier kỹ thuật còn kiểm sự có mặt hơn là đầy đủ
mọi thứ tự/semantic. Draft tự nhận KEEP sai cổng là lý do đỗ, không che bằng
câu “13/13 PASS”. Chưa kiểm nghiệm UI, người dùng ngoài nhóm, offline toàn
hành trình, hay đầy đủ cổng bảo mật của runtime gốc.

## 6. G1–G3 và B1–B8

| Cổng | Trạng thái trong lần này |
|---|---|
| G1 | CHƯA ĐO toàn sản phẩm; workload chỉ dữ liệu giả, không đổi UI/giáo dục |
| G2 | CHƯA ĐẠT đối với phán quyết tự chủ: model KEEP khi thiếu cổng; điều phối đã bác |
| G3 | CHƯA ĐO toàn sản phẩm; phiên này không đọc/copy secret hay dữ liệu học sinh thật; confinement candidate có bằng chứng trực tiếp |

| Chiều | Kết quả và giới hạn |
|---|---|
| B1 | Chưa đo lại hành trình; lưu bền UI vẫn chưa nối |
| B2 | Chưa kiểm nguồn/chất lượng giáo dục lại |
| B3 | Chưa thao tác UI |
| B4 | Có bằng chứng composition/subagent thật và tests; chưa nghiệm thu toàn kiến trúc |
| B5 | Candidate 79/79 checks; store failure regression thật; chưa phủ đầy lỗi hệ thống |
| B6 | Live Wonder/Spark qua XuVi; không đo lại image/voice/offline |
| B7 | Tham vấn Meta và danh mục model phục vụ đúng điểm nghẽn; không quét research bên ngoài hay adopt model mới |
| B8 | Có trace specs → sinh → red/green → nháp, nhưng tự phán quyết chưa qua cổng |
| Tổng/delta | Không tính; bằng chứng một phần, chưa đủ benchmark tổng thể |

## 7. Artefakt và cách kiểm lại tại máy này

Code thử nghiệm: `app/evolution-candidate/` (Package.swift + Sources, bị ignore).
Bằng chứng: `app/evolution-evidence/` (log cha/con, draft nguyên văn, task spec,
output các lần thử, output tests, boundary JSON, SHA256 candidate; bị ignore).

```sh
swift run --package-path app/evolution-candidate xuvi-tests
```

Lệnh trên không gọi provider. Live chỉ qua CLI candidate với bản sao allowlist,
marker `.evolution-workspace`, consent `--confirm-remote`, và credentials qua
seam của ứng dụng. Không chạy live trên repo gốc chứa bí mật. Code sinh và log
không đưa vào commit; specs là contract để tái sinh, artefakt chỉ để đối chiếu.

## 8. Bước tiếp theo duy nhất và lý do theo Đạo

**Điểm nghẽn kế tiếp: chốt phán quyết bằng dữ liệu được kiểm độc lập.** Khi
cổng CHƯA ĐO/CHƯA ĐẠT, controller phải trả ĐỖ/không chuyển nhà kể cả model tự
viết KEEP. Thêm test phủ định, sinh từ spec đã bổ sung, đo cùng checklist.
Chỉ sau đó mới đo đủ benchmark tổng thể và quyết định kích hoạt candidate.
Lưu bền UI tiếp tục được đỗ; không mở thêm tính năng để che lỗi phán quyết.

Theo Đạo: sự sống và riêng tư được bảo vệ bằng dữ liệu giả/workspace hẹp;
trung thực buộc bác kết luận giữ vượt bằng chứng; log/spec/result giúp hệ học
tiếp được. Dừng đợt ở đây khi phát hiện lỗi cổng, không ép model kể thành công.

## Vệ sinh

Không sửa Đạo, kết quả cũ, runtime gốc; không commit/push/deploy/gửi tin nhắn.
Các file mới/thay đổi của đợt ở tầng được phép commit chỉ gồm specs và results.
`git diff --check` và kiểm Đạo được chạy khi kết thúc. Các thay đổi có trước
được giữ nguyên; candidate và bằng chứng sinh đều nằm trong vùng ignore.
