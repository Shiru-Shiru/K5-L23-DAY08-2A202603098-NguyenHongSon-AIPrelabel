# Thông tin học viên

- Họ và tên: Nguyễn Hồng Sơn
- MSHV: 2A202603098
- Lớp (L2 / L3 / khác): K5-L23
- Tài khoản GitHub: Shiru-Shiru
- Repo bài nộp (URL): https://github.com/Shiru-Shiru/K5-L23-DAY08-2A202603098-NguyenHongSon-AIPrelabel
- Commit chốt bài (hash): (cập nhật sau khi git commit cuối)
- Máy: Windows 11 + WSL2 (Ubuntu), CPU Intel Core i7 (không có GPU rời)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 383 / 4 / 41 (mẫu số: 428 box máy); added: 16 (mẫu số: 403 đối tượng)
- So với gold: precision / recall của **máy**: 57.4% / 75.6%
- Bắt bẫy: 8/12 = 66.7%
- Thời gian: gán tay 5.0 phút/ảnh; duyệt có pre-label 2.5 phút/ảnh
- Lỗi QC: 0.0% theo ảnh (0/5), 0.0% theo đối tượng (0/86)
- **Quyết định:** Rework — lý do một dòng: Tỉ lệ bắt bẫy đạt 66.7% (< ngưỡng 80%), lọt 3/3 bẫy loose do thiên kiến tự động hóa, yêu cầu duyệt lại viền box toàn lô.
