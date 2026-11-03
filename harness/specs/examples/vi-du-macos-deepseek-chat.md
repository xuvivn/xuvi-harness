# Ví dụ đã điền — Harness cá nhân: macOS native + DeepSeek + chat tools

> Đây là một bộ quyết định **hoàn chỉnh và nhất quán** theo mẫu [technology/quyet-dinh-hien-tai.md](../technology/quyet-dinh-hien-tai.md), để bạn hình dung thành phẩm của bước "điền công nghệ" trước khi viết code. Sao chép nó làm điểm xuất phát rồi sửa theo ý bạn. Mọi lựa chọn dưới đây đều thay được mà không đụng tới `core/`.

---

## Bảng tổng kết

| # | Quyết định | Lựa chọn |
|---|---|---|
| 1 | Ngôn ngữ + runtime | TypeScript trên Node, một monorepo chia package theo nhóm khả năng |
| 2 | Cơ chế module/plugin | Framework context/plugin kiểu Cordis (kho dịch vụ + inject + effect + sự kiện có kiểu) |
| 3 | Mô hình tiến trình | Host (lõi) một tiến trình + client; bản macOS đóng gói lõi làm tiến trình con của app |
| 4 | Provider/model mặc định | `deepseek-official` — một model nhanh làm mặc định, một model mạnh cho việc khó (tên cụ thể điền khi có danh mục chính thức) |
| 5 | Provider phụ | Một cổng tương thích đa-provider để dùng thêm 2–3 model của hãng khác; model local để thử ngoại tuyến |
| 6 | Nơi chạy model | API đám mây là chính; engine local là route tuỳ chọn |
| 7 | Bí mật | Tham chiếu dạng tên biến môi trường; nguồn: môi trường → `.env` dự án → kho người dùng |
| 8 | Mặt tiếp xúc chính | Ứng dụng macOS native (SwiftUI) nói chuyện với lõi qua IPC cục bộ có token; web chat giữ làm mặt phụ để dùng từ trình duyệt |
| 9 | Tool trong chat | File, shell, terminal, web, LSP, job nền, subagent, skill, hỏi-người |
| 10 | Tool ngoài | Một MCP server stdio cho tài liệu nội bộ; một MCP server HTTP cho tìm kiếm |
| 11 | Approval mặc định | Preset "hỏi trước khi ghi/chạy": đọc tự do, ghi file và chạy lệnh thì hỏi; headless dùng `never` |
| 12 | Lưu session | Mỗi session một file log nối thêm (JSONL, nén khi đóng), chỉ mục tìm kiếm SQLite dựng lại được |
| 13 | Sandbox | macOS: Seatbelt; Linux: bubblewrap; chế độ mặc định `workspace-write` |
| 14 | Phát hành | App macOS ký Developer ID + notarize; lõi cũng phát hành dạng gói để chạy headless trên máy Linux |
| 15 | Chiến lược ngữ cảnh | Lai: chỉ thị workspace nạp trước, nội dung file nạp đúng lúc bằng tool tìm/đọc theo đoạn; đếm token bằng tokenizer của provider; cắt kết quả tool khi lịch sử vượt ~70% cửa sổ, nén bằng model phụ rẻ hơn ở ~85%; ghi chú tiến độ là một file trong workspace; kết quả tool mặc định bị chặn ở mức ngắn và lấy thêm theo trang |

## Vài quyết định đáng chú ý và lý do

- **SwiftUI thuần thay vì bọc web client**: đánh đổi việc tự dựng UI stream/thẻ tool để lấy trải nghiệm macOS chuẩn và quyền hệ thống (Accessibility) cho computer-use sau này. Hệ quả đã chấp nhận: web chat là codebase UI thứ hai — cả hai đều chỉ là mặt tiếp xúc trên cùng feed sự kiện ([core/01, mục 4](../core/01-tong-quan-kien-truc.md)) nên không nhân đôi logic hội thoại.
- **Hai model của cùng một provider thay vì hai provider ngay từ đầu**: tuyến mặc định ổn định giúp fixture replay (core/14) nhỏ; provider phụ vào sau bằng đúng đường "thêm adapter" của [ai-models.md, mục 4](../technology/ai-models.md).
- **Chỉ mục SQLite nhưng log là JSONL**: tìm kiếm session cần toàn văn nhanh, nhưng nguồn sự thật vẫn là log nối thêm dễ đọc dễ khôi phục; chỉ mục xoá là dựng lại ([core/11, mục 6](../core/11-luu-tru-va-khoi-phuc.md)).

## Thứ tự xây theo bộ quyết định này

Đi đúng lộ trình [core/14](../core/14-lo-trinh-xay-dung-va-kiem-thu.md). Riêng với bộ quyết định này:

1. Giai đoạn 0–4 làm bằng TypeScript/Node, mặt headless trước — chưa đụng Swift.
2. Giai đoạn 6 cắm adapter provider mặc định; provider phụ để sau giai đoạn 8.
3. Giai đoạn 8 làm web chat trước (rẻ hơn để kiểm chứng presenter/thẻ tool), rồi mới làm SwiftUI dùng lại cùng hợp đồng sự kiện.
4. Ký số/notarize và App Sandbox chỉ vào ở bước phát hành — không để chúng chặn phát triển (nhưng quyết định "lõi là tiến trình con" phải giữ từ đầu vì nó định hình kênh IPC).

## Điều cố tình không làm trong lần này

- Không đa người dùng, không telemetry gửi ra ngoài.
- Không computer-use (để dành khi UI native ổn định).
- Không hạ cấp định dạng: bản mới ghi định dạng mới thì bản cũ từ chối mở, theo [core/11, mục 4](../core/11-luu-tru-va-khoi-phuc.md).
