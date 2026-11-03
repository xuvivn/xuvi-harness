# XuVi — kế hoạch triển khai đầy đủ

Ngày: 03/11/2026. Nguồn yêu cầu: `00_master_prompt.md`, `references/xuvistory.md`, các tài liệu chương trình trong `references`, và các note sản phẩm/chương trình liên quan trong repo `minhxuvi` (chỉ đọc).

## Mục tiêu và tiêu chí hoàn thành

Một ứng dụng chạy local có lưu trữ bền vững, giao diện tiếng Việt và định vị **giáo dục lấy cảm hứng Waldorf tại Việt Nam**. Phạm vi bao gồm toàn bộ XuVi Story và sáu loại hồ sơ trong master prompt. Không đưa thông tin cá nhân học sinh thật từ kho ghi chép vào dữ liệu mẫu.

1. Tổng quan: công việc hôm nay, nhịp tuần, tình trạng chương trình, truyện đang luyện và khoảng trống đối chiếu.
2. Chương trình: triết lý, mục tiêu, phẩm chất/năng lực, lĩnh vực, phương pháp, giờ học, đánh giá; chương trình theo tuổi/lớp từ mầm non đến lớp 12 dựa trên tài liệu nguồn.
3. Ma trận GDPT: yêu cầu đầu ra có nguồn/trang, nội dung tương ứng, thời lượng, minh chứng, mức độ đối chiếu; tìm kiếm/lọc/sửa/nhập; không biến gợi ý thành chứng nhận đạt chuẩn.
4. Kế hoạch năm: mốc năm học, tuần học, phân phối môn, block/chủ đề, trải nghiệm, đánh giá và dự phòng; liên kết chương trình và lịch tuần.
5. Lịch tuần: thứ Hai–Sáu, 08:00–15:00; bài dạy, người phụ trách, mục tiêu, thời lượng thực tế; phân biệt học với sinh hoạt; chỉnh sửa và in.
6. Nhật ký và hồ sơ: điểm danh, nội dung, đánh giá, nhiệm vụ, sản phẩm học tập, ghi chú tiến bộ, báo cáo theo kỳ.
7. XuVi Story: nhập DOC/DOCX/PDF/HTML/MD/TXT và duyệt văn bản; nguồn/quyền sử dụng; độ tuổi/ngôn ngữ/bối cảnh/mục tiêu; bản đồ cảnh có sửa; tranh tạo/tải lên; ba chế độ luyện; ghi âm/tải âm thanh/nghe lại/sửa transcript; góp ý có căn cứ, không điểm tổng; lịch sử và nhịp bảy ngày; xuất/xóa dữ liệu.
8. Hồ sơ xuất: tạo đủ sáu loại từ dữ liệu hiện hành, xem trước, tải và in/PDF; các mẫu chứa đủ trường bắt buộc.
9. Cài đặt và dữ liệu: trạng thái AI, giải thích dữ liệu gửi đi, cấu hình phía server, sao lưu/khôi phục/xóa; dữ liệu riêng tư theo mặc định.

## Kiến trúc và chất lượng

Muse CLI thực hiện phát triển chính. Stack cụ thể, schema, lệnh chạy và quyết định provider sẽ được bổ sung trong quá trình triển khai. Ưu tiên React/TypeScript + backend local và SQLite, các adapter AI phía server; thao tác thủ công vẫn hoạt động khi mạng/provider không sẵn sàng. Không giả lập thành công AI hoặc gọi hình vector mẫu là tranh AI đã sinh. Không lưu khóa vào frontend/Git/log.

Thiết kế: nền giấy ấm, xanh rừng, đất nung, chữ có chân cho tiêu đề, bố cục thoáng, minh họa thực vật; sidebar rõ nhóm công việc; tương thích điện thoại, bàn phím và trạng thái tải/lỗi/rỗng.

## Các đợt thực hiện

1. Đọc nguồn, kiểm khả năng Meta hiện hành, lưu nghiên cứu và mô hình dữ liệu.
2. Dựng ứng dụng, nhập nguồn có provenance, tạo dữ liệu ví dụ minh bạch.
3. Hoàn thiện các mô-đun chương trình, năm, tuần, nhật ký, hồ sơ và xuất tài liệu.
4. Hoàn thiện Story từ thư viện đến luyện kể/góp ý và adapter AI thật.
5. Chạy typecheck/build, kiểm thử nghiệp vụ và API/lưu trữ/xuất; kiểm tra giao diện bằng trình duyệt; sửa lỗi.
6. Lưu báo cáo kiểm chứng, lệnh tái dựng, giới hạn thực tế; để bản local chạy cho người dùng kiểm tra.

## Nguồn kỹ thuật đã kiểm tra

Ngày 03/11/2026: https://dev.meta.ai/docs/models, https://dev.meta.ai/docs/overview, https://dev.meta.ai/docs/image-generation, https://dev.meta.ai/docs/speech-to-text.

Hướng chọn: Wonder cho phát triển và suy luận phức tạp, Spark cho cấu trúc văn bản/góp ý; Image cho tranh cảnh; Voice Transcribe cho tiếng Việt. SAM, Spark Open, Glimmer và các nền tảng hosting sẽ được đánh giá theo nhu cầu thực tế, không đưa vào chỉ để đủ tên. CLI đã đọc được đăng nhập hiện có khi chạy với quyền truy cập Keychain của máy.


## Đợt hoàn thiện 03/11/2026 (đã triển khai, không còn là kế hoạch)

- AI: contract voice `request`+`audio` WAV, image sniff WebP/PNG/JPEG lưu bền vững, text/scene JSON có kiểm chứng cấu trúc theo từng đoạn (không cắt cuối truyện), góp ý kể chuyện sâu bằng AI (`POST /api/attempts/:id/ai-feedback`) đối chiếu theo snapshot đã đóng băng (văn bản + bản đồ cảnh + độ tuổi + ghi chú chuyển thể + transcript đã rà soát, không cho điểm tổng, không suy diễn giọng kể từ chữ), tự động retry/fallback khi mô hình quá tải 503, CLI bridge bằng lệnh `muse exec` đã tài liệu hoá, nạp `.env` phía server, hỗ trợ `MODEL_API_KEY`.
- Nguồn: duyệt/tìm 1555 trang MOET từ `server/assets/moet-pages.json`; trích yêu cầu hàng loạt với trạng thái duyệt; 41 bản ghi curriculum đầy đủ từ `curriculum-source.json` với provenance file/dòng/SHA; xóa đúng 4 trích dẫn bịa cũ, thay bằng trích dẫn đã đối chiếu trang thật.
- Hồ sơ: đủ 6 loại với trường bắt buộc đã kiểm bằng fixture populate, bộ lọc theo lớp/tuổi (`gradeId`), ngày thẻ dạy (`day`) và học sinh (`studentId`), nhãn tiếng Việt rõ ràng (không xuất JSON thô hay mã nội bộ), bảng Markdown có separator/escape, DOCX khổ ngang có độ rộng cột tỷ lệ + header màu ấm + hàng không tách trang (`cantSplit`), bản in/PDF có `@page` chuẩn A4.
- Lịch tuần đủ 08:00–15:00 × 5 ngày = 2100 phút; liên kết tiết↔yêu cầu và trạng thái bao phủ bị ảnh hưởng khi đổi tiết; thẻ dạy gọn chọn theo từng ngày Thứ 2–Thứ 6.
- Story: snapshot đóng băng (văn bản+cảnh+nhân vật+điều kiện) cho mỗi lượt, mờ dần có hẹn giờ thật, ghi WAV 16kHz, phát lại ngay cạnh góp ý, góp ý cục bộ và góp ý AI có dẫn chứng, nhịp 7 ngày lưu bền vững, xuất gói truyện, xóa dọn tệp mồ côi (trừ tệp còn được snapshot tham chiếu).
- Sao lưu v2 gồm media, khôi phục stage + rollback cả DB lẫn tệp.

## Lệnh tái dựng (Rebuild)

```bash
npm install
ruby scripts/extract-curriculum.rb      # Tái tạo server/assets/curriculum-source.json (41 bản ghi)
python3 scripts/extract-moet.py         # Tái tạo server/assets/moet-pages.json (1555 trang)
npm run typecheck
npm test
npm run build
npm start                               # Mở http://127.0.0.1:5174
```
