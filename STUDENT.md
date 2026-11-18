# Thông tin học viên

- Họ và tên: Hoàng Thu Hương
- MSHV: 2A202603143
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: https://github.com/borisbrjch
- Repo bài nộp (URL): https://github.com/borisbrjch/K5-L23-Day08-2A202603143-HoangThuHuong-AI-Assisted-Labeling
- Commit chốt bài (hash): <<>>
- Máy: Apple M1 8GB
- CVAT (Docker): <<2.74.1>>

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: **389 / 50 / 59** (mẫu số: 522 box máy); added: **152** (mẫu số: 593 đối tượng)
- So với gold: precision / recall của **máy**: **58.5% / 79.5%** (39 đối tượng gold, 4 ảnh; KTC 95%: 45–71% / 64–89%)
- Bắt bẫy: **9/12 = 75%** (KTC 95%: 47–91%)
- Thời gian: gán tay **3.34 phút/ảnh** (4 ảnh); duyệt có pre-label **3.45 phút/ảnh** (24 ảnh)
- Lỗi QC: **66.7% theo ảnh (4/6 ảnh mẫu)**, **4.2% theo đối tượng (4/95 đối tượng)**; cận trên Wilson 95% của tỉ lệ ảnh lỗi: **90%**
- **Quyết định: Rework** — tỉ lệ ảnh có lỗi 4/6 vượt ngưỡng 2% và bắt bẫy 75% dưới ngưỡng 80%; cần duyệt lại ảnh có lỗi và kiểm lại lớp three_wheeler trước khi Release.
