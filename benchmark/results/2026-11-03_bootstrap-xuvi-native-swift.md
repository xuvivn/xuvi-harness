# Kết quả benchmark — 2026-11-03 — đợt bootstrap: XuVi Harness macOS native thuần Swift + Waldorf

- **Thời điểm đo:** 2026-11-03 (Asia/Saigon), cuối đợt bootstrap trên DeepSeek Harness
- **Ai đo:** agent phiên này (DeepSeek Harness), chạy trực tiếp trong repo `xuvi-harness`
- **Đối tượng đo:** code sinh ra trong `app/` (SwiftPM, bị ignore): harness core Swift
  (`XuviCore`), adapter Meta duy nhất (`XuviMeta`), plugin Waldorf (`XuviWaldorf`),
  CLI headless (`xuvi`), test runner tự chứa (`xuvi-tests`), app SwiftUI (`XuViApp`
  đóng gói `dist/XuVi.app` ký ad-hoc)
- **Baseline so sánh:** `2026-11-03_baseline-truoc-loop-1.md` — tổng 32.5, cổng ?/?/?
- **Giả thuyết của đợt:** nếu chốt specs công nghệ (Swift thuần, chỉ Meta,
  Wonder orchestrator / Spark Contributor worker) rồi sinh harness + Waldorf tích hợp
  từ specs, thì B1/B4/B5/B6 tăng có bằng chứng tại đây mà không cổng Đạo nào regress
- **Phán quyết:** **GIỮ** — và đạt cổng dogfood (mục cuối), đủ điều kiện chuyển hẳn
  sang xuvi-harness theo `plugins/waldorf/specs/11` §5

## Điều đã làm (3 loop của đợt, theo spec 11 §3)

- **Pha 0 (specs trước):** điền đủ 15 ô `harness/specs/technology/quyet-dinh-hien-tai.md`
  và phần quyết định trong 6 file technology liên quan; thêm
  `plugins/waldorf/specs/11_macos-native-swift-va-tich-hop-harness.md`; ghi vết
  supersede vào `plugins/waldorf/specs/10_loop-dau-tien-hackathon.md` §2b.
  `ĐẠO.md` không diff.
- **Loop A:** harness core headless — module system (inject/disposer), session log
  JSONL + `deriveMessages`, model gateway (replay adapter trước, Meta adapter sau),
  agent loop turn/step, tool file/glob/grep/shell/todo, **subagent tối thiểu**:
  orchestrator uỷ quyền worker theo hợp đồng 4 thành phần (core/10 §1), cha chỉ
  nhận bản chưng cất + tham chiếu session con.
- **Loop B:** app SwiftUI (Chat/agent thấy uỷ quyền gần nhất, XuVi Story với bản đồ
  cảnh + góp ý cục bộ, 6 loại hồ sơ, lịch tuần, cài đặt/bằng chứng) + domain Waldorf
  port từ spec `03` (chia cảnh, góp ý không điểm tổng, timetable 2100 phút, 6 hồ sơ
  đủ trường, store JSON nguyên tử + backup).
- **Loop C:** đo như dưới đây; đóng gói `.app` ad-hoc, mở app thật trên máy này
  (tiến trình chạy và thoát sạch qua lệnh quit).

## Cổng veto

| Cổng | Trạng thái | Bằng chứng |
|---|---|---|
| G1 Đạo & an toàn sự sống | ĐẠT | Góp ý kể chuyện không có điểm tổng (test tự động xác nhận `overallScore` vắng mặt cả khi encode); seed chỉ dữ liệu minh họa hư cấu gắn nhãn; không tính năng gây áp lực/gây nghiện cho trẻ |
| G2 Trung thực bằng chứng | ĐẠT | Phân tích cục bộ gắn nhãn "không phải AI"; ma trận GDPT ghi rõ chưa có nguồn/chưa đối chiếu và không tuyên bố đạt chuẩn; DOCX khai đỗ lại; mức bằng chứng từng chiều khai ở bảng dưới, không nâng mức |
| G3 Riêng tư & bí mật | ĐẠT | Key chỉ nằm trong bộ nhớ tiến trình của adapter (CLI/app tự nạp `.env`, agent không đọc giá trị); log dogfood kiểm trực tiếp chỉ chứa provider/model/request, không chứa key; mọi kiểm chứng live dùng dữ liệu tổng hợp/hư cấu (câu số học, prompt tranh tổng hợp, giọng Linh tổng hợp của macOS); không dữ liệu học sinh thật |

## Điểm từng chiều

| Chiều | Điểm (0–4) | Mức bằng chứng cao nhất | Bằng chứng (chạy tại repo này, đợt này) | Delta vs baseline |
|---|---|---|---|---|
| B1 Giá trị & trọn vẹn sản phẩm (20%) | 2 | tự động + thủ công | App mở được; Story có bản đồ cảnh, luyện kể + góp ý cục bộ, snapshot đóng băng (test); 6 hồ sơ sinh đủ trường từ cùng dữ liệu (test + `xuvi seed-waldorf` OK cả 6). Chưa đạt 3: ghi âm trong UI và lưu bền dữ liệu Waldorf từ UI chưa nối (app đang dùng seed trong bộ nhớ), xuất mới ở Markdown/in | +1 |
| B2 Giáo dục & văn hoá (15%) | 2 | thủ công | Nội dung seed đúng tinh thần kể chuyện/không điểm tổng, gắn nhãn minh họa, không tuyên bố chứng nhận; chưa lấy mẫu provenance MOET vì `references/` không có trong repo — đối chiếu hiện đúng trạng thái chưa-kiểm-chứng | 0 |
| B3 Trải nghiệm người dùng (10%) | 2 | thủ công | `.app` native mở trên máy này, 5 màn hình chính hoạt động ở mức tiến trình + điều hướng có trong code đã build. Trần bị giữ ở 2: **chưa kiểm chứng trực quan** (screencapture bị từ chối quyền trong phiên này), chưa đi kịch bản màn hình hẹp/bàn phím | +1 |
| B4 Toàn vẹn kiến trúc harness (15%) | 3 | tự động | 44 checks của `xuvi-tests` đạt, gồm: gỡ plugin → đăng ký biến mất; log suy ra đúng thứ model thấy; thay tuyến bằng cấu hình (replay/meta); uỷ quyền worker — log cha không chứa tool call của con, chỉ chưng cất + tham chiếu; từ chối uỷ quyền thiếu thành phần; cắt kết quả tool kèm tham chiếu | +1 |
| B5 Tin cậy kỹ thuật & bảo mật cục bộ (15%) | 3 | tự động | `swift build` xanh (0 lỗi, gồm release cho `.app`); test phủ định: path traversal bị chặn, approval từ chối → tool không chạy và được ghi log, backup hỏng → khôi phục từ bản sao, log rách dòng cuối vẫn đọc được tiền tố; store Waldorf sống qua khởi động lại (test) | +2 |
| B6 AI thật & chọn model (10%) | 3 | **live** | Qua chính code Swift của repo này: `xuvi models` liệt kê đủ 4 model bắt buộc (chỉ in id); **dogfood live đạt** — Wonder orchestrator uỷ quyền Spark Contributor worker ghi `result.txt = 42`, tự kiểm chứng bằng `read_file`, session log cha/con tách riêng đúng model; `xuvi capability-smoke`: ảnh WebP 648.200 byte (sniff RIFF) qua `muse-image-1.0`, phiên âm giọng Linh tổng hợp qua `muse-voice-transcribe-1.0` thu lại câu tiếng Việt gần nguyên văn. Chưa đạt 4: ảnh/phiên âm mới qua CLI adapter, chưa nối vào màn hình Waldorf | +2 |
| B7 Nghiên cứu & học hỏi (5%) | 2 | tự động | Danh mục model Meta kiểm lại live ngày 2026-11-03 tại đây; quyết định adopt (Wonder/Spark Contributor/Image/Voice) — reject (mọi provider khác, model local) ghi trong specs technology + spec 11 kèm lý do; chưa có đợt research theo nguồn đăng ký riêng | +1 |
| B8 Sức khoẻ tiến hoá & vệ sinh specs (10%) | 3 | tự động | Specs → code → kết quả này truy vết được; `git status` cuối đợt chỉ gồm specs + benchmark/results thuộc diện commit (code/`app/` bị ignore đúng luật); loop dogfood để lại session log cha + con đọc lại được. Chưa đạt 4: chưa có người ngoài nhóm xây tự dùng | +2 |
| **Tổng có trọng số (/100)** | **62.5** | — | 10 + 7.5 + 5 + 11.25 + 11.25 + 7.5 + 2.5 + 7.5 | **+30.0** |

## Độ lệch so với plan đã duyệt — khai thật

1. **Test runner tự chứa thay cho XCTest:** máy chỉ có Command Line Tools, không có
   XCTest/Swift Testing (chúng đi kèm Xcode full). Bộ 44 checks chạy bằng executable
   `xuvi-tests` (assertion thuần, exit ≠ 0 khi fail) — vẫn là bằng chứng `tự động`,
   nhưng chưa phải XCTest. Chuyển sang XCTest khi có Xcode; không chặn cổng chuyển nhà
   vì tiêu chí là hành vi test, không phải tên framework.
2. **Waldorf store là JSON thay cho SQLite:** chọn ở tầng specs (đơn giản hơn, ít cam
   kết hơn theo Đạo); SQLite đỗ lại qua seam persistence khi quy mô đòi.
3. **Tên product app đổi `XuVi` → `XuViApp`:** filesystem macOS không phân biệt
   hoa/thường làm product CLI `xuvi` và app `XuVi` ghi đè cùng một file — đã sửa và
   build lại sạch. Bài học ghi vào đây thay vì sửa kết quả cũ.

## Research đã quét trong đợt

| Nguồn | Ngày kiểm tra | Điều mới | Quyết định |
|---|---|---|---|
| `GET https://api.meta.ai/v1/models` (qua `xuvi models`) | 2026-11-03 | Có đủ `muse-wonder-1.0`, `muse-spark-1.3-contributor`, `muse-image-1.0`, `muse-voice-transcribe-1.0` (cùng `muse-spark-1.3`, `sam-3.1`…) | adopt đúng 4 model ở spec 11; reject SAM/Spark Open/Glimmer/hosting bên thứ ba cho v1 (giữ quyết định của spec `03`) |

## Cổng dogfood và chuyển nhà (spec 11 §5)

Chạy `xuvi dogfood-live` trong workspace tạm dùng-một-lần
(`…/T/xuvi-dogfood-873C21DF-…`, còn trên máy để đối chiếu):

1. ✅ Orchestrator là **Wonder của chính XuVi** (log cha: `model: muse-wonder-1.0`),
   tự đọc workspace bằng tool của nó.
2. ✅ Uỷ quyền thật cho worker **`muse-spark-1.3-contributor`** (log con riêng
   `worker-D034108E-….jsonl`): worker đọc `notes.txt`, ghi `result.txt = 42`;
   orchestrator dùng `read_file` kiểm chứng rồi mới kết luận ("Đã kiểm chứng bằng
   read_file: result.txt chứa 42").
3. ✅ Toàn bộ chỉ gọi `api.meta.ai` bằng `META_API_KEY`; log cha 3 assistant message /
   2 tool call (delegate + read_file), log con tách biệt — cha chỉ nhận chưng cất.
4. ✅ Bản nháp kết quả này do phiên bootstrap soạn từ bằng chứng của loop dogfood;
   từ loop sau, việc soạn nháp này là của orchestrator XuVi (profile `evolution`).

**Kết luận chuyển nhà: ĐẠT.** Từ đây mọi loop của repo này chạy bằng xuvi-harness
(mở `app/dist/XuVi.app` hoặc `app/.build/debug/xuvi`, workspace là repo này);
DeepSeek Harness chỉ còn là công cụ đã sinh ra hệ ở đợt này. Điều kiện quay lại
(rollback): nếu một loop dogfood sau này thất bại cổng nào ở §5 của spec 11, ghi
"đỗ lại" và quay về chẩn đoán ở specs — không lặng lẽ quay lại DeepSeek Harness
như thể chưa từng chuyển.

## Bài học và điểm nghẽn kế tiếp (cho loop đầu tiên của xuvi-harness)

- Điểm nghẽn chính: **B1/B3** — nối nốt hành trình Waldorf trong app (ghi âm →
  phiên âm qua adapter ngay trong UI, lưu bền dữ liệu Waldorf từ UI, kiểm trực quan
  + màn hình hẹp) để B1 lên 3 và B3 có bằng chứng thủ công đầy đủ.
- Phụ (chỉ khi chặn điểm chính): nối ảnh/tranh cảnh vào màn hình Story (adapter đã
  live qua CLI); XCTest khi có Xcode; DOCX khi có nhu cầu gửi Word thật.
- Ngân sách: đợt dừng ở đây theo điều kiện dừng của `cron.md` — mốc đã đạt, phần
  token còn lại là tiết kiệm cho đợt sau, không phải nghĩa vụ phải đốt.

## Vệ sinh commit

- `git status` tại thời điểm kết thúc: chỉ specs/root whitelist + `benchmark/results/**`
  (kiểm lại ngay trước khi kết thúc đợt và ghi vào INDEX)
- ĐẠO.md có diff không: không (bắt buộc)
