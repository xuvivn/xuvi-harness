# Specs — Xây lại một AI Agent Harness từ đầu

Bộ tài liệu này mô tả **cách một agent harness hoạt động**, được tách ra từ kiến trúc của DeepSeek Harness, nhưng viết lại ở mức **generic (trung lập công nghệ)** để bạn có thể tự xây một harness cá nhân theo ý mình.

> **Harness là gì?** Là phần mềm bao quanh mô hình AI: nó nhận yêu cầu của người dùng, gọi mô hình, cho mô hình dùng công cụ (đọc file, chạy lệnh, tìm web…), ghi lại toàn bộ quá trình, hỏi ý kiến người dùng khi cần, và hiển thị kết quả. Mô hình chỉ "suy nghĩ và trả lời" — mọi việc còn lại là của harness.

---

## Cách tổ chức: hai lớp tách bạch

```
specs/
├── README.md                  ← bạn đang ở đây
├── core/                      ← LOGIC CỐT LÕI, không phụ thuộc công nghệ
│   ├── 01-tong-quan-kien-truc.md
│   ├── 02-module-va-plugin.md
│   ├── 03-agent-loop.md
│   ├── 04-session-log.md
│   ├── 05-model-gateway.md
│   ├── 06-tools-va-thuc-thi.md
│   ├── 07-prompt-va-ngu-canh.md
│   ├── 08-capability-seams.md
│   ├── 09-tuong-tac-nguoi.md
│   ├── 10-phoi-hop-va-mo-rong.md
│   ├── 11-luu-tru-va-khoi-phuc.md
│   ├── 12-composition-va-cau-hinh.md
│   ├── 13-bao-mat-loi-va-quan-sat.md
│   ├── 14-lo-trinh-xay-dung-va-kiem-thu.md
│   └── 15-ban-do-doi-chieu-deepseek-harness.md
├── technology/                ← QUYẾT ĐỊNH CÔNG NGHỆ, thay được mỗi lần chạy
│   ├── README.md
│   ├── quyet-dinh-hien-tai.md      ← file BẠN điền trước mỗi lần xây/chạy
│   ├── runtime-va-ngon-ngu.md
│   ├── ai-models.md
│   ├── ai-model-runtime.md
│   ├── chat-va-tools-integration.md
│   ├── ung-dung-macos-native.md
│   └── nen-tang-luu-tru-va-dong-goi.md
└── examples/
    └── vi-du-macos-deepseek-chat.md  ← một bộ quyết định đã điền đầy đủ
```

**Quy tắc vàng:**

1. File trong `core/` **không** nhắc tên một ngôn ngữ, framework, hệ điều hành hay nhà cung cấp AI cụ thể. Nếu một chi tiết công nghệ lọt vào `core/`, đó là lỗi của tài liệu.
2. Mọi chi tiết công nghệ nằm trong `technology/`. Muốn đổi công nghệ → chỉ sửa/thay file trong `technology/`, **không sửa `core/`**.
3. `core/` định nghĩa *giao diện (interface) và quy tắc*. `technology/` chọn *cài đặt cụ thể* cho các giao diện đó.

---

## Đọc theo thứ tự nào?

### Nếu bạn mới bắt đầu (đọc để hiểu)

Đọc `core/` theo số thứ tự 01 → 08. Đó là xương sống. Các file 09–13 đọc khi cần tính năng tương ứng. File 14 là lộ trình biến specs thành code.

### Nếu bạn sắp viết code (đọc để làm)

1. Đọc [core/01](core/01-tong-quan-kien-truc.md) và [core/14](core/14-lo-trinh-xay-dung-va-kiem-thu.md).
2. Điền [technology/quyet-dinh-hien-tai.md](technology/quyet-dinh-hien-tai.md) (xem [ví dụ đã điền](examples/vi-du-macos-deepseek-chat.md)).
3. Làm theo từng giai đoạn trong file 14. Mỗi giai đoạn chỉ cần một vài file `core/` — file 14 chỉ rõ giai đoạn nào đọc file nào.

### Nếu bạn muốn đổi một công nghệ (chạy lại với lựa chọn khác)

Ví dụ: đang dùng web UI, muốn chuyển sang macOS native app; hoặc đổi từ model A sang model B:

1. Mở file `technology/` tương ứng, sửa phần quyết định.
2. Cập nhật [technology/quyet-dinh-hien-tai.md](technology/quyet-dinh-hien-tai.md).
3. Kiểm tra mục "Ảnh hưởng lên core" ở cuối file technology đó — nó cho biết giao diện nào bị chạm tới. Ngoài các giao diện đó, code core không phải sửa.

---

## Sáu ý tưởng cốt lõi (đọc 2 phút)

Toàn bộ kiến trúc rút gọn còn sáu ý tưởng. Mọi file `core/` đều là hệ quả của chúng:

1. **Mọi thứ là module cắm được (plugin).** Vòng lặp agent, bộ ghi log, nhà cung cấp model, từng công cụ — đều là module đăng ký vào một bộ máy chung. Không có "nhân đặc quyền" phải sửa khi muốn mở rộng; muốn thêm tính năng thì cắm thêm module.
2. **Nhật ký sự kiện là nguồn sự thật duy nhất.** Mọi thứ mô hình đã thấy đều được ghi thành sự kiện nối tiếp (append-only) trong một session log. Lịch sử hội thoại *suy ra* từ log, không lưu riêng. Nhờ vậy: khôi phục sau crash, fork cuộc trò chuyện, replay để kiểm thử — đều là đọc lại log.
3. **Khả năng thay thế được gọi là "seam" và luôn có 3 vai:** định nghĩa giao diện, bên cài đặt (provider), bên sử dụng (consumer). Đổi provider (ví dụ đổi nơi lưu trữ, đổi cách chạy lệnh) là đổi toàn bộ hành vi mà không sửa consumer.
4. **Model-visible ⟺ logged.** Bất cứ thứ gì đưa vào yêu cầu gửi cho mô hình đều phải tái dựng được từ log. Muốn thêm dữ kiện cho mô hình thấy → thêm một loại sự kiện.
5. **Đăng ký nào cũng phải huỷ được.** Module nạp lên đăng ký công cụ/prompt/listener; khi gỡ module, mọi đăng ký phải tự thu hồi sạch. Đây là điều kiện để nạp/gỡ nóng và để test không rò rỉ trạng thái.
6. **Ngữ cảnh là ngân sách hữu hạn, không phải kho chứa.** Mỗi token đưa vào một lần gọi đều tiêu tốn "ngân sách chú ý" của mô hình: ngữ cảnh càng dài, khả năng nhớ đúng và suy luận xuyên suốt càng suy giảm dần (context rot). Mục tiêu của mỗi lần ghép là **tập token nhỏ nhất có tín hiệu cao nhất** vẫn đủ để mô hình làm đúng việc — tải dữ kiện đúng lúc (just-in-time) thay vì nạp hết từ đầu, cắt kết quả tool thừa, và khi việc dài vượt cửa sổ thì nén (compaction), ghi chú ra ngoài ngữ cảnh (structured note-taking) hoặc cách ly việc con vào agent con. Chi tiết: [core/07](core/07-prompt-va-ngu-canh.md).

Chi tiết: [core/01-tong-quan-kien-truc.md](core/01-tong-quan-kien-truc.md).

---

## Thuật ngữ dùng thống nhất

| Thuật ngữ trong specs | Nghĩa |
|---|---|
| **Agent** | Một thực thể sống đang điều khiển một cuộc hội thoại: có hộp thư đến, trạng thái idle/running, và quyền huỷ. |
| **Session** | Một cuộc hội thoại bền vững, được lưu thành nhật ký sự kiện. Agent là "người lái", Session là "cuốn sổ". |
| **Turn (lượt)** | Một vòng xử lý: từ khi nhận đầu vào cho tới khi mô hình và công cụ dừng hẳn. |
| **Step (bước)** | Một lần gọi mô hình + các công cụ mà câu trả lời đó yêu cầu. Một turn gồm 0..nhiều step. |
| **Tool (công cụ)** | Khả năng mô hình gọi được (đọc file, chạy lệnh…), khai báo bằng schema + hàm thực thi. |
| **Provider** | Bên cài đặt một giao diện (ví dụ: một nhà cung cấp model, một backend lưu trữ). |
| **Seam** | Một khả năng thay thế được, gồm đủ 3 vai: định nghĩa / provider / consumer. |
| **Profile** | Một bản phối các module để chạy một kiểu ứng dụng (giao diện chat, chạy một lần rồi thoát, nhúng vào app khác…). |
| **Preset** | Bản phối module riêng cho một session/agent, khác với phần còn lại trong cùng tiến trình. |
| **Ngân sách ngữ cảnh** | Phần cửa sổ ngữ cảnh của tuyến model hiện tại được phân cho từng loại token của một lần gọi (prompt, tool schema, lịch sử, kết quả tool…), kèm phần dự trữ cho đầu ra. Xem [core/07](core/07-prompt-va-ngu-canh.md). |
| **Truy hồi đúng lúc (JIT)** | Chỉ giữ định danh nhẹ (đường dẫn, truy vấn, liên kết, id) trong ngữ cảnh và nạp nội dung thật bằng tool khi cần, thay vì nạp toàn bộ từ đầu. Xem [core/07](core/07-prompt-va-ngu-canh.md). |
| **Compaction (nén)** | Tóm tắt một đoạn lịch sử dài thành bản ngắn thay thế nó trên bề mặt gửi model; bản gốc vẫn ở trong log. Xem [core/10](core/10-phoi-hop-va-mo-rong.md). |
| **Ghi chú có cấu trúc** | Ghi tiến độ/quyết định/việc còn mở ra bộ nhớ ngoài cửa sổ ngữ cảnh (file ghi chú, todo…) để đọc lại sau khi nén hoặc reset. Xem [core/10](core/10-phoi-hop-va-mo-rong.md). |

Bản đầy đủ kèm các khái niệm phụ (inbox, scope, projection…): xem từng file core liên quan — mỗi file tự định nghĩa thuật ngữ của nó ở đầu.

---

## Nguồn gốc và độ trung thực

Specs này được reverse-engineer từ DeepSeek Harness (kiến trúc plugin trên một framework context/event, session log dạng event-sourcing, các seam khả năng). File [core/15-ban-do-doi-chieu-deepseek-harness.md](core/15-ban-do-doi-chieu-deepseek-harness.md) đối chiếu từng spec với package/tài liệu gốc để bạn tra cứu cài đặt tham chiếu khi cần. Khi specs generic và cài đặt gốc khác nhau, **specs generic thắng** — mục tiêu là harness của bạn, không phải bản sao.

Phần kỹ thuật ngữ cảnh (ý tưởng 6 ở trên, [core/07](core/07-prompt-va-ngu-canh.md), phần thiết kế tool trong [core/06](core/06-tools-va-thuc-thi.md) và phần ngữ cảnh dài hạn trong [core/10](core/10-phoi-hop-va-mo-rong.md)) tham chiếu các bài viết của Anthropic: [Effective context engineering for AI agents](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents), [Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents) và [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system). Các bài viết đó là nguồn nguyên tắc; cách diễn đạt thành giao diện và bất biến trong bộ specs này vẫn theo quy tắc vàng ở trên (core trung lập công nghệ).

---

## Vị trí của harness trong hệ tự tiến hoá

Bộ specs này là **một tầng** của XuVi Harness — hệ agent tự tiến hoá có
[ĐẠO.md](../../ĐẠO.md) làm la bàn (bức tranh toàn hệ: [README.md](../../README.md)).
Quan hệ cần nhớ khi sửa bất cứ file nào ở đây:

- Harness là *runtime* mà các plugin (đầu tiên: `plugins/waldorf/`) chạy trên đó.
  Thay đổi ở harness specs phải tự hỏi: nó làm mọi plugin tốt lên, hay chỉ đang
  vá cho một plugin? Trường hợp sau thuộc về specs của plugin đó.
- Harness **tiến hoá qua loop** định nghĩa ở [cron.md](../../cron.md): sửa specs
  ở đây chỉ được giữ khi benchmark tổng thể
  ([benchmark/specs/01-benchmark-tong-the.md](../../benchmark/specs/01-benchmark-tong-the.md),
  đặc biệt chiều B4) đo trước/sau và kết quả được lưu trong `benchmark/results/`.
- Trong loop đầu tiên (hackathon), harness ở vai *agent đang chạy loop*, không
  phải sản phẩm code thứ hai phải xây: xem
  [plugins/waldorf/specs/10_loop-dau-tien-hackathon.md](../../plugins/waldorf/specs/10_loop-dau-tien-hackathon.md).
  Vì vậy `technology/quyet-dinh-hien-tai.md` còn trống là trạng thái đúng cho
  tới khi một điểm nghẽn benchmark thật sự nằm ở tầng harness — không điền nó
  chỉ để "cho đủ bộ".
