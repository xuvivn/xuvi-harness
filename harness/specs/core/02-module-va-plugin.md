# 02 — Module, plugin và vòng đời

> File này định nghĩa bộ máy cắm module: nhờ nó mà mọi khối trong harness đều thay thế, nạp nóng và gỡ sạch được. Đây là phần "hệ điều hành thu nhỏ" của harness. Công nghệ cụ thể để cài đặt nó (framework nào, ngôn ngữ nào) nằm ở [technology/runtime-va-ngon-ngu.md](../technology/runtime-va-ngon-ngu.md).

---

## 1. Năm khái niệm

### 1.1. Plugin (module)

Một đơn vị chức năng đóng gói. Plugin có thể:

- cung cấp một **dịch vụ** (service) cho plugin khác dùng,
- đăng ký đóng góp (tool, mảnh prompt, provider, listener…),
- lắng nghe sự kiện.

Plugin **không** tự khởi tạo nhau theo thứ tự cứng. Nó chỉ tuyên bố: "tôi cần dịch vụ X, Y" — bộ máy chờ đủ dịch vụ rồi mới kích hoạt nó. Thứ tự nạp vì thế được suy ra từ phụ thuộc, không phải danh sách sắp tay.

### 1.2. Context

Một "kho dịch vụ" dùng chung. Dịch vụ đăng ký dưới một khoá ổn định, ví dụ:

```
sessions    → kho session đang sống
agents      → registry các agent
agentLoop   → nhà máy tạo agent (cài đặt vòng lặp cụ thể)
llm         → cổng gọi mô hình
tools       → registry công cụ
systemPrompt→ bộ ghép prompt
fs, shell, subprocess, sandbox, web, jobs, approval, ...
```

Plugin khác lấy dịch vụ qua khoá, **không import lớp cài đặt**. Nhờ vậy cài đặt có thể bị thay mà nơi dùng không biết.

### 1.3. Inject (tuyên bố phụ thuộc)

Plugin liệt kê các dịch vụ bắt buộc. Bộ máy chỉ kích hoạt plugin khi mọi dịch vụ đó đã có mặt. Dịch vụ tuỳ chọn thì lấy qua phép "lấy nếu có" (`get` không ném lỗi) tại thời điểm dùng.

### 1.4. Sự kiện có kiểu và chế độ dispatch

Các dịch vụ tuyên bố tên sự kiện của mình vào một bản đồ sự kiện dùng chung (plugin mới được phép *thêm* tên sự kiện của mình vào bản đồ qua cơ chế mở rộng khai báo — không sửa file của chủ sở hữu).

Mỗi sự kiện có **một** chế độ dispatch, là một phần của hợp đồng công khai:

| Chế độ | Chờ listener? | Thứ tự | Giá trị trả về | Dùng cho |
|---|---|---|---|---|
| `emit` | Không | Theo thứ tự đăng ký | Không | Thông báo quan sát (đã xảy ra…) |
| `waterfall` | Không* | Theo thứ tự đăng ký | Có — chuỗi "around middleware" | Chặn và biến đổi một quyết định/đầu vào |
| `parallel` | Có | Song song | Không | Việc độc lập cần chờ xong |
| `serial` | Có | Theo thứ tự đăng ký | Có | Chuỗi quyết định phải tuần tự (ví dụ khởi tạo agent) |
| `bail` | Không | Tới listener đầu tiên chốt | Có | Một quyết định duy nhất, ai nhận trước thắng |

\* Với `waterfall`: listener nhận `(…tham số, next)`. **Muốn chuyển tiếp bắt buộc gọi `next()`**; trả về mà không gọi `next()` là cố ý chặn chuỗi (short-circuit) — chỉ dùng khi listener sở hữu quyết định đó. Listener "chỉ quan sát/ghi chú" mà quên `next()` sẽ âm thầm chặn cả hệ thống: đây là lỗi kinh điển cần tránh và cần test.

### 1.5. Effect và disposer

Mọi đăng ký đều đi qua cơ chế effect và nhận lại **disposer** (hàm huỷ). Gỡ plugin → bộ máy gọi các disposer theo thứ tự ngược → đăng ký biến mất sạch: tool khỏi registry, listener khỏi sự kiện, provider khỏi seam.

Hệ quả bắt buộc:

- Không giữ trạng thái đăng ký trong biến toàn cục/singleton ngoài vòng đời plugin.
- Một đăng ký không chứng minh được khả năng huỷ sạch là một đăng ký chưa xong.

---

## 2. Vòng đời

```
(nạp) ──► chờ inject đủ ──► kích hoạt (apply) ──► đang chạy ──► gỡ ──► thu hồi effects ──► (hết)
                              │                                        ▲
                              └──── lỗi kích hoạt: fail loud ──────────┘
                                   (không để lại đăng ký nửa vời)
```

Quy tắc:

1. **Kích hoạt thất bại không được để lại rác.** Nếu `apply` ném lỗi giữa chừng, các effect đã đăng ký trong lần kích hoạt đó phải tự thu hồi (giao dịch nạp: hoặc vào hết, hoặc lùi hết).
2. **Gỡ phải chờ quiesce.** Khi gỡ một plugin sở hữu agent/dịch vụ đang bận: dừng nhận việc mới, chờ việc đang bay kết thúc hoặc bị huỷ, rồi mới thu hồi.
3. **Huỷ theo thứ tự phụ thuộc ngược.** Cái được tạo sau huỷ trước — đặc biệt: agent phải kết thúc xong mới được thu hồi các đăng ký theo phạm vi của nó (mục 4).

---

## 3. Đăng ký đóng góp: mẫu chung

Mọi registry trong harness (tool, prompt section, provider, command…) đều theo cùng một mẫu:

```
register(đóng_góp) -> disposer
```

- Đóng góp trùng tên trong cùng một phạm vi → lỗi ngay (fail loud), không âm thầm ghi đè.
- Registry phát sự kiện "đã thay đổi" (không kèm diff cũng được) để consumer tự đọc lại.
- Đọc registry luôn cho kết quả là bản chụp (snapshot) tách rời — người đọc sửa bản chụp không ảnh hưởng registry.

---

## 4. Scope: đăng ký riêng cho từng agent

Một đóng góp có thể ở một trong hai mức:

- **Global**: mọi agent đều thấy.
- **Scoped**: chỉ một agent (chính xác hơn: một *scope key*) thấy, và sống chết theo agent đó.

Ứng dụng: persona riêng cho một phiên, tool riêng cho một subagent, hạn chế tool cho một agent con.

Quy tắc:

1. **Một sự thật, hai hệ quả**: đăng ký qua context của agent thì vừa *chỉ agent đó thấy*, vừa *tự huỷ khi agent bị huỷ*. Không có hai cờ riêng.
2. **Shadowing (che)**: đóng góp scoped trùng tên sẽ che bản global *chỉ trong scope đó*. Đây là cơ chế persona/tool-variant theo agent.
3. **Restriction (hạn chế)**: một scope có thể *lọc bớt* tool global. Tool bị lọc thì biến mất khỏi cả prompt lẫn khả năng thực thi — như thể không tồn tại. Không được chỉ giấu khỏi prompt mà vẫn chạy được.
4. **Scope phẳng, không thừa kế cây.** Agent con không tự động thấy đăng ký scoped của cha. Quan hệ cha/con được mang theo dạng *dữ liệu* (lineage: session cha, độ sâu uỷ quyền), không phải cấu trúc scope.
5. **Cửa sổ setup**: khi tạo agent có một khoảnh khắc — sau khi scope và đối tượng agent đã tồn tại, nhưng *trước khi* agent/session được công bố và trước lần ghép prompt đầu tiên — để người tạo đăng ký thế giới riêng cho agent. Setup chỉ đăng ký, không được điều khiển agent chạy.
6. **Dispatch theo scope**: sự kiện về hoạt động của một agent chỉ tới listener của agent đó và listener global; sự kiện về *chính registry* (ví dụ "có tool mới") thì không lọc theo scope.

---

## 5. Danh tính và ID

- ID truyền qua ranh giới giữa các module phải là **ID có nhãn (branded)**: về mặt lưu trữ là chuỗi/số bình thường, nhưng về mặt kiểu thì `SessionId` không thể đưa nhầm vào chỗ cần `ToolCallId`. Nếu ngôn ngữ không có branded type, dùng lớp bọc (wrapper/newtype) hoặc ít nhất là tiền tố + hàm tạo validate duy nhất.
- ID do đúng chủ sở hữu của nó phát hành. Không module nào tự chế ID thuộc về module khác.

---

## 6. Những điều cấm

- Cấm import cài đặt cụ thể xuyên tầng: consumer của một seam chỉ được phụ thuộc **định nghĩa** của seam, không phụ thuộc provider.
- Cấm trạng thái dùng chung không có chủ: cache/singleton cấp module che giấu chủ sở hữu. Trạng thái dùng chung phải treo trên một dịch vụ hoặc một thực thể có vòng đời rõ ràng.
- Cấm đăng ký không có disposer, cấm "đăng ký một lần cho mãi mãi" ngoài vòng đời plugin.
- Cấm dùng sự kiện làm kênh gọi hàm trực tiếp khi một phương thức dịch vụ là đủ (sự kiện dành cho chặn/chính sách/ quan sát; gọi khả năng trực tiếp thì dùng service method).

---

## 7. Tiêu chí nghiệm thu

- [ ] Nạp một plugin tool thử nghiệm qua cấu hình (không sửa code core), thấy tool xuất hiện trong prompt; gỡ plugin, tool biến mất khỏi cả prompt và thực thi.
- [ ] Huỷ một agent: mọi đăng ký scoped của nó biến mất; agent khác không bị ảnh hưởng.
- [ ] Một listener `waterfall` quên gọi `next()` trong test phải bị phát hiện (test chứng minh chuỗi bị chặn là hành vi có chủ đích, không phải mặc định vô tình).
- [ ] Kích hoạt một plugin lỗi giữa chừng: hệ thống không còn đăng ký nào của nó sót lại.
