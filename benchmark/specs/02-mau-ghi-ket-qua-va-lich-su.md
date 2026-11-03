# 02 — Mẫu ghi kết quả benchmark và lịch sử xu hướng

> Cách dùng: sao toàn bộ khối mẫu dưới đây thành một file mới trong
> `benchmark/results/`, đặt tên `YYYY-MM-DD_<so-loop>_<nhan-ngan>.md`
> (ví dụ `2026-11-04_loop-1_demo-hackathon.md`). Điền bằng bằng chứng thật.
> Sau khi ghi xong, thêm đúng một dòng vào `benchmark/results/INDEX.md`.
>
> Không viết lại file kết quả cũ. Sai thì tạo file đính chính.

---

```markdown
# Kết quả benchmark — <ngày> — <loop/đợt> — <nhãn>

- **Thời điểm đo:** <ngày giờ, múi giờ>
- **Ai đo:** <người / agent + model>
- **Đối tượng đo:** <trạng thái specs + sản phẩm sinh ra tại thời điểm đo;
  ghi cách tái dựng: lệnh chạy, thư mục, cổng — không commit artefakt>
- **Baseline so sánh:** <tên file kết quả trước đó, hoặc "chưa có">
- **Giả thuyết của loop này:** <câu nếu–thì ở pha 3 của cron.md>
- **Phán quyết:** GIỮ / HUỶ / ĐỖ — <một đoạn lý do dựa trên cổng và số>

## Cổng veto

| Cổng | Trạng thái | Bằng chứng |
|---|---|---|
| G1 Đạo & an toàn sự sống | ĐẠT / CHƯA ĐẠT / CHƯA ĐO | |
| G2 Trung thực bằng chứng | ĐẠT / CHƯA ĐẠT / CHƯA ĐO | |
| G3 Riêng tư & bí mật | ĐẠT / CHƯA ĐẠT / CHƯA ĐO | |

## Điểm từng chiều

| Chiều | Điểm (0–4) | Mức bằng chứng cao nhất | Bằng chứng (lệnh kiểm, file, quan sát) | Delta so với baseline |
|---|---|---|---|---|
| B1 Giá trị & trọn vẹn sản phẩm (20%) | | | | |
| B2 Giáo dục & văn hoá (15%) | | | | |
| B3 Trải nghiệm người dùng (10%) | | | | |
| B4 Kiến trúc harness (15%) | | | | |
| B5 Tin cậy kỹ thuật (15%) | | | | |
| B6 AI thật & chọn model (10%) | | | | |
| B7 Nghiên cứu & học hỏi (5%) | | | | |
| B8 Sức khoẻ tiến hoá (10%) | | | | |
| **Tổng có trọng số (/100)** | | — | — | |

## Research đã quét trong loop này

| Nguồn (đã đăng ký trong research/specs) | Ngày kiểm tra | Điều mới | Quyết định |
|---|---|---|---|
| | | | adopt / watch / reject — lý do một dòng |

## Điều đã sửa ở specs trong loop này

- <đường dẫn specs> — <ý định thay đổi một dòng>

## Bài học và điểm nghẽn kế tiếp

- <điều hệ học được, kể cả từ phán quyết HUỶ>
- Điểm nghẽn quan trọng nhất cho loop sau: <một điểm duy nhất>

## Vệ sinh commit

- `git status` tại thời điểm kết thúc: <chỉ gồm diện được commit? có gì phải dọn?>
- ĐẠO.md có diff không: không (bắt buộc)
```

---

## Dòng xu hướng cho INDEX.md

Mỗi lần đo thêm đúng một dòng vào bảng trong `benchmark/results/INDEX.md`:

```markdown
| <ngày> | <loop/đợt> | <tổng /100> | <G1/G2/G3: Đ=đạt, X=chưa đạt, ?=chưa đo> | <phán quyết> | [<nhãn>](<tên file>) |
```

`INDEX.md` luôn là nơi đọc đầu tiên để biết hệ đang tiến hoá thật hay đang
đi ngang — không phải nơi trình bày thành tích. Một dòng tổng điểm đứng yên
kèm bài học rõ ràng vẫn là một lần đo có giá trị.
