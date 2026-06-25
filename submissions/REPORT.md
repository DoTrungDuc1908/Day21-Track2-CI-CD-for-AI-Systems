# Báo cáo thực hành MLOps (Wine Quality CI/CD)

*   **Học viên**: Đỗ Trung Đức
*   **Mã sinh viên**: 2A202600918
*   **Repository URL**: https://github.com/DoTrungDuc1908/Day21-Track2-CI-CD-for-AI-Systems

---

## 1. Kết quả thực nghiệm MLflow UI (Bước 1)
*   **Bộ siêu tham số tốt nhất chọn**:
    *   `n_estimators`: 200
    *   `max_depth`: 20
    *   `min_samples_split`: 2
*   **Lý do**: Khi chạy thực nghiệm cục bộ với 6 tổ hợp siêu tham số khác nhau, bộ cấu hình này đem lại độ chính xác cao nhất trên tập đánh giá `eval.csv` là **0.6840** (so với mặc định ban đầu là 0.5640).

---

## 2. So sánh độ chính xác trước và sau khi gộp dữ liệu mới (Bước 2 & 3)

| Giai đoạn | Số mẫu dữ liệu | Độ chính xác (Accuracy) | Trạng thái Eval Gate | Trạng thái Deploy |
|---|---|---|---|---|
| **Bước 2** (Phase 1) | 2998 mẫu | **0.6840** | **FAIL** (Accuracy < 0.70) | Bị chặn deploy (Hành vi đúng thiết kế) |
| **Bước 3** (Phase 1 + 2) | 5996 mẫu | **0.7460** | **PASS** (Accuracy >= 0.70) | Tự động Deploy thành công lên EC2 VM |

**Nhận xét**: Việc bổ sung 2998 mẫu dữ liệu mới làm tăng lượng dữ liệu huấn luyện gấp đôi giúp cải thiện độ chính xác tăng thêm **6.2%** (từ 0.6840 lên 0.7460), vượt ngưỡng tối thiểu `0.70` để tự động triển khai phiên bản mô hình mới lên máy chủ ảo.

---

## 3. Khó khăn gặp phải và giải pháp

1.  **Lỗi quyền bảo mật SSH key `.pem` trên Windows**:
    *   *Khó khăn*: OpenSSH Client trên Windows báo lỗi key file permissions too open và từ chối kết nối.
    *   *Giải quyết*: Chạy lệnh `icacls` trên PowerShell để thu hồi quyền kế thừa, chỉ cấp quyền Đọc/Ghi duy nhất cho tài khoản User hiện tại.
2.  **Lỗi đẩy file workflow lên GitHub**:
    *   *Khó khăn*: GitHub từ chối push do Token đăng nhập thiếu quyền `workflow`.
    *   *Giải quyết*: Cấp thêm scope `workflow` cho GitHub Personal Access Token và cập nhật lại URL remote của Git.
3.  **Thay đổi Cloud Provider sang AWS**:
    *   *Khó khăn*: Hướng dẫn gốc sử dụng GCP GCS, trong khi yêu cầu triển khai sử dụng AWS S3 và EC2.
    *   *Giải quyết*: Cấu hình DVC trỏ về S3 remote, viết lại logic tải/đẩy mô hình bằng thư viện `boto3` trong file `serve.py` và cấu hình lưu trữ AWS Credentials trong GitHub Actions workflow.
