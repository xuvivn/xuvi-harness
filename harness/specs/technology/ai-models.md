# Technology — Hỗ trợ model AI nào

> File này trả lời: harness **hỗ trợ những nhà cung cấp và model nào**, mỗi model có khả năng gì. Hợp đồng adapter mà mọi model phải tuân nằm ở [`core/05-model-gateway.md`](../core/05-model-gateway.md) — file này không định nghĩa lại hợp đồng, chỉ điền danh sách.
>
> Cách model thực thi lúc chạy (endpoint, streaming, retry, bí mật) thuộc [ai-model-runtime.md](ai-model-runtime.md).

---

## 1. Mẫu khai báo cho một nhà cung cấp

Mỗi nhà cung cấp bạn hỗ trợ khai một hàng như sau (sao chép mẫu):

| Trường | Điền gì |
|---|---|
| Provider id | Khoá định tuyến ổn định, ví dụ `deepseek-official` — một khi đã ghi vào log thì không đổi tên |
| Đường gọi | API chính thức của hãng / cổng tương thích OpenAI-style / SDK của hãng / engine local |
| Xác thực | API key (tên tham chiếu bí mật) / OAuth / tài khoản của nền tảng / không cần |
| Model mặc định | Model chọn khi người dùng chưa chọn |
| Cách liệt kê model | Danh mục tĩnh trong code / hỏi endpoint lúc chạy / người dùng tự khai |

## 2. Mẫu khai báo cho một model

Mỗi model (theo đúng provider của nó) cần các dữ kiện mà [core/05, mục 4](../core/05-model-gateway.md) gọi là *metadata đúng-tuyến*. Thiếu trường nào thì ghi "không rõ" — core cấm tự đoán:

| Trường | Ví dụ dạng giá trị |
|---|---|
| Model id | chuỗi provider cấp |
| Cửa sổ ngữ cảnh (token) | số, hoặc "không rõ" |
| Trần output mặc định | số, hoặc "theo provider" |
| Mức reasoning effort hỗ trợ | danh sách id theo thứ tự tăng dần + mức mặc định, hoặc "không hỗ trợ" |
| Modal đầu vào | text / +image / +file… |
| Cập nhật system prompt giữa hội thoại | `in-history` hay chỉ-đầu-phiên |
| Cập nhật tool giữa hội thoại | nạp trễ được hay khai lại toàn bộ |
| Giá/đơn vị tính (nếu cần cho hiển thị ngân sách) | theo tài liệu của provider |

## 3. Các họ nhà cung cấp điển hình (điểm bắt đầu, tự rút gọn theo nhu cầu)

| Họ | Ghi chú khi tích hợp |
|---|---|
| API chính thức của một hãng model *(tham chiếu: DeepSeek official)* | Thường có mở rộng riêng trên wire (trường cộng thêm ngoài message) — các mở rộng đó là plugin của adapter đó, không vào core |
| Cổng tương thích một chuẩn phổ biến *(tham chiếu: lớp adapter đa-provider kiểu pi-ai — OpenAI/Anthropic/Google… qua một adapter)* | Một adapter phục vụ nhiều provider; phân biệt bằng cấu hình route (endpoint + bí mật). Coi chừng: "tương thích" hiếm khi giống hệt ở streaming tool-call và reasoning |
| Model local (engine suy luận trên máy) | Không mạng, không bí mật; đổi lại cửa sổ ngữ cảnh và tốc độ là của máy người dùng. Khai trong [ai-model-runtime.md](ai-model-runtime.md) |
| Cổng tự host / gateway nội bộ | Như một route do cấu hình khai báo: harness không biết trước model nào — phân giải metadata bằng cách hỏi endpoint, thiếu thì bắt người dùng khai |

## 4. Quy tắc khi thêm một model/provider mới

1. Viết adapter tuân [core/05, mục 3](../core/05-model-gateway.md) — đặc biệt: arguments giữ chuỗi JSON thô, lỗi chuẩn hoá theo `code`, một lần gọi = một lần thử.
2. Khai metadata đúng-tuyến (mục 2 ở trên). Không có metadata → model không được chọn cho hội thoại (được liệt kê để khám phá thì được).
3. Thêm một fixture stream đã ghi của model đó vào bộ test replay (core/14, mục 2).
4. Không thêm bất kỳ nhánh `if provider == …` nào ngoài adapter và cấu hình.

## 5. Quyết định của lần này

| Provider id | Đường gọi | Xác thực | Model mặc định | Ghi chú |
|---|---|---|---|---|
| `meta` | API chính thức Meta (`https://api.meta.ai/v1`) | API key, tham chiếu bí mật duy nhất `META_API_KEY` | `muse-wonder-1.0` | Provider **duy nhất** của lần này. Không đăng ký adapter/provider nào khác; DeepSeek Harness chỉ là công cụ bootstrap sinh hệ, không nằm trong sản phẩm |

| Provider / Model | Vai trò | Context | Effort hỗ trợ | Modal | Ghi chú |
|---|---|---|---|---|---|
| `meta` / `muse-wonder-1.0` | **orchestrator** — agent chính: chẩn đoán, lập kế hoạch, việc phức tạp/mơ hồ/xuyên module, tích hợp kết quả worker, review, phán quyết | không rõ (chờ kiểm chứng tài liệu Meta chính thức — cấm đoán) | không rõ | text (+image/file nếu tuyến xác nhận) | Tuyến mặc định của mọi session người dùng và profile `evolution` |
| `meta` / `muse-spark-1.3-contributor` | **worker** — agent con việc tự chứa có ranh giới; đồng thời là tuyến `aux` (tiêu đề session, tóm tắt nén) | không rõ (như trên) | không rõ | text | Chỉ được chọn qua preset agent con/`aux`, sau khi `GET /v1/models` tại repo này xác nhận còn tồn tại; không âm thầm fallback |
| `meta` / `muse-image-1.0` | capability Waldorf (không phải agent) | — | — | text→image | Sinh tranh cảnh; lỗi provider hiển thị thành thật, không giả thành công |
| `meta` / `muse-voice-transcribe-1.0` | capability Waldorf (không phải agent) | — | — | audio→text | `POST /v1/asr/transcribe`, WAV mono PCM16 16/24kHz; xem `plugins/waldorf/specs/03_research_and_architecture.md` §2 |

- Model dùng cho việc phụ (tóm tắt nén, đặt tiêu đề session…): tuyến worker `muse-spark-1.3-contributor` với `purpose: aux` qua cùng gateway — không mở tuyến text thứ ba.
- Lý do một dòng: một provider, hai vai rõ ràng — Wonder giữ mạch và phán quyết, Spark Contributor gánh việc con cách ly ngữ cảnh; đúng nhu cầu người dùng và hợp đồng subagent ở core/10.

## 6. Ảnh hưởng lên core

- Không đổi core. Mọi khác biệt giữa các model phải nằm trong adapter + metadata.
- Việc phụ (nén/tiêu đề) gọi model qua cùng gateway với một `purpose` đã khai báo (core/05, mục 2) — không mở đường gọi riêng.
