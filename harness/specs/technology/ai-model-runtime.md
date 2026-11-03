# Technology — Runtime của model AI (model chạy thế nào lúc thực thi)

> [ai-models.md](ai-models.md) quyết định *hỗ trợ model nào*. File này quyết định *mỗi lần gọi model thực sự chạy ra sao*: gọi đi đâu, bằng giao thức gì, bí mật lấy ở đâu, chờ bao lâu, thử lại thế nào, đếm token bằng gì. Hợp đồng logic vẫn là [`core/05`](../core/05-model-gateway.md).

---

## 1. Nơi thực thi

| Lựa chọn | Ý nghĩa | Hệ quả |
|---|---|---|
| **API đám mây của hãng** *(tham chiếu)* | Mỗi yêu cầu là một HTTP request streaming tới endpoint của provider | Cần mạng + bí mật; độ trễ và giá theo provider |
| Cổng trung gian (gateway/proxy tự quản) | Harness gọi một endpoint duy nhất, cổng định tuyến tiếp | Một điểm cấu hình endpoint cho nhiều provider; cổng trở thành một phần của bề mặt tin cậy |
| **Engine local** | Model chạy trên máy người dùng (tiến trình con hoặc thư viện suy luận) | Không bí mật, riêng tư; adapter phải tự quản vòng đời tiến trình/model và ánh xạ khả năng hạn chế của engine vào hợp đồng core/05 (ví dụ: không có reasoning tách bạch → khối reasoning vắng mặt, không bịa) |
| Hỗn hợp theo route | Route nào khai endpoint local thì chạy local, còn lại lên cloud | Cấu hình route quyết định; core không biết |

## 2. Các quyết định vận hành (điền cho từng đường gọi)

| Quyết định | Lựa chọn điển hình | Mặc định tham chiếu |
|---|---|---|
| Giao thức streaming | SSE / chunked JSON / WebSocket / pipe từ tiến trình con | Theo adapter của provider; ánh xạ về `StreamChunk` của core/05 |
| Timeout treo luồng (không chunk mới) | Số ms cấu hình được | 5 phút, dương và hữu hạn |
| Trần chờ toàn yêu cầu | Số ms hoặc "không trần ngoài trần output" | Theo triển khai |
| Chính sách retry | `{maxRetries, retryableCodes, backoff, jitter}` hoặc chế độ luôn-thử-lại cho chạy dài không người trông | Thường: 5 lần, chỉ retry code tạm thời; `Retry-After` của provider được tôn trọng |
| Ai được retry | Chỉ tầng trên adapter (core/05, mục 3.4) — adapter tắt retry của thư viện | Bắt buộc |
| Đếm token cho đo áp lực ngữ cảnh | Bộ đếm của provider / tokenizer xấp xỉ / đếm từ usage đã ghi | Đo từ log + usage thật làm mốc (core/10, mục 6.1) |
| Cache phía provider | Khai báo cho adapter biết trường nào của yêu cầu phá cache, để giữ tiền tố ổn định | System prompt và header tách bạch (core/04, mục 5) phục vụ đúng mục đích này |
| Đồng thời tối đa | Số yêu cầu model song song mỗi agent / toàn cục | Agent con và workflow có ngân sách riêng (core/10) |
| Ngưỡng áp lực ngữ cảnh | Mức % cửa sổ (theo bộ đếm ở trên) tại đó từng bậc xử lý của core/07, mục 6.2 được kích hoạt | Khai ngưỡng cho ít nhất bậc cắt kết quả tool và bậc nén; bậc nào không có provider thì ghi "không bật" thay vì để mặc định ngầm |
| Model tóm tắt nén | Model dùng cho lần gọi tóm tắt của compaction (core/10, mục 6.1) | Thường là model rẻ/nhanh hơn model chính; vẫn qua cùng gateway với `purpose` đã khai |

## 3. Bí mật và danh tính

- Bí mật đi qua seam credentials ([core/08, mục 2.11](../core/08-capability-seams.md)): cấu hình chỉ chứa **tham chiếu** (ví dụ tên biến môi trường). Điền cho mỗi provider: tên tham chiếu, lớp nguồn ưu tiên (môi trường tiến trình → file `.env` của dự án → kho người dùng), và đường OAuth nếu có.
- Mọi yêu cầu mang định danh ứng dụng tĩnh (product/version) — không mang định danh người dùng (core/05, mục 3.9).
- Log chẩn đoán được ghi: provider, model, request id của provider, mã lỗi. Không bao giờ ghi: bí mật, toàn văn prompt trong log chẩn đoán vận hành (prompt đã có trong session log theo quyền của người dùng).

## 4. Quyết định của lần này

- Nơi thực thi mặc định: API đám mây Meta, gọi trực tiếp từ adapter trong tiến trình app/CLI.
- Endpoint/gateway (nếu tự quản): `https://api.meta.ai/v1` (chat-completions tương thích cho text; `/images/generations`; `/asr/transcribe`). **Allowlist mạng: code sản phẩm chỉ gọi host này**; override base URL (env `META_API_BASE`) chỉ để trỏ mock server localhost trong test.
- Timeout treo luồng: 5 phút không chunk mới (cấu hình được); huỷ của người gọi luôn thắng.
- Chính sách retry: tầng trên adapter, tối đa 5 lần, chỉ mã tạm thời, tôn trọng `Retry-After`; adapter tắt retry của thư viện bên dưới, một lần gọi adapter = một lần thử.
- Tham chiếu bí mật cho từng provider: chỉ một — `META_API_KEY` cho provider `meta`. Nguồn theo thứ tự: biến môi trường tiến trình → `.env` do chính app/CLI nạp lúc runtime (agent làm việc không đọc giá trị) → Keychain của chính app khi người dùng tự nhập trong Settings. Key không vào config, log, UI (ngoài trường nhập bảo mật), báo cáo hay backup.
- Làm rõ cho app theo yêu cầu nhập trên UI: khóa đã lưu từ Cài đặt được ưu tiên
  trước env/.env, để thao tác thay khóa có hiệu lực. CLI không inject store giữ
  thứ tự cũ. Contract ở [cau-hinh-meta-tren-ui.md](cau-hinh-meta-tren-ui.md).
- Giới hạn đồng thời: trần uỷ quyền của orchestrator: độ sâu 1, tối đa 3 worker song song; mỗi worker có ngân sách token/tool-call và timeout riêng (cấu hình trong profile, không viết cứng).
- Ngưỡng áp lực ngữ cảnh theo bậc: bậc 1 cắt kết quả tool quá trần (mặc định 8000 ký tự, phần dư truy hồi bằng tham chiếu); bậc 2 uỷ quyền agent con ngữ cảnh sạch; bậc 3 nén bằng tuyến `aux` — chỉ bật khi đã có bộ đếm token từ log + usage thật, và metadata cửa sổ của tuyến đã kiểm chứng (chưa rõ thì không tự đặt ngưỡng theo % cửa sổ).
- Model tóm tắt nén: `muse-spark-1.3-contributor` (`purpose: aux`), cùng gateway.
- Lý do một dòng: một host, một bí mật, hai vai model — bề mặt tin cậy nhỏ nhất để kiểm chứng được tuyên bố "chỉ dùng META_API_KEY".

## 5. Ảnh hưởng lên core

- Không đổi core. Runtime chỉ là cách adapter thực hiện hợp đồng [core/05](../core/05-model-gateway.md).
- Nếu engine local không đáp ứng được một điều khoản của hợp đồng (ví dụ không tách được reasoning): adapter khai báo thiếu khả năng trong metadata thay vì mô phỏng — consumer hạ cấp theo dữ kiện, không theo tên engine.
