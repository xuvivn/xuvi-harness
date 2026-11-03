# Technology — Runtime và ngôn ngữ

> Chọn nền để cài đặt logic trong [`core/02-module-va-plugin.md`](../core/02-module-va-plugin.md) và toàn bộ harness. Core chỉ yêu cầu nền của bạn cung cấp được: kho dịch vụ có inject, effect/disposer, dispatch sự kiện 5 chế độ, và huỷ hợp tác (signal). Nền nào không cung cấp sẵn thì bạn tự viết các nguyên thuỷ đó — chúng nhỏ và là giai đoạn 0 của lộ trình ([core/14](../core/14-lo-trinh-xay-dung-va-kiem-thu.md)).

---

## 1. Các lựa chọn

### 1.1. Ngôn ngữ + runtime

| Lựa chọn | Mạnh | Yếu / đánh đổi |
|---|---|---|
| **TypeScript / Node** *(tham chiếu: DeepSeek Harness)* | Hệ sinh thái AI tooling lớn nhất; cùng ngôn ngữ cho host, web client và tool runtime; kiểu mạnh hỗ trợ branded ID và discriminated union đúng như core mô tả | Tiến trình đơn luồng: việc nặng CPU phải đẩy ra worker/tiến trình con |
| Python | Thư viện AI/dữ liệu phong phú; hợp khi harness là một phần của pipeline dữ liệu | Đóng gói app desktop khó hơn; hệ kiểu yếu hơn cho các bất biến của core (phải bù bằng validate runtime) |
| Rust / Swift / Kotlin… (native) | Hiệu năng, một binary, tích hợp OS sâu | Tốc độ phát triển tính năng harness chậm hơn; hệ sinh thái MCP/tooling phải tự bắc cầu |
| Hỗn hợp: lõi ở ngôn ngữ A, mặt tiếp xúc ở ngôn ngữ B | Mỗi phía dùng điểm mạnh của mình | Phải có một giao thức wire giữa hai phía (đó là profile `sdk`, [core/12](../core/12-composition-va-cau-hinh.md)) — thêm một bề mặt phải giữ tương thích |

### 1.2. Cơ chế module/plugin

| Lựa chọn | Ghi chú |
|---|---|
| Dùng một framework context/plugin có sẵn *(tham chiếu: Cordis — kho dịch vụ + inject + effect + sự kiện có kiểu)* | Được ngay vòng đời nạp/gỡ và HMR ở mức framework; phải chấp nhận quy ước của framework |
| Tự viết module system tối thiểu | Toàn quyền; chỉ nên làm khi không framework nào khớp mô hình tiến trình của bạn. Phạm vi tối thiểu: đúng các nguyên thuỷ ở [core/02](../core/02-module-va-plugin.md) — đừng xây hơn thế ở giai đoạn 0 |

### 1.3. Mô hình tiến trình

| Lựa chọn | Khi nào hợp |
|---|---|
| Một tiến trình (headless/CLI) | Bản đầu tiên; chạy việc rồi thoát |
| Host (tiến trình lõi) + Client (UI) nối qua RPC/stream cục bộ *(tham chiếu)* | Có UI phong phú; host giữ session/agent, client chỉ hiển thị từ sự kiện. Client và host có thể khác công nghệ |
| Host + nhiều Worker | Có việc nền nặng hoặc cần cô lập thực thi code của model (PTC-style) — worker là chi tiết triển khai của một seam, không lọt ra core |

Bất biến dù chọn mô hình nào: **chỉ host sở hữu session log và agent**; client/worker không giữ nguồn sự thật thứ hai (core/01, 11).

---

## 2. Quyết định của lần này

- Ngôn ngữ + runtime:
- Framework module (hoặc "tự viết"):
- Mô hình tiến trình:
- Quản lý gói / monorepo (nếu có):
- Lý do một dòng:

## 3. Ảnh hưởng lên core

- Không đổi giao diện core nào. Nếu ngôn ngữ thiếu branded type/discriminated union: dùng wrapper type + hàm tạo validate cho ID (core/02, mục 5) và kiểm chứng log tại ranh giới đọc (core/13, mục 1).
- Mô hình tiến trình quyết định *nơi* các seam chạy (cùng tiến trình hay qua wire) — nếu qua wire, giao thức đó là một cài đặt của profile `sdk`, vẫn theo core/12.
