# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603098 - Nguyễn Hồng Sơn

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe hai bánh chạy điện: phân loại vào bicycle hay motorbike? | Xe máy điện hoặc xe đạp điện có kết cấu khung xe máy, yên xe máy và vận tốc cao được gán vào `motorbike`. Chỉ gán `bicycle` cho xe đạp cơ học hoặc xe đạp thể thao có trợ lực nhẹ. | `img_009.jpg` | CAL-01 |
| 2 | Xe ba bánh có cabin / mui che lớn: car hay three_wheeler? | Tất cả phương tiện 3 bánh thương mại (tuk-tuk, xe ba gác máy, xe xích lô) đều thuộc lớp `three_wheeler`, không gán `car` dù có nóc che. | `img_013.jpg` | CAL-02 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013.jpg` | Xe tuk-tuk 3 bánh đỗ trên đường | YOLO26 gán nhầm thành `car` do xe có mui che kín; YOLOE phát hiện đúng `three_wheeler` | Giữ nhãn `three_wheeler`, xoá nhãn `car` của YOLO26 | Mục 2.7: Mọi phương tiện 3 bánh chở khách/chở hàng gắn nhãn three_wheeler bất kể kiểu dáng cabin |
| `img_003.jpg` | Người lái và hành khách trên xe ba bánh tricycle | Máy vẽ box bao gồm cả người lái hoặc bỏ sót người lái | Tách riêng từng người thành box `person` riêng; box `three_wheeler` chỉ ôm sát thân xe | Mục 3.1: Người điều khiển và hành khách luôn gán box person riêng biệt, box phương tiện ôm sát thân xe |
| `img_031.jpg` | Xe đạp có người lái bị bẫy gán thành motorbike | Bẫy tiêm nhãn sai `motorbike` | Đổi nhãn về `bicycle` và chỉ vẽ khung xe đạp (không kéo bao trùm cả người) | Mục 3.2: Box xe đạp bicycle chỉ chứa khung và bánh xe, tuyệt đối không trùm lên thân người lái |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | - Làm rõ phân loại xe 2 bánh điện vào `motorbike` (CAL-01).<br>- Khẳng định tuk-tuk và xe 3 bánh có mui che thuộc `three_wheeler`, không phải `car` (CAL-02).<br>- Siết chặt quy tắc tách rời người lái (`person`) khỏi xe (`bicycle`, `three_wheeler`), không vẽ gộp (CAL-03, CAL-04). | CAL-01 (`img_009.jpg`), CAL-02 (`img_013.jpg`), CAL-03 (`img_003.jpg`), CAL-04 (`img_031.jpg`) |
