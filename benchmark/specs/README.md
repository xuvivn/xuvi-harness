# Benchmark — đo toàn thể sản phẩm, và lưu lại để tiến hoá có trí nhớ

Benchmark ở đây **không** phải một bộ test kỹ thuật. Nó là cách hệ trả lời, bằng
bằng chứng: *"Sản phẩm — gồm harness, plugin Waldorf, trải nghiệm của người dùng
thật và tính trung thực của mọi tuyên bố — đang tốt đến đâu, và loop vừa rồi có
làm nó tốt lên thật không?"*

Không có benchmark này, "tự tiến hoá" chỉ là "tự thay đổi". Vì vậy kết quả của
mọi lần đo đều **được lưu lại** và không bị viết lại.

## Các file ở đây

| File | Vai trò |
|---|---|
| [01-benchmark-tong-the.md](01-benchmark-tong-the.md) | Bộ tiêu chí: 3 cổng veto + 8 chiều chấm điểm, cách chấm, mức bằng chứng, cách tính tổng |
| [02-mau-ghi-ket-qua-va-lich-su.md](02-mau-ghi-ket-qua-va-lich-su.md) | Mẫu để tạo một file kết quả mới và cập nhật chỉ số xu hướng |
| `../results/INDEX.md` | Bảng xu hướng: mỗi lần đo một dòng, đọc 30 giây là thấy hệ đang đi lên hay đi ngang |
| `../results/<ngày>_<loop>_<nhãn>.md` | Từng lần đo, bất biến sau khi ghi (sửa lỗi chính tả phải ghi chú sửa) |

## Chạy một lần benchmark trong 6 bước

1. Sao mẫu trong [02-mau-ghi-ket-qua-va-lich-su.md](02-mau-ghi-ket-qua-va-lich-su.md)
   thành một file mới trong `benchmark/results/`, đặt tên theo ngày và loop.
2. Ghi baseline là kết quả gần nhất trước đó (mặc định là dòng cuối của `INDEX.md`).
3. Kiểm 3 cổng trước. Một cổng `CHƯA ĐẠT` làm toàn bộ lần đo không được giữ
   thay đổi — ghi rõ, dừng tính tổng như một thành tích.
4. Chấm 8 chiều theo thang 0–4, mỗi điểm phải kèm bằng chứng và mức bằng chứng.
   Chiều nào chưa đo được thì ghi `chưa kiểm chứng` và điểm bị chặn theo quy tắc
   trong file 01 — không bỏ trống, không đoán.
5. Tính điểm tổng có trọng số, nhưng trình bày **hồ sơ từng chiều trước, tổng sau**.
6. Ghi phán quyết của loop (giữ / huỷ / đỗ — theo cổng benchmark trong
   [cron.md](../../cron.md)) và thêm đúng một dòng vào `INDEX.md`.

## Hai quy tắc lưu trữ

- **Chỉ thêm, không sửa lịch sử.** Kết quả cũ là trí nhớ của hệ tại thời điểm đó.
  Phát hiện kết quả cũ sai: tạo file mới ghi đính chính, không viết lại file cũ.
- **Kết quả được commit, artefakt đo thì không.** File kết quả (`.md`) thuộc
  diện commit theo [.gitignore](../../.gitignore). Database, media, bản build
  dùng trong lúc đo vẫn là phần sinh ra — không commit.
