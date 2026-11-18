# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 428; đối tượng sau duyệt: 403; ngưỡng accepted: IoU >= 0.9
- Đã **loại 30 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 383 | 383/428 = 89.5% | 428 box máy |
| edited (sửa) | 4 | 4/428 = 0.9% | 428 box máy |
| deleted (xoá) | 41 | 41/428 = 9.6% | 428 box máy |
| added (vẽ thêm) | 16 | 16/403 = 4.0% | 403 đối tượng sau duyệt |

Trong số deleted + added, **10 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 6.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 165 | 0 | 2 | 0 |
| bicycle | 2 | 0 | 1 | 2 |
| motorbike | 168 | 3 | 14 | 1 |
| car | 28 | 0 | 2 | 6 |
| bus | 1 | 0 | 1 | 3 |
| truck | 2 | 0 | 0 | 2 |
| three_wheeler | 17 | 1 | 21 | 2 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 41 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 31 | 23 | 10 | 57.4% (KTC 95%: 44–70%) | 75.6% (KTC 95%: 61–86%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 3/3
  - loose: 0/3
  - extra: 3/3
  - wrong_class: 2/3
  - lọt: img_010.jpg (loose), img_023.jpg (loose), img_031.jpg (wrong_class), img_005.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
