# 04 — Báo cáo kiểm chứng (cập nhật 03/11/2026, đợt hoàn thiện)

Phân biệt rõ: **[live]** gọi Meta thật bằng `.env` do người dùng cấp (khóa chỉ trong bộ nhớ tiến trình, không in/log/lưu backup), **[mock]** provider giả lập cục bộ trong test, **[local]** logic cục bộ, **[chưa]** chưa kiểm chứng bằng trình duyệt thật.

## Đã kiểm chứng bằng chạy thật

| Kiểm tra | Loại | Kết quả |
|---|---|---|
| `npm run typecheck` (server + client) | local | Đạt 0 lỗi |
| `npm test` (tsx --test tests/*.test.ts) | local+mock | Đạt 11/11: unit (timetable/range, split bao phủ toàn văn bản + chương, feedback cục bộ không điểm tổng, validateAiFeedback không điểm tổng/không suy diễn audio/giữ chuyển thể có chủ ý, sanitize, validateScenes, voice form contract) + integration isolated temp + 2 test tài sản nguồn + workflow QA xuyên suốt |
| `npm run build` | local | Đạt (0 cảnh báo); `client/dist` + `server/dist` |
| GET /v1/models qua app `/api/ai/models` | **live** | 200; chỉ in ID: muse-wonder-1.0, sam-3.1, muse-spark-1.3-contributor, muse-voice-transcribe-1.0, muse-spark-1.3, muse-image-1.0, muse-spark-1.2-contributor, muse-spark-1.2, muse-spark-1.1 |
| POST /api/stories/:id/ai-scenes (truyện Việt tổng hợp ngắn & truyện thư viện số 1) | **live** | 200, cảnh có cấu trúc chương/chi tiết cốt yếu/từ khóa đã kiểm chứng và lưu vào DB |
| POST /api/attempts/:id/ai-feedback (góp ý kể chuyện từ snapshot đóng băng) | **live** | 200; trả về `covered`, `strengths` có trích dẫn từ bản kể, `omissions` có trích dẫn từ bản đóng băng, `adaptations` ghi nhận chuyển thể có chủ ý không trừ lỗi, `ageFit`, `nextStep`, `overallScore: null`, `deliveryNote` ghi rõ chưa phân tích biểu cảm âm thanh |
| POST /api/ai/image → file bền vững | **live** | 200, sniff đúng WebP (655.136 byte qua route app), phục vụ lại 200 tại /files/*.webp; ảnh màu nước mẫu lưu tại `client/public/images/garden-watercolor.webp` |
| POST /api/ai/transcribe (WAV tổng hợp 1 giây, không phải giọng người thật) | **live** | 200 theo contract multipart `request`+`audio`; trả transcript/audioDurationMs/turns. Nội dung "Yeah" là artefact của WAV im lặng tổng hợp — không phải kiểm chứng độ chính xác tiếng Việt (orchestrator đã kiểm chứng WAV giọng tổng hợp tiếng Việt riêng, xem 08) |
| Integration test isolated (XUVI_DATA_DIR tạm, cổng riêng, tự tắt server) | mock+local | 2100 phút/40 tiết; 400 ngoài khung 08:00–15:00; 409 trùng giờ; ID sai 400/404; provenance tr.320; 41 grade; bulk-from-pages 201; đủ trường 6 loại hồ sơ bằng fixture đã populate (gồm điểm danh/sản phẩm/báo cáo kỳ trong loại 6); cả 6 DOCX có header PK; snapshot đóng băng sau khi sửa truyện; fake-provider text/image(transcribe) qua contract thật; upload .exe 400; HTML bị sanitize; backup v2 gồm media, restore hỏng 400 giữ nguyên dữ liệu, media sống sót sau restore; bundle truyện; restart persistence |
| Migration trên bản sao DB đã seed cũ (2 lần khởi động) | local | Lần 1: grades 16 placeholder → 41 bản ghi nguồn, 4 yêu cầu bịa bị xóa và thay bằng trích dẫn đã kiểm chứng tr.320/68/386/189, timetable → 2100. Lần 2 idempotent, không nhân bản |
| Fake provider (mock HTTP trong integration test) | **mock** | chat → text/cảnh JSON; images b64 → lưu file; asr request+audio → transcript+turns. Đây là kiểm chứng contract phía app, không thay thế live |

## Chưa kiểm chứng / giới hạn trung thực

- **[chưa]** Chưa chạy kiểm tra trình duyệt tự động (screenshot/responsive/MediaRecorder thật trên thiết bị). UI đã build và typecheck; orchestrator/người dùng cần mở http://127.0.0.1:5174 để kiểm tra trực quan. Ghi âm: code chuyển WebM→WAV 16kHz bằng AudioContext; từ chối micro có hướng dẫn tải WAV.
- Độ chính xác phiên âm tiếng Việt bằng giọng thật: không dùng dữ liệu cá nhân; chỉ orchestrator đã kiểm chứng bằng giọng tổng hợp (08_meta_live_check.md).
- DOC cũ (.doc): dùng macOS textutil khi có; máy không có textutil sẽ báo lỗi rõ ràng thay vì giả trích xuất.
- Không tuyên bố đạt chuẩn/chứng nhận GDPT hay Waldorf chính thức ở bất kỳ đâu trong app/hồ sơ.
