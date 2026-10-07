# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Đinh Văn Hùng |
| MSSV | 2A202602443 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/hungdinh82/K4-L3-DAY21-DinhVanHung-2A202602443-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

Tôi chọn `n_estimators=200`, `learning_rate=0.1`, `max_depth=5` vì lần chạy 3 có F1 cao nhất, 0.7149, vượt quality gate 0.65. Accuracy cao nhất lại là lần 1 (0.8780), không trùng cấu hình có F1 cao nhất. Accuracy có thể che khuất lớp thiểu số. Lần 2 ít cây, học chậm, cây nông nên F1 chỉ 0.6051. Giảm `learning_rate` thường cần tăng `n_estimators`; cấu hình 3 dùng nhiều cây, sâu hơn nên F1 tốt hơn.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Lớp dương `target=1` (thu nhập trên 50K) chiếm khoảng 24,8% dữ liệu. Vì vậy một mô hình luôn đoán “thu nhập thấp” vẫn có thể đạt accuracy xấp xỉ 75%, nhưng không tìm được một mẫu dương nào. Accuracy không phân biệt được loại sai sót này. F1 của riêng lớp dương kết hợp precision và recall: mô hình chỉ có F1 tốt khi vừa phát hiện được nhiều người thu nhập cao, vừa hạn chế dự đoán dương sai. Do đó quality gate đặt `f1_score >= 0.65` bảo vệ đúng mục tiêu nghiệp vụ hơn accuracy. Mã nguồn gọi trực tiếp `f1_score(y_eval, preds)`, không dùng `average="weighted"` hay `average="macro"`; các cách trung bình này có thể làm ảnh hưởng của lớp đa số che lấp chất lượng thật của lớp cần quan tâm.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| DVC không kết nối được Google Cloud Storage | Python cục bộ thiếu CA certificate | Dùng `certifi` qua biến `SSL_CERT_FILE`, sau đó `dvc push` thành công. |
| Không tạo được JSON key ở project cũ | Project kế thừa policy của organization chặn service-account key | Tạo project `No organization`, bucket và service account mới. |
| Release bị lỗi sudo qua SSH | GitHub Actions không có terminal để nhập mật khẩu sudo | Thêm sudoers tối thiểu cho git, pip và systemctl của tài khoản triển khai. |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (22.361 mẫu) | 0.7149 | 0.8740 |
| Bước 3 (44.722 mẫu) | 0.7354 | 0.8820 |

Sau khi ghép batch 2, F1 tăng từ 0.7149 lên 0.7354 và accuracy tăng từ 0.8740 lên 0.8820. Chênh lệch nhỏ vì hai batch cùng phân phối. Commit thay đổi tệp `.dvc` đã tự kích hoạt đủ Unit Test, Train, Quality Gate và Release, đồng thời API VM nhận mô hình mới.
