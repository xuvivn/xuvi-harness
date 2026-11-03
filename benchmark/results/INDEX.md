# Chỉ số xu hướng benchmark

Mỗi lần đo một dòng, mới nhất ở dưới. Đọc bảng này trong 30 giây phải trả lời
được: hệ đang tốt lên thật, đi ngang, hay đang tự kể chuyện?
Quy tắc ghi: xem [../specs/02-mau-ghi-ket-qua-va-lich-su.md](../specs/02-mau-ghi-ket-qua-va-lich-su.md).
Không sửa số của dòng cũ — sai thì thêm dòng đính chính.

| Ngày | Loop/đợt | Tổng (/100) | Cổng G1/G2/G3 | Phán quyết | Kết quả |
|---|---|---|---|---|---|
| 2026-11-03 | baseline | 32.5 | ?/?/? | GHI NHẬN | [trước loop 1](2026-11-03_baseline-truoc-loop-1.md) |
| 2026-11-03 | bootstrap (3 loop: harness + app native + đo/dogfood) | 62.5 | ĐẠT/ĐẠT/ĐẠT | GIỮ — chuyển hẳn sang xuvi-harness | [bootstrap xuvi native swift](2026-11-03_bootstrap-xuvi-native-swift.md) |
| 2026-11-03 | chẩn đoán sau bootstrap; kiểm tra một phần, không đo tổng thể | — | ?/CHƯA ĐẠT kết luận chuyển nhà/? | ĐỖ kết luận chuyển nhà — chưa đủ bằng chứng spec 11 §5; không chấm lại 62.5 | [chẩn đoán và bước tiếp theo](2026-11-03_chan-doan-sau-bootstrap.md) |
| 2026-11-03 | evolution candidate: live red/green, 13/13 kỹ thuật, 79/79 checks độc lập | — | ?/CHƯA ĐẠT phán quyết/? | ĐỖ chuyển nhà — model KEEP khi cổng CHƯA ĐO; candidate chưa kích hoạt | [loop evolution candidate](2026-11-03_loop-evolution-candidate.md) |
| 2026-11-03 | cấu hình Meta trên UI; kiểm phạm vi yêu cầu trực tiếp | — | toàn hệ chưa đo lại | Bản dùng thử sẵn; ĐỖ nghiệm thu UI/Keychain thật; checks 44→54 đạt | [ô nhập khóa trên UI](2026-11-03_ui-meta-key.md) |
| 2026-11-03 | UI Vườn học (Spec 12) + XuVi Story Khối 11 Tiền tệ (sau nghỉ Thu) | — | ĐẠT/ĐẠT/ĐẠT (phạm vi B3/B1; toàn hệ chưa đo lại) | GIỮ thay đổi B3/B1 (checks 54→80 đạt, 6 màn 900×600 & 1280×880 đạt); ĐỖ chấm tổng 100 | [UI Vườn học & Story Khối 11](2026-11-03_ui-vuon-hoc-va-story-khoi-11.md) |
| 2026-11-03 | Tự cập nhật 3 tầng (`SelfUpdater` + `SelfUpdateTools` + `EvolutionGateJudge`) | — | ĐẠT/ĐẠT/ĐẠT (checks 80→210 đạt, 9/9 invariants, live API + UI đạt) | GIỮ — hợp nhất `Evolution` + `EvolutionGateJudge` vào runtime chính & tự cập nhật 3 tầng | [tự cập nhật và tiến hoá](2026-11-03_tu-cap-nhat-va-tien-hoa.md) |
