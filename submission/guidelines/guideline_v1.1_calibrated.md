# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603143 - Hoàng Thu Hương

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay ignore_region? | Gán từng xe nếu **tách được ranh giới từng cá thể**; ignore_region chỉ khi mờ/đè nhau không tách được — không phụ thuộc gần/xa camera | img_005 | CAL-06 |
| 3 | Hàng nhô ra khỏi xe: box xe có gồm hàng không? | Box xe gồm hàng **gắn cố định**; hàng nhô > ~20% chiều dài xe thì box kết thúc ở biên xe, phần hàng nhô không gán riêng | img_008 | CAL-03 |
| 4 | Người ngồi trong xe ba bánh có mui che (chỉ thấy thấp thoáng): ngưỡng ≥20% có hợp lý không? | Gán person nếu còn thấy rõ bộ phận (tay/đầu) **VÀ** box cao ≥ 20px; che 80-95% còn thấy rõ → gán; che > 95% hoặc chỉ thấy mờ → không gán | img_010 | CAL-04 |

**Câu hỏi chưa chốt** (do không gặp trong batch):
- #1 (xe đạp điện / xe máy điện nhỏ): không xuất hiện trong 28 ảnh
- #5 (kính chắn gió thuộc mask xe hay không): chưa vẽ polygon đủ để phán đoán
- #6 (box máy chồng nhau quá nhiều ảnh đêm): batch không có ảnh đêm với tình huống này
- #7 (mức tin cậy tối thiểu để chấp nhận box): đã chọn ngưỡng theo class trong Part 2–3, không dùng ngưỡng chung

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_014 | Người bán hàng rong vác gánh tre (2 giỏ trên vai), không có bánh xe | YOLO26/E gán nhầm thành three_wheeler | **Xoá nhãn**: gánh hàng rong không phải xe ba bánh (không có bánh xe nhìn thấy) | **Mục 2**: three_wheeler phải có ≥2 bánh xe nhìn thấy; gánh hàng rong / xe đẩy tay không bánh → không gán |
| img_009, img_013, img_010 | Tuk-tuk / xích lô bị tách thành 2 box (thân + mui); xe máy chở hàng cồng kềnh / mái che sạp bán hàng nhầm thành three_wheeler | YOLO26 gán nhầm mái che sạp; YOLOE tách box thành 2 phần | **Gộp box tách đôi** thành 1 three_wheeler; **xoá** box mái che sạp (không phải xe) | **Mục 2**: cả xe lẫn mái/thùng gắn liền là **MỘT vật** → 1 box; mái che sạp bán hàng cố định không phải xe |
| img_004 | Bẫy missing: three_wheeler thật bị xoá khỏi pre-label, không nhận ra thiếu vật khi duyệt | Bẫy lọt (không phải lỗi máy thật) | **Gán lại vật thiếu**; áp quy tắc quét ảnh trước | **Mục 7 (QC)**: quét ảnh **TRƯỚC** khi bật layer pre-label; với lớp nhiều FP phải đếm vật thật trước khi xoá |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | **1.** Chốt three_wheeler phải có bánh xe nhìn thấy (≥2 bánh); gánh hàng rong / xe đẩy tay → không gán<br>**2.** Chốt xe + mái/thùng gắn liền = 1 box; mái che sạp cố định không phải xe<br>**3.** Chốt hàng nhô >~20% chiều dài xe → box dừng ở biên xe<br>**4.** Chốt person che 80-95% còn thấy rõ bộ phận VÀ ≥20px → gán; >95% → không<br>**5.** Chốt ignore_region theo khả năng tách cá thể, không theo khoảng cách<br>**6.** Thêm quy tắc QC: quét ảnh trước khi bật pre-label với lớp có nhiều FP | CAL-01 (img_014 gánh hàng rong), CAL-02 (img_009/013/010 box tách đôi), CAL-03 (img_008 hàng nhô), CAL-04 (img_010 can_review), CAL-06 (img_005 xe dày đặc), CAL-05 (img_004 bẫy missing) |
