# Báo cáo Day 08 — 2A202603098 - Nguyễn Hồng Sơn

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26-n | conf=0.25 (sweep: motorbike=0.1, car=0.45) | `r01`, `r02`: YOLO26-n chạy nhẹ trên CPU (~0.38s/ảnh), nhận diện tốt các lớp COCO cơ bản ở cự ly gần và trung bình. |
| three_wheeler | YOLOE-26s, prompt: three-wheeled motorized vehicle, auto rickshaw, tuk-tuk | conf=0.15 | `r03`: YOLOE zero-shot phát hiện được xe ba bánh đặc thù châu Á mà YOLO COCO bị mù hoàn toàn. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py --base work/prelabel/y26_all/pre.coco.json --extra work/prelabel/yoloe_all/pre.coco.json --extra-classes three_wheeler --thr "motorbike=0.1,car=0.45,three_wheeler=0.15" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn `mobile_sam.pt` (SAM 1 Mobile / lightweight) vì bài thực hành chạy hoàn toàn trên CPU máy cá nhân trong môi trường WSL2 (RAM cấp phát 6GB, không GPU rời). Mobile SAM tạo mask nhanh (~2s/ảnh), tiêu thụ RAM thấp (<2GB) giúp tránh lỗi OOM crash, đường biên mask sắc nét và không đòi hỏi quyền truy cập hay token online như SAM 3.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi phân loại nhầm giữa các lớp có hình dáng tương đồng (như xe tuk-tuk bị YOLO26 gọi là `car` do có mui che và 4 chỗ, hoặc xích lô bị gọi là `bicycle`). Đổi ngưỡng confidence chỉ lọc bớt hoặc lấy thêm box chứ không đổi tên lớp của mô hình. Cách sửa: Dùng mô hình chuyên biệt bổ sung (như YOLOE với prompt text riêng cho `three_wheeler`) hoặc người duyệt sửa thủ công đổi lớp (`changed_cls`).
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - Điểm cao mà sai: Ở `img_013.jpg`, xe tuk-tuk bị YOLO26 gán `car` với conf 0.78 do xe có cabin lớn.
  - Điểm thấp mà đúng: Ở `img_009.jpg`, xe máy ở hậu cảnh xa có conf chỉ 0.12 nhưng vị trí hoàn toàn chuẩn xác.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Tập gold chỉ có 4 ảnh (41 đối tượng), sweep ngưỡng trực tiếp trên chính tập này (in-sample) dẫn đến hiện tượng over-fitting, khoảng tin cậy 95% rất rộng (Precision 44–70%, Recall 61–86%).

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 383 | 89.5% | 428 box máy |
| edited | 4 | 0.9% | 428 box máy |
| deleted | 41 | 9.6% | 428 box máy |
| added | 16 | 4.0% | 403 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp `three_wheeler` máy tệ nhất: bị xoá 21 box và chỉ giữ 17 box. Nguyên nhân do xe ba bánh ở phố châu Á có hình dáng biến thể rất đa dạng (từ xe lôi, tuk-tuk đến xe ba gác tự chế, ví dụ ở `img_003.jpg`, `img_010.jpg`).
- So với gold: precision / recall của **máy** Precision 57.4% (KTC 95%: 44–70%) / Recall 75.6% (KTC 95%: 61–86%) (mẫu số: 41 đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 8/12 = 66.7% (KTC 95%: 39–86%); loại nào lọt: 3 bẫy `loose` (`img_005.jpg`, `img_010.jpg`, `img_023.jpg`) và 1 bẫy `wrong_class` (`img_031.jpg`); vì sao (bạn đã bỏ bước nào của checklist)?
  - Với bẫy `loose`: Người duyệt chỉ nhìn lướt qua thấy có box bao quanh xe là bấm accept mà không zoom cận cảnh kiểm tra mép viền (bỏ qua bước zoom kiểm tra tight box theo checklist).
  - Với `img_031.jpg`: Đã phát hiện xe đạp nhưng khi kéo chỉnh lại kéo trùm cả người lái, khiến diện tích box tăng gấp 3 lần làm IoU giảm xuống 0.34 (< 0.50).
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Tỉ lệ accepted cao (89.5%) trong khi bẫy loose lọt 3/3 phản ánh rõ nét hiện tượng Tự động hóa thiên kiến (Automation Bias): người duyệt có xu hướng tin tưởng vào vị trí máy vẽ, chỉ tập trung tìm các đối tượng bị bỏ sót (`missing`) hoặc box rác thừa (`extra`), dẫn đến việc buông lỏng kiểm soát mép biên box.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 5.0 phút/ảnh (20 phút / 4 ảnh) vs duyệt 2.5 phút/ảnh (60 phút / 24 ảnh). Thiên lệch (bạn gán gold trước nên đã quen cảnh): Tốc độ duyệt nhanh gấp đôi (tiết kiệm 50% thời gian), tuy nhiên một phần do đã quen thuộc với bối cảnh và nhãn từ bước làm gold.
- Kết luận: Có đáng dùng, đặc biệt rất hiệu quả cho các lớp đối tượng phổ biến như `person`, `motorbike`, `car` (tỉ lệ accepted > 90%). Tuy nhiên đối với lớp đặc thù như `three_wheeler`, máy sót và nhầm nhiều nên vẫn cần người kiểm soát chặt chẽ.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Quy tắc quét 5 giây không nhãn: Tắt hiển thị nhãn máy trong 5 giây đầu tiên khi mở ảnh để mắt quét tự nhiên các đối tượng bị bỏ sót trước khi bật nhãn máy lên.
  2. Quy tắc zoom 200% kiểm tra viền: Luôn zoom tối thiểu 200% vào ít nhất 2 phương tiện lớn trên mỗi ảnh để kiểm tra độ ôm sát viền trước khi bấm chấp nhận.
  3. Quy tắc rà soát danh sách phân loại (Class Checklist): Đối chiếu checklist các cặp lớp dễ nhầm (tuk-tuk vs car, bicycle vs motorbike) thay vì chỉ nhìn vào khung toạ độ.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | Mobile SAM (`mobile_sam.pt`) | 3 click (2 pos, 1 neg) | Không gồm người lái, ôm sát thân tuk-tuk | Click ban đầu bị lan mask sang người lái, click neg đã tách sạch | `masks/tuktuk_img013.jpg` |
| `img_004.jpg` | Mobile SAM (`mobile_sam.pt`) | 1 click pos | Chỉ gồm xe máy, tách riêng người lái | Mask bám sát bánh xe và yếm xe | `masks/motorbike_img004.jpg` |
| `img_003.jpg` | Mobile SAM (`mobile_sam.pt`) | 1 click pos | Chỉ gồm thân xe ba bánh | Mask bám tốt khung xe xích lô | `masks/tricycle_img003.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Quy tắc tách biệt người lái khỏi phương tiện giúp định hướng điểm click âm (negative point) để ngắt vùng tràn mask; tuy nhiên guideline còn thiếu hướng dẫn cụ thể về việc xử lý hàng hóa cồng kềnh buộc phía sau xe máy/xe ba bánh.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: Đã thử nghiệm offline qua script `sam2_points.py` với `mobile_sam.pt`, ưu điểm chạy trực tiếp trên CPU mượt mà và kiểm soát toạ độ click chính xác.

## 6. Calibration (Part 6)

- Số quyết định: 4 quyết định (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): `CAL-02` (khẳng định tuk-tuk và xe ba bánh có cabin thuộc `three_wheeler`, không gán `car`).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Phân định xe 2 bánh điện vào `motorbike`, xe 3 bánh có mui vào `three_wheeler`, và siết chặt quy tắc tách người lái (`person`) khỏi xe (`bicycle`, `three_wheeler`).
- Nguyên nhân gốc: Kết hợp cả guideline v1.0 còn chỗ mở (xe đạp điện) và máy pre-label nhầm lẫn do đặc thù xe cộ phố châu Á.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh batch ngẫu nhiên (seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: 0/86 = 0.0% · theo **ảnh**: 0/5 = 0.0% (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: Lỗi vị trí kích thước (box hơi lỏng ở xe ba bánh `img_010.jpg`).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Mẫu 5 ảnh là nhỏ, khoảng tin cậy Wilson rộng nên không phản ánh hết toàn bộ phương sai của lô dữ liệu.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): Script tự động không kiểm tra được độ ôm sát viền thực tế và các vật mà cả máy lẫn người cùng bỏ sót.

**Quyết định: Rework**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): Tỉ lệ bắt bẫy đạt 66.7% (< ngưỡng yêu cầu 80% trong taxonomy rules), đặc biệt bẫy lệch viền loose lọt 3/3 cho thấy nguy cơ thiên kiến tự động hóa cao. Do đó đưa ra quyết định Rework để yêu cầu rà soát lại toàn bộ lô 24 ảnh, siết chặt kiểm tra độ ôm sát của khung bao trước khi nghiệm thu.
Nếu Rework / Hold, việc cần làm tiếp: Mở lại CVAT, thực hiện rà soát theo checklist 3 quy tắc giảm thiên kiến (zoom 200% kiểm tra viền, rà soát xe ba bánh và người lái).

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Dùng trợ lý AI Antigravity để giải thích các khái niệm (IoU, Wilson score, automation bias), gợi ý lệnh terminal WSL2 và cấu hình môi trường CPU. Toàn bộ thao tác gán nhãn gold, review trên CVAT và đối chiếu số liệu đều do cá nhân tự thực hiện và kiểm chứng bằng các script kiểm tra.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không, 4 ảnh gold được gán thủ công hoàn toàn từ đầu trên CVAT trước khi chạy bất kỳ model pre-label nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11 + WSL2 (Ubuntu), CPU Intel Core i7 (16 vCPU, không có GPU rời). CVAT v2.74.1 qua Docker. Vướng mắc ban đầu: Ubuntu Resolute mặc định Python 3.14 không tương thích PyTorch, đã giải quyết bằng cách cài Python 3.12 qua `uv`.
- Dùng cache `data/cache/` thay vì tự chạy? Không, tự chạy toàn bộ các model YOLO26, YOLOE và Mobile SAM trên CPU máy thật.
- Lỗi / bug script đã gặp (nếu có): Không gặp lỗi logic trong script của repo.
- Một điều bạn làm khác lần sau: Sẽ thực hiện bước zoom cận cảnh 200% để kiểm tra viền box ngay từ ảnh đầu tiên, tránh tâm lý chủ quan khi thấy máy đã bao quát được đối tượng.
