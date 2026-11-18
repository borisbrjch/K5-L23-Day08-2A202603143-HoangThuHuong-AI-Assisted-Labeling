# Báo cáo Day 08 — 2A202603143- Hoàng Thu Hương

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26-m | 0.3 (mặc định) | r05: chạy trên `all` với conf 0.05, cho 877 person, 495 motorbike, 195 car, 58 bicycle, 35 truck, 9 bus. Các lớp COCO này không đạt recall 90% trên gold. |
| three_wheeler | YOLOE-26s | 0.15 | r06: chạy trên `all` với conf 0.05, cho 125 box three_wheeler. Ngưỡng 0.15 đạt recall 100%, precision 78% trên 7 vật gold. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  ```bash
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json \
      --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
  **Thứ tự quan trọng**: YOLOE trước Y26 để bảo vệ three_wheeler khỏi bị loại bởi car/truck boxes (cross-class NMS ở IoU 0.6).

- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1-b (r04: 16 masks, 1.76s/img trên 2 ảnh). Cân bằng tốc độ và chất lượng mask, không cần quyền truy cập đặc biệt như SAM 3.

- Lỗi nào **không sửa được bằng đổi ngưỡng**:
  - **img_001, car → truck**: YOLO26 gán lớp `truck` với conf 0.93 (IoU 0.93 với gold car). Đây là class confusion, không phải thiếu confidence. Sửa: dùng model có taxonomy phù hợp hơn, hoặc remap sau.
  - **img_036, motorbike → person**: box người lái nuốt box xe (IoU người-xe 0.62). Threshold không tách được. Sửa: guideline rõ về việc vẽ riêng người và xe.
  - **img_036, person**: 2 người trong 1 box máy (IoU 0.50 với 1 trong 2 gold person). Sửa: NMS nhẹ hơn hoặc tách tay.
  - **img_020, person**: miss hoàn toàn (IoU 0.00 với gold person box [783,230,827×284]). Đây là detection failure, không sửa được bằng threshold.

- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Trong gold: person recall đỉnh 86% (12/14), motorbike 89% (8/9), car 88% (7/8). False negatives xuất hiện ở threshold floor 0.10, nghĩa là model không tạo ra predictions với conf đủ cao cho các vật này.

- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold ở ngưỡng đã chọn là con số **lạc quan** (chọn và chấm trên cùng dữ liệu). Chỉ 4 ảnh → ước lượng thô. Lớp có ít vật (bicycle n=1, bus/truck n=0) không được chọn ngưỡng riêng là đúng.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 389 | 74.5% | 522 box máy |
| edited | 50 | 9.6% | 522 box máy |
| deleted | 59 | 11.3% | 522 box máy |
| added | 152 | 25.6% | 593 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? **three_wheeler** là lớp máy tệ nhất: 26 deleted / 40 box máy ban đầu = 65% bị xoá, 6 added. Lý do: model gán nhầm gánh hàng rong và xe máy chở hàng cồng kềnh thành three_wheeler (img_014, img_009 theo time_log), bbox bị tách đôi (img_010), và taxonomy không đồng nhất (mỗi xe chở hàng khác nhau). person có 80 added / 253 pre = 31.6% bỏ sót, chủ yếu do vật thể nhỏ / truncated / occluded.
- So với gold: precision / recall của **máy** 58.5% (31 TP / 53 predictions, KTC 95%: 45–71%) / 79.5% (31 TP / 39 gold objects) (mẫu số: 39 đối tượng gold trên 4 ảnh). Con số lạc quan vì ngưỡng được chọn trên chính các ảnh gold này.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **9/12 = 75%** (KTC 95%: 47–91%); loại nào lọt: **extra** (img_013: 1/3), **missing** (img_004: 2/3), **loose** (img_009: 2/3); wrong_class bắt hết 3/3. Vì sao: img_013 extra trap (thêm 1 box person giả) bị bỏ qua vì accepted rate cao (74.5%) tạo kỳ vọng máy đúng → quét nhanh không kỹ; img_004 missing trap (xoá 1 three_wheeler thật) lọt vì three_wheeler có nhiều FP thật → quen với việc thấy box thừa, bỏ sót việc kiểm kỹ vật thể đã xoá; img_009 loose trap (box motorbike rộng hơn 20%) lọt vì nhiều box cần chỉnh nhỏ → mỏi mắt, không đo kỹ từng box.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Accepted 74.5% là cao, nhưng bắt bẫy 75% chỉ đạt ngưỡng, không vượt 80%. Điều này cho thấy **thiên kiến xác nhận**: tin máy đúng khi thấy nhiều box accepted → quét nhanh, bỏ sót lỗi tinh vi (extra, loose). Wrong_class bắt hết vì sai lớp dễ nhận ra hơn; missing/extra/loose cần quét toàn ảnh trước khi nhìn nhãn máy, nhưng workflow CVAT có sẵn box → xu hướng chỉ kiểm box có sẵn, không quét lại ảnh trắng.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay **3.34 phút/ảnh** (4 ảnh gold, tổng 13.3 phút) vs duyệt **3.45 phút/ảnh** (24 ảnh batch, tổng 82.8 phút). Thiên lệch: gần như **không giảm thời gian/ảnh**, nhưng batch có ảnh khó hơn (nhiều vật chồng lấn) và bạn đã quen cảnh khi gán gold trước. Ngoài ra 82.8 phút chia đều nên ảnh đầu mất 5-8 phút để làm quen workflow, trong khi gold_manual đã có ảnh đầu 4.4 phút.
  - Nếu bỏ 3 ảnh đầu (img_003: 5.7p, img_005: 8.4p, img_024: 10.1p là ảnh khó nhất): trung bình còn ≈ 3.07 phút/ảnh, tức nhanh hơn ~8% so với gán tay.
  - **Chất lượng**: 74.5% box máy được giữ nguyên, tiết kiệm đáng kể thao tác vẽ mới. Nhưng thời gian tiết kiệm bị ăn vào bởi việc sửa/xoá: 50 edited + 59 deleted = 109 box/522 (20.9%) phải can thiệp.
- Kết luận: **Có, nhưng chỉ cho person / motorbike / car** (accepted rate: person 219/253 = 86.6%, motorbike 103/133 = 77.4%, car 47/55 = 85.5%). **Không cho three_wheeler** (5/40 = 12.5% accepted, 65% deleted) — tệ hơn vẽ tay. bicycle / bus / truck mẫu quá ít (6-8 box) để kết luận.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. **Quét ảnh trắng trước, mở nhãn máy sau**: mỗi ảnh dành 10-15 giây nhìn ảnh gốc liệt kê vật thể cần gán, rồi mới bật layer pre-label. Kiểm tra: ghi lại số vật tìm được trước khi xem máy trên 5 ảnh ngẫu nhiên, so với số added cuối cùng.
  2. **Kiểm kích thước box một cách có hệ thống**: sau khi duyệt lớp, dùng phím tắt CVAT duyệt qua từng box một, đo nhanh xem box có "ôm sát" không (đặc biệt với vật bị che khuất / truncated). Kiểm tra: tỉ lệ edited ≥ 9.6% hiện tại không giảm về 0 giả tạo.
  3. **Với ba lớp accepted cao (person/motorbike/car), chủ động tìm vật bị bỏ sót**: quét viền ảnh, vùng tối, vật nhỏ < 20px trước khi chấp nhận. Kiểm tra: đếm số added trên ảnh mà máy accepted ≥ 80% — nếu bằng 0 trên mọi ảnh, nghi ngờ thiên kiến xác nhận.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: **6** dòng trong `qc/calibration_decisions.csv` (2 `loi_may`, 1 `bay_lot`, 1 `can_review`, 2 `cau_hoi_mo`). Quan trọng nhất là **CAL-01** và **CAL-02** vì chúng định nghĩa lại chính lớp mà máy tệ nhất: CAL-01 (img_014) chốt `three_wheeler` phải có **≥2 bánh xe nhìn thấy**, nên gánh hàng rong / xe đẩy tay không bánh bị loại khỏi lớp; CAL-02 (img_009, img_013, img_010) chốt **xe + mái/thùng gắn liền là MỘT vật → 1 box**, và mái che sạp bán hàng cố định không phải xe. Hai quyết định này giải thích trực tiếp 26/40 box three_wheeler bị xoá (65%) và 9 box bị sửa ở `review_stats.md`.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: 6 thay đổi, mỗi thay đổi gắn một `case_id` ở mục 3 (lịch sử thay đổi) và có ảnh ví dụ ở mục 2 — (1) three_wheeler phải có bánh, (2) xe + mái/thùng = 1 box, mái che sạp không phải xe, (3) hàng nhô > ~20% chiều dài xe thì box dừng ở biên xe, (4) person che 80–95% còn thấy rõ bộ phận **và** box cao ≥ 20px → gán, che > 95% → không, (5) `ignore_region` theo **khả năng tách cá thể** chứ không theo khoảng cách, (6) quét ảnh **trước** khi bật layer pre-label với lớp nhiều FP. Ba câu hỏi mở của v1.0 (#2, #3, #4) được chốt; bốn câu còn lại (#1, #5, #6, #7) ghi rõ **chưa chốt và vì sao** — lần lượt là: không gặp trong 28 ảnh, chưa vẽ đủ polygon, batch không có ảnh đêm tình huống đó, và đã dùng ngưỡng theo lớp từ Part 2–3.
- Nguyên nhân gốc, phân biệt ba loại: **(a) guideline mơ hồ** — CAL-03, CAL-04, CAL-06: v1.0 có quy tắc "hàng nhô > 20%" nhưng không nói phần hàng nhô được gán thế nào, và hai quy tắc che 80% (4.1.2) với box ≥ 20px (4.1.6) đá nhau mà không có thứ tự ưu tiên. **(b) máy sai** — CAL-01, CAL-02: model thật sự gán gánh hàng rong và mái che sạp thành three_wheeler; đây không phải lỗi của người duyệt và không sửa được bằng đổi ngưỡng (conf của các box này không thấp hơn box đúng). **(c) tôi sai** — CAL-05 (img_004): bẫy missing bị lọt vì tôi quen thấy box three_wheeler thừa nên khi một vật thật bị xoá khỏi pre-label tôi không đối chiếu lại với ảnh gốc; đây là lỗi quy trình của tôi, không phải của guideline hay model.

## 7. QC và quyết định (Part 6)

- Mẫu: **6 ảnh** trong 24 ảnh batch (1 ảnh `can_review` — img_010 — + 5 ảnh ngẫu nhiên, seed 42), **95 đối tượng**; nguồn: `qc/qc_sample.csv`, cách chọn do `scripts/qc_sample.py` thực hiện. Người kiểm: **bạn** (**hạn chế**: vừa là người gắn vừa là người duyệt, nên phiếu này là self-review — cùng một người tạo ra và kiểm tra lỗi).
- Lỗi theo **đối tượng**: **4/95 = 4.2%** · theo **ảnh**: **4/6 = 66.7%** (ngưỡng Release ≤ 2% tính theo ảnh). Hai cách đếm chênh nhau vì lỗi tập trung ở ít ảnh: 3 ảnh sạch, 3 ảnh mỗi ảnh 1–2 lỗi.
- Top lỗi + ảnh ví dụ:
  - **sai_vi_tri_kich_thuoc** — 2 lỗi: img_014 #154 (box person ôm cả gánh hàng tre trên vai thay vì chỉ ôm người), img_003 #87 (box person chỉ ôm nửa thân trên, không tới chân).
  - **bỏ sót** — 2 lỗi: img_006 (thiếu box person cho người lái chiếc motorbike #2), img_017 (thiếu box person cho người đi bộ thứ hai đi song song).
  - **thừa / sai lớp / sai thuộc tính / sai mask**: 0 lỗi trong mẫu này. Đáng chú ý: đúng những loại **không** xuất hiện lại là loại máy sai nhiều nhất ở toàn lô (three_wheeler thừa/sai lớp), nên mẫu 6 ảnh này **không đại diện** cho lỗi three_wheeler.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): với 4/6 ảnh có lỗi, cận trên Wilson là **90%** — tức là mẫu này **không thể chứng minh** tỉ lệ ảnh lỗi thật ≤ 2%. Ngưỡng 2% chỉ kiểm chứng được với cỡ mẫu lớn hơn nhiều (với 0 lỗi, cần ~150 ảnh mới có cận trên dưới 2%). 95 đối tượng cũng quá ít để nói gì về lỗi theo lớp.
- Script kiểm được / không kiểm được: `check_submission.py` và `validate_labels.py` kiểm được **hợp lệ cấu trúc** (đủ ảnh, đủ trường, cat_id đúng taxonomy, box nằm trong ảnh) và cảnh báo **box cao < 20px** (bắt được 17 cảnh báo ở img_018/025/027/033 — đây là các box đáng lẽ phải là `ignore_region`). Script **không** kiểm được: box có **ôm sát** vật không, có **bỏ sót** vật không, thuộc tính `occluded`/`rotation` có đúng không, và **cùng một thiên kiến bỏ sót với máy** — máy bỏ sót vật nhỏ/che khuất, người duyệt nhìn pre-label cũng dễ bỏ sót đúng những vật đó. Ba loại này chỉ phát hiện được bằng mắt trên ảnh gốc, đúng theo checklist đã ghi ở mục 4.

**Quyết định: REWORK**, ngày 2026-11-19. Đi theo cây Hold → Rework → Release: **Hold** không áp dụng vì không còn cờ guideline-gap hay data-problem (v1.1 đã chốt 3 câu hỏi mở, `validate_labels.py --all` sạch, không có ảnh hỏng). **Rework** áp dụng vì hai điều kiện của cây cùng vi phạm: (i) tỉ lệ ảnh có lỗi **66.7% (4/6)** vượt xa ngưỡng 2%; (ii) bắt bẫy **9/12 = 75%** thấp hơn ngưỡng 80% → lô duyệt chưa đủ tin cậy để Release.
- Việc cần làm tiếp (Rework): (1) **duyệt lại** 3 ảnh có lỗi theo checklist "quét ảnh trắng trước khi bật pre-label" — img_003 (sửa box #87 cho ôm đủ thân), img_014 (tách gánh hàng khỏi box person #154), img_006/img_017 (thêm box person bị bỏ sót); (2) **kiểm lại các ảnh three_wheeler** trong 24 ảnh vì loại lỗi máy sai nhiều nhất (thừa/sai lớp) không hề xuất hiện trong mẫu 6 ảnh → mẫu chưa đại diện; (3) sau khi sửa, lấy mẫu QC lớn hơn (≥ 10 ảnh, hoặc toàn bộ 24 ảnh cho lần chốt) để có cận trên Wilson thực sự dưới 2%.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): gpt (xác định, phân rã và phản biện vấn đề), claude (viết script, copy nội dung báo cáo chính xác từ exported file và tôi tự kiểm tra lại)
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? không 
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: không
- Dùng cache `data/cache/` thay vì tự chạy? không
- Lỗi / bug script đã gặp (nếu có): một số lỗi nhỏ khi chạy script notebook hoặc import /export annotation cvat.
- Một điều bạn làm khác lần sau: <<>>
