---
title: "Dọn dẹp và bàn giao"
date: 2026-11-07
weight: 9
chapter: false
pre: " <b> 4.9. </b> "
---

## Trước khi dọn dẹp

Chỉ dọn tài nguyên demo thuộc phạm vi được phép. Kiểm tra tài nguyên còn phục vụ đồ án đến 28/11 hay không; không xóa hạ tầng dùng chung khi kết thúc một buổi workshop. Lập danh mục resource ID, Region/account, chủ sở hữu và dữ liệu cần giữ. Hướng dẫn này không thực hiện xóa tài nguyên AWS.

## Bàn giao

Lưu commit/JAR và hướng dẫn khởi động, danh mục cấu hình không chứa secret, schema/migration đã áp dụng, kết quả test và ảnh minh chứng. Bản sao lưu dữ liệu phải được lưu ở nơi được phép; kiểm tra khả năng phục hồi trên bản sao. Signed URL và access token không phải nội dung bàn giao.

## Thứ tự dọn môi trường demo sau khi được xác nhận

1. Dừng truy cập/tiến trình ứng dụng, ghi lại cấu hình cần giữ.
2. Sao lưu DB nếu cần, sau đó xóa RDS demo đúng phạm vi; rà soát final snapshot và retained backup còn phát sinh phí.
3. Xử lý tệp thử trên S3 đã xác định. Nếu bucket bật Versioning, rà soát phiên bản/delete marker riêng; xóa object hiện tại không tự xóa mọi version.
4. Terminate EC2 demo khi hết nhu cầu; kiểm tra EBS còn giữ, snapshot và public IPv4/Elastic IP.
5. Xóa Cognito pool/client demo nếu không còn dùng; không xóa pool nhóm đang sử dụng.
6. Rà soát CloudWatch log groups/alarms, tài nguyên mạng như NAT/endpoint, IAM role/policy và các tài nguyên liên quan thật sự đã tạo.
7. Kiểm tra Billing các Region sau thời gian cập nhật; dừng EC2 không tự ngừng chi phí EBS, snapshot hoặc IP còn cấp phát.

**Kết quả mong đợi:** có biên bản bàn giao và danh sách tài nguyên giữ/xóa; không ghi “chi phí bằng 0” khi chưa đối soát hóa đơn.
