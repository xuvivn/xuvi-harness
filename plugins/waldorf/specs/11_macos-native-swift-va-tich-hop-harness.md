# 11 — Waldorf trên macOS native thuần Swift, tích hợp vào XuVi Harness (bootstrap rồi tự chủ)

> File này ghi quyết định của người dùng ngày 2026-11-03 và **thay thế (supersede)**
> hai điểm của các spec trước cho đường đi từ nay:
>
> 1. Stack React/TypeScript + Node + SQLite làm *sản phẩm chạy* trong
>    `03_research_and_architecture.md` — giữ làm lịch sử và nguồn schema/nghiệp vụ
>    để port, không còn là stack của sản phẩm.
> 2. Điều "không xây harness generic trong đợt đầu" trong
>    `10_loop-dau-tien-hackathon.md` — đợt bootstrap này xây **cả harness lẫn
>    Waldorf tích hợp**, vì mục tiêu là chuyển hẳn sang xuvi-harness.
>
> File này đứng dưới `00_master_prompt.md` và dưới Đạo. Contract Meta
> (endpoint, model, định dạng) giữ nguyên như đã ghi ở `03` §2 và kiểm chứng ở
> `08` — nhưng mọi mức "đã kiểm chứng" phải chạy lại tại repo này mới tính.

## 1. Quyết định

| # | Quyết định | Giá trị |
|---|---|---|
| 1 | Dạng sản phẩm | Ứng dụng **macOS native thuần Swift** (SwiftUI/AppKit), một SwiftPM package trong `app/` (vùng sinh, bị ignore). Không web server, không WKWebView |
| 2 | Quan hệ với harness | Waldorf là **plugin chạy trên XuVi Harness**: đăng ký domain service, tool và màn hình vào harness; không tự sở hữu agent/session/model gateway riêng |
| 3 | Provider/model | Duy nhất Meta, duy nhất bí mật `META_API_KEY`. **Orchestrator `muse-wonder-1.0`** (agent chính, việc phức tạp, góp ý chuyên sâu, phán quyết), **worker `muse-spark-1.3-contributor`** (agent con việc tự chứa; chia cảnh/dàn ý nhanh; tuyến `aux`), capability `muse-image-1.0` (tranh cảnh), `muse-voice-transcribe-1.0` (phiên âm) |
| 4 | Ai xây lượt đầu | DeepSeek Harness sinh specs → code → test trong đợt bootstrap này. Qua cổng dogfood ở §5 thì **nghỉ hẳn DeepSeek Harness**; mọi loop sau chạy bằng chính xuvi-harness (profile `evolution`, orchestrator là Wonder của XuVi) |
| 5 | Phát hành v1 | `.app` ký ad-hoc chạy trên máy này. Developer ID/notarize/App Sandbox: đỗ lại |
| 6 | Lưu trữ Waldorf | JSON Codable ghi nguyên tử + backup qua seam persistence (v1); schema/nghiệp vụ port từ `03` §4. SQLite chỉ khi quy mô/truy vấn đòi, qua đúng seam |
| 7 | Xuất hồ sơ v1 | Xem trước trong app + xuất Markdown/HTML và in/PDF qua AppKit. **DOCX đỗ lại**, ghi điều kiện quyết tiếp ở §6 — không giả là đã có |

## 2. Mô hình orchestrator / worker áp vào harness và Waldorf

Hợp đồng đầy đủ ở `harness/specs/core/10` §1 và bảng quyết định trong
`harness/specs/technology/ai-models.md` §5. Tóm tắt bất biến:

- Orchestrator (Wonder) là agent chính của session: chẩn đoán, lập kế hoạch,
  tự làm việc phức tạp/mơ hồ/xuyên module, tích hợp, review, phán quyết.
- Worker (Spark Contributor) là agent con **session riêng, ngữ cảnh sạch**:
  chỉ nhận việc tự chứa gồm đủ mục tiêu · định dạng kết quả · hướng dẫn
  tool/nguồn · ranh giới. Cha chỉ nhận bản chưng cất + tham chiếu, không nhận
  transcript thô của con.
- Trần v1: độ sâu 1 (worker không sinh con), ≤ 3 worker song song, ngân sách
  token/tool + timeout riêng từng worker; huỷ cha thì con dừng theo.
- Trong Waldorf: chia cảnh/dàn ý = worker; góp ý kể chuyện chuyên sâu, soạn
  brief phức tạp = orchestrator; tranh/phiên âm = model capability, không
  phải agent. Sinh 6 hồ sơ là code cục bộ xác định từ dữ liệu hiện hành,
  không cần model — model chỉ hỗ trợ soạn thảo khi người dùng yêu cầu.
- Tuyến worker chỉ được chọn sau khi `GET /v1/models` chạy tại repo này xác
  nhận `muse-spark-1.3-contributor` còn tồn tại. Không xác nhận được thì
  fail loud, **không âm thầm fallback** sang model khác.

## 3. Phạm vi đợt bootstrap (3 loop, mỗi loop vẫn qua đủ pha của `cron.md`)

| Loop | Điểm nghẽn chính | Đầu ra |
|---|---|---|
| A | Chưa có harness chạy được tại đây (B4/B5) | Harness core Swift ở mức headless: module system, session log JSONL, model gateway (replay trước, Meta sau), agent loop, tool file/shell/todo, **subagent tối thiểu** (orchestrator → worker), profile `headless` + `evolution`; kịch bản replay trọn vẹn không cần key |
| B | Sản phẩm Waldorf chưa chạy native tại đây (B1/B3) | `.app` SwiftUI: màn hình Chat/agent (thấy cây orchestrator → worker) + Waldorf (Story trọn vòng, 6 hồ sơ, lịch tuần, nhật ký) dùng chung dữ liệu; luồng thủ công trọn khi không key/mạng |
| C | Bằng chứng và chuyển nhà (B6/B8) | Test đủ tầng + live Meta bằng dữ liệu tổng hợp (gồm một uỷ quyền thật Wonder → Spark), benchmark mới so với baseline 32.5, rồi loop dogfood ở §5 |

## 4. Hành trình Waldorf bắt buộc (giữ nguyên mốc của `10` §1/§4)

Mở app → chọn/nhập truyện dân gian (duyệt văn bản trước lưu) → bản đồ cảnh
sửa tay được → luyện kể 3 chế độ (gợi ý / mờ dần / tự do) → ghi âm hoặc tải
audio → sửa transcript → góp ý có dẫn chứng, **không điểm tổng**, phân biệt
bỏ sót với chuyển thể có chủ ý → lịch sử có snapshot đóng băng. Sáu loại hồ
sơ của `00_master_prompt.md` sinh từ cùng dữ liệu hiện hành, xem trước và
xuất/in được. Mọi lần gửi dữ liệu ra Meta cần `confirmRemote: true` tường
minh kèm disclosure; tắt key/mạng vẫn hoàn thành toàn bộ bằng thao tác thủ
công; fallback cục bộ gắn nhãn "phân tích cục bộ — không phải AI".

Trung thực nguồn: thư mục `references/` (PDF MOET…) **không có trong repo
này**. Không bịa trích dẫn/trang MOET; phần đối chiếu GDPT hiển thị đúng
trạng thái chưa-đối-chiếu/chưa-kiểm-chứng cho tới khi nguồn thật được nạp.

## 5. Cổng chuyển nhà (dogfood) — định nghĩa "tests oke thì bỏ DeepSeek Harness"

**Làm rõ trước lần nghiệm thu tiếp theo (2026-11-03):** contract thực thi ở
`harness/specs/technology/evolution-dogfood.md`. Trong môi trường chỉ có Command
Line Tools, các chữ `swift test` ở §5/§7 dùng custom runner
`swift run --package-path app xuvi-tests` với cùng phạm vi hành vi bắt buộc.
Không hồi tố: smoke task 17 + 25 của bootstrap chưa đạt cổng này.

Chỉ khi bộ test của đợt xanh, chạy **một loop tiến hoá thật bằng chính
xuvi-harness** (profile `evolution`; approval `never` chỉ trong bản sao
workspace dùng-một-lần, sandbox-exec bật):

1. Orchestrator (Wonder của XuVi) tự đọc Đạo/README/`cron.md` + specs liên
   quan bằng tool của nó (JIT), lập todo, viết giả thuyết một điểm nghẽn.
2. Việc thật trọn vòng trong bản sao: orchestrator tự làm phần tích
   hợp/phức tạp, **uỷ quyền ít nhất một việc con cho worker Spark
   Contributor** đúng hợp đồng 4 thành phần, review bản chưng cất (mở tham
   chiếu kiểm chứng ít nhất một chi tiết), tích hợp, tự chạy `swift test`.
3. Toàn bộ quá trình chỉ gọi `api.meta.ai` bằng `META_API_KEY` (kiểm từ log
   đã redact: một host, provider `meta`, đúng vai theo model); session log
   của cha và con tái dựng được mọi request.
4. Orchestrator tự soạn nháp file kết quả benchmark của loop đó.

Đạt cả 4 → ghi benchmark + `benchmark/results/INDEX.md`, từ đó mọi loop
chạy bằng xuvi-harness. Không đạt → ghi phán quyết "đỗ lại" kèm điều kiện
thiếu vào file kết quả; **không tuyên bố đã chuyển**.

## 6. Đỗ lại có chủ đích

DOCX (quyết tiếp khi có nhu cầu gửi file Word thật hoặc một thư viện/đường
sinh DOCX được kiểm chứng trong Swift), notarize/Developer ID + App Sandbox
(quyết khi phát hành ra máy khác), XCUITest (quyết khi có Xcode full), MCP,
workflow/agent-teams/độ sâu uỷ quyền > 1, fork ngữ cảnh cho worker, SQLite
(quyết khi dữ liệu Waldorf lớn tới mức JSON chậm/ khó truy vấn — đo trước),
model local/provider thứ hai, đa nền tảng, đồng bộ/đa người dùng.

## 7. Bằng chứng phải đạt trong đợt (khai thật mức)

- `swift test` xanh không cần key: unit + composition + replay cả hai tuyến
  + test cách ly worker (cha chỉ nhận chưng cất) + phủ định an toàn
  (path traversal, approval từ chối, backup hỏng) + persistence/kill.
- Live (tách riêng, chỉ dữ liệu tổng hợp/hư cấu, có đồng ý gửi): models,
  chat Wonder, uỷ quyền Wonder → Spark, ảnh, phiên âm WAV tổng hợp.
- UI: kiểm thủ công theo kịch bản demo (mức `thủ công`, trần B3 tương ứng)
  cho tới khi có Xcode để chạy XCUITest.
- Benchmark: file kết quả mới trong `benchmark/results/`, không sửa file cũ;
  điểm từng chiều kèm trần bằng chứng; qua 3 cổng G1/G2/G3 mới giữ thay đổi.
