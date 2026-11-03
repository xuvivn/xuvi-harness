# 03 — Nghiên cứu & kiến trúc XuVi Waldorf

Ngày kiểm chứng: 03/11/2026. Nguồn: https://dev.meta.ai/docs/models và các trang capability liên quan (đã fetch trực tiếp), brief đã xác minh cùng ngày, tài liệu local trong `references/`.

## 1. Lựa chọn sản phẩm Meta (có lý do)

| Sản phẩm | Quyết định | Lý do |
|---|---|---|
| muse-wonder-1.0 | Dùng (adapter server, tùy chọn) | Suy luận văn bản phức tạp: tóm tắt bài dạy, góp ý kể chuyện có căn cứ. Qua API key env, timeout, không log secret. |
| muse-spark-1.3 | Dùng (adapter server, tùy chọn) | Tác vụ cấu trúc nhanh: chia cảnh, trích nhân vật/chi tiết. Cùng endpoint chat, model cấu hình được. |
| muse-image-1.0 | Dùng (adapter server, tùy chọn) | Sinh tranh cảnh phong cách màu nước/sáp/bút chì cho giáo viên chuẩn bị. Không có key → nút báo rõ chưa cấu hình, vẫn tải ảnh thủ công được. |
| Muse Voice Transcribe (POST /v1/asr/transcribe) | Dùng (adapter server, tùy chọn) | Chuyển âm thanh → văn bản, tiếng Việt được hỗ trợ (theo brief đã xác minh 03/11/2026). KHÔNG dùng endpoint OpenAI `audio/transcriptions`. Không có key → nhập/sửa transcript thủ công. |
| SAM 3.1 (Segment Anything) | Không dùng | Bài toán là kể chuyện/chương trình học; không cần phân đoạn ảnh. Thêm vào chỉ tăng độ phức tạp. Có trang cookbook SAM trên dev.meta.ai nhưng ngoài phạm vi. |
| Muse Spark Open (open-weight) | Không dùng ở bản local này | Cần tự host GPU; ứng dụng chạy local cho giáo viên, ưu tiên API hosted tùy chọn + chế độ thủ công không mạng. Có thể thay adapter sau. |
| Muse Glimmer (open-weight nhỏ) | Không dùng | Mô hình nhỏ tự host; không phù hợp pipeline hiện tại, độ chính xác tiếng Việt chưa kiểm chứng trong repo này. |
| Baseten / Modal / Fireworks | Không dùng | Nền tảng hosting/inference bên thứ ba; yêu cầu local-only, loopback, không deploy. Khi cần triển khai thật sẽ đánh giá lại. |
| Muse CLI bridge | Dùng tùy chọn (server-side, readonly) | `POST /api/ai/cli-brief`: gọi `muse exec --preset empty --disable-shell --disable-write --disable-web-tools --disable-reminders --disable-request-user-input --no-foreign-personal-context --no-session-log --prompt-file <file tạm>` trong thư mục tạm rỗng phía server nếu bật `XUVI_CLI_BRIDGE=1`; lỗi rõ ràng nếu CLI vắng mặt. Không bao giờ đọc credential/Keychain từ code ứng dụng. |

Nguyên tắc: mọi tính năng thủ công hoạt động không cần mạng. Mọi lần gửi dữ liệu ra ngoài đều cần cờ xác nhận `confirmRemote:true` từ UI kèm hộp thoại tiết lộ (disclosure) trước khi gửi. Thuật toán dự phòng cục bộ luôn gắn nhãn “phân tích cục bộ — không phải AI”.

## 2. Kiểm chứng endpoint (không bịa)

- Trang models liệt kê: `muse-wonder-1.0`, `muse-spark-1.3`, `muse-image-1.0`, `muse-voice-transcribe-1.0`, `muse-spark-open`, `muse-glimmer`, SAM cookbook — khớp brief.
- Transcribe: `POST /v1/asr/transcribe`, multipart gồm field `request` (JSON `{mode:"DIARIZATION",model:"muse-voice-transcribe-1.0",audioEncoding:"WAV",languageBias:["Vietnamese"]}`) và field `audio` (WAV mono PCM16 16/24kHz, tối đa 10 phút) — đã kiểm chứng live 03/11/2026. Trình duyệt chuyển WebM→WAV bằng AudioContext trước khi tải lên. Đáp ứng: `transcript`, `audioDurationMs`, `turns[]`. Base URL qua env `META_API_BASE`.
- Text: chat-completions tương thích qua `POST /v1/chat/completions` với model cấu hình được (`META_TEXT_MODEL`, mặc định `muse-spark-1.3`; brief soạn thảo dùng `muse-wonder-1.0`).
- Image: `POST /v1/images/generations` model `muse-image-1.0` qua env. Nếu provider trả lỗi, UI hiển thị lỗi nguyên văn đã redact secret.

## 3. Kiến trúc

```
XuVi (local, loopback 127.0.0.1)
├─ client/  Vite + React 18 + TypeScript — SPA tiếng Việt, hash routing (không cần server rewrite)
├─ server/  Node 26 + Express + TypeScript (tsx dev / tsc build)
│   ├─ node:sqlite (DatabaseSync) → data/xuvi.db (WAL)
│   ├─ data/files/  ảnh/audio/tài liệu tải lên (tên UUID, chống path traversal)
│   ├─ adapters/ai.ts  text/image/transcribe/cli — env key, timeout 60s, redact
│   ├─ lib/documents.ts  sinh 6 loại hồ sơ (DOCX qua `docx`, Markdown, JSON, HTML in/PDF qua print CSS)
│   └─ lib/extract.ts  TXT/MD/HTML/DOCX(mammoth)/PDF(pdf-parse) + review trước khi lưu
└─ tests/  node:test — persistence, timetable math/overlap, mapping/unmapped, documents, story split, upload reject
```

- `npm run dev`: một lệnh — concurrently chạy API (5174) + Vite (5173, proxy /api → 5174). Production: `npm run build` → server phục vụ `client/dist` tĩnh + API cùng cổng 5174.
- Bảo mật local: bind 127.0.0.1, giới hạn upload (ảnh 10MB, audio 25MB, tài liệu 15MB), whitelist MIME/extension, sanitize HTML (strip script/on*), không secret ở frontend/log.

## 4. Schema dữ liệu (SQLite)

`settings(key,value)` · `programs(id,level,title,philosophy,goals,qualities,areas,methods,hours,assessment,updatedAt)` · `grade_programs(id,gradeLevel,ageRange,title,overview,subjectsJson,milestonesJson,sourceRef)` · `sources(id,kind,title,path,pages,sha,note,createdAt)` · `requirements(id,sourceId,code,gradeLevel,subject,text,page,createdAt)` · `mappings(id,requirementId,content,hours,evidence,degree,note,updatedAt)` degree∈{chua-doi-chieu, mot-phan, tuong-dong, chat-che} · `year_plans(id,year,startDate,endDate,weeks,subjectsJson,themesJson,activitiesJson,assessmentJson,contingency)` · `timetable_entries(id,day,start,end,title,teacher,objective,kind,actualMinutes)` kind∈{hoc,sinh-hoat} — server kiểm tra chồng lấn cùng ngày · `students(id,name,birthYear,className,note,illustrative)` · `attendance(id,studentId,date,status,note)` · `journal_entries(id,date,content,assessment,assignment,note)` · `artifacts(id,studentId,title,kind,date,note,filePath)` · `reports(id,studentId,period,summary,strengths,nextSteps,createdAt)` · `stories(id,title,sourceText,sourceInfo,rights,ageRange,language,context,goal,version,createdAt,updatedAt)` · `scenes(id,storyId,ord,title,summary,keywords,imagePath,imageStyle,prompt)` · `characters(id,storyId,name,details)` · `attempts(id,storyId,mode,assistance,audioPath,transcript,storyVersion,feedbackJson,createdAt)` mode∈{prompt,fading,free}, assistance∈{none,text,keywords} ghi riêng · `documents(id,type,title,format,createdAt)` snapshot sinh từ dữ liệu hiện hành khi xuất.

## 5. Kho nguồn & provenance (dữ liệu thật trong repo)

| Nguồn | Loại | Quy mô | Dùng làm |
|---|---|---|---|
| `references/01-GDPT2018-TT32…pdf` | MOET GDPT 2018 (TT32) | 1555 trang, ~14MB | Duyệt trang/tìm kiếm/trích yêu cầu vào ma trận. Chỉ những yêu cầu đã nhập mới được đối chiếu — bộ đếm “chưa đối chiếu” tính trung thực, không tuyên bố đạt chuẩn toàn phần. |
| `references/01-Waldorf-co-dien-lop-1-12.md` | Waldorf cổ điển L1–12 | 12 lớp | Chương trình theo lớp, block/môn. |
| `references/02-Waldorf-Cambridge-lop-1-12.md` | Waldorf–Cambridge L1–12 | 12 lớp | Tham chiếu đối sánh, nhãn examStatus là nội bộ. |
| `references/03-Nha-Thuan-Nhien-mam-non-lop-12.md` | Nhà Thuận Nhiên MN–L12 | 18 tài liệu | Mầm non + phổ thông, giữ sourceRows/provenance. |
| `references/Việt Huyền Sử Ký.md` | Truyện Việt | 28 truyện (## headings) | Thư viện Story mẫu, ghi nguồn rõ ràng. |
| `references/xuvistory.md` | Đặc tả XuViStory | — | Tiêu chí góp ý: không điểm tổng, phân biệt thiếu sót vs chuyển thể có chủ ý. |

Seed học sinh: chỉ tên minh họa hư cấu, cột `illustrative=1`, UI gắn nhãn “dữ liệu minh họa”.

## 6. Rủi ro & giới hạn trung thực

PDF 1555 trang: trích xuất text theo trang khi duyệt (pdf-parse), tìm kiếm trên text đã trích; trang scan không có text sẽ báo “không trích được text”. AI không có key vẫn dùng được toàn bộ luồng thủ công. Góp ý “delivery” từ audio chỉ xuất hiện khi có phân tích audio thật; mặc định ghi rõ chưa phân tích.
