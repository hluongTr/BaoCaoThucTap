---
title: "Chuẩn bị môi trường"
date: 2026-11-07
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

## Điều kiện trước khi bắt đầu

| Thành phần | Cần chuẩn bị |
| --- | --- |
| Repository | Bản CloudDocs đang sử dụng, gồm Back_end, FrontEnd, scripts |
| Công cụ | Git, Java 21, Maven, MySQL client, Node.js; AWS CLI nếu dùng CLI |
| Database | Bản export schema hiện có và dữ liệu thử đã được phép sử dụng |
| AWS | Quyền tạo/quản lý tài nguyên trong môi trường demo được giao |
| Môi trường | Region của S3/RDS/EC2; Region và issuer riêng của Cognito |
| Tên miền | Origin HTTPS dự kiến; URL callback/logout khớp tuyệt đối |

Không giả định S3 và Cognito cùng Region: cấu hình hiện có có thể khác nhau. Backend dùng AWS_REGION cho S3; COGNITO_ISSUER_URI phải thuộc User Pool thực tế.

## Kiểm tra bản nguồn tại máy

Chạy trong repository CloudDocs, không chạy trong thư mục website báo cáo:

~~~powershell
java -version
mvn -version
node --version
mvn -f Back_end/pom.xml package
node --test FrontEnd/tests/*.test.cjs
~~~

Các lệnh build/test không chứng minh đã kết nối AWS thật. Lưu kết quả và phiên bản commit; không viết số ca pass khi chưa chạy.

## Chạy local để có bản đối chiếu

1. Import schema đúng và migration cần thiết theo mục RDS.
2. Cấu hình Cognito public client/callback tại FrontEnd/js/cognito-config.js, API_BASE trong app.js.
3. Dùng scripts/start-local-backend.ps1; script hỏi mật khẩu DB. Nếu pool khác cấu hình mẫu, điều chỉnh issuer/client/UserInfo đồng bộ; script hiện có gán giá trị phát triển cụ thể.
4. Dùng scripts/start-frontend.ps1 hoặc node scripts/serve-frontend.cjs; mở URL script in ra.
5. Kiểm tra /api/health và đăng nhập. Local vẫn dùng Cognito, không có cơ chế đăng nhập mật khẩu local thay thế.

**Đầu ra:** môi trường, commit, schema và cấu hình công khai đã được đối chiếu; không lưu secret vào báo cáo.
