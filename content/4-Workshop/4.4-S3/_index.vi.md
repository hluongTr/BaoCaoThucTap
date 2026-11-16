---
title: "Tạo S3 private và cấu hình CORS"
date: 2026-11-07
weight: 4
chapter: false
pre: " <b> 4.4. </b> "
---

## 1. Tạo hoặc đối chiếu bucket

Trong S3 Console, chọn bucket dự án ở Region đã thống nhất, giữ **Block Public Access**, Object Ownership **Bucket owner enforced** và mã hóa mặc định. Nếu bucket đang dùng có dữ liệu, đối chiếu trước khi chỉnh sửa. Không bật public website cho bucket tài liệu.

Backend hiện sử dụng prefix documents/ và key UUID. Mẫu policy, CORS và script nằm trong Back_end/aws/s3; đây là công cụ thay đổi hạ tầng khi chạy, không phải bước khởi động ứng dụng bắt buộc.

## 2. CORS theo origin thật

Ví dụ nhập tại S3 Console → Permissions → CORS (thay origin):

~~~json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "HEAD"],
    "AllowedOrigins": ["https://YOUR_FRONTEND_HOST"],
    "ExposeHeaders": ["ETag"],
    "MaxAgeSeconds": 3600
  }
]
~~~

Trong Console nhập **mảng rule** như trên. File JSON dùng với AWS CLI có dạng object chứa CORSRules như mẫu repository. Nếu cần local, thêm đúng origin http://localhost:5500 hoặc http://127.0.0.1:5500 đang dùng. CORS không thay thế IAM và không tự làm bucket công khai.

Luồng upload hiện tại đi qua backend nên không phụ thuộc CORS của S3; xem trước/tải xuống trực tiếp vẫn cần CORS phù hợp. Quyền PutObject thuộc IAM/bucket policy của backend. Cấu hình thực hành cũ tại H04 còn cho phép PUT; đây không phải yêu cầu cho upload hiện tại.

## 3. Cấu hình backend

| Biến | Giá trị cần đặt |
| --- | --- |
| AWS_REGION | Region bucket thật |
| S3_BUCKET | Tên bucket của nhóm |
| S3_PRESIGN_EXPIRY | Thời gian giây; mặc định hiện có 900, có thể giảm theo nhu cầu |

## 4. Kiểm tra luồng chuẩn

1. Đăng nhập, chọn tệp và mô tả tùy chọn tại giao diện; thư mục đích là thư mục của chính tài khoản.
2. Frontend gửi POST /api/documents/upload dạng multipart gồm file, metadata, requestId (UUID) và folderId tùy chọn. Dùng giao diện/DTO hiện hành, không gửi JSON theo phỏng đoán.
3. Backend kiểm tra quyền/hạn mức, tải lên S3 bằng object key riêng và HEAD object; metadata chỉ commit khi toàn bộ bước xử lý thành công.
4. Khi mất phản hồi, thử lại cùng requestId và cùng tệp để nhận lại tài liệu đã lưu; lỗi giao dịch rollback dữ liệu và thử dọn object. Theo dõi log nếu dọn thất bại.
5. GET preview-url/download-url bằng tài khoản có quyền, mở presigned GET mới được cấp. Hai API tạo metadata riêng và lấy upload-url cũ đã bị chặn; không dùng lại hướng dẫn PUT trực tiếp từ trình duyệt.

**Kết quả mong đợi:** object private tồn tại, metadata xác nhận upload và tài khoản khác bị từ chối khi chưa được chia sẻ. URL ký trước có thể còn dùng được tới khi hết hạn; credential tạm hết hạn có thể làm URL hết hiệu lực sớm hơn.

## Lỗi thường gặp

403 khi xem/tải: kiểm tra principal ký, policy, prefix, Region và URL hết hạn. CORS: so origin đầy đủ scheme/host/port; test từ browser. Upload lỗi: kiểm tra log backend, quyền PutObject/GetObject và kết nối S3. HTTP 413: đối chiếu giới hạn multipart tại backend/Nginx. Không tự xóa bản ghi lỗi cũ khi chưa kiểm tra object S3 tương ứng.

Tham khảo [CORS](https://docs.aws.amazon.com/AmazonS3/latest/userguide/cors.html) và [presigned URL](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html).

<!-- workshop-evidence -->

## Thực hành và kết quả minh họa

### H01. Bucket và Region

Mở S3 → Buckets. Bucket clouddocs-dev-private nằm tại Asia Pacific (Singapore), ap-southeast-1; dùng đúng Region này cho S3 client của backend.

![H01. Bucket và Region](/images/workshop-evidence/03-hai-luong-s3/h01-s3-bucket-region.png)

### H02. Chặn truy cập công khai

Trong Permissions, Block all public access là On và Object Ownership là Bucket owner enforced. Tài liệu được truy cập qua quyền được cấp, không qua website public của bucket.

![H02. Chặn truy cập công khai](/images/workshop-evidence/03-hai-luong-s3/h02-s3-block-public-access.png)

### H03. Bucket policy chéo tài khoản

Bucket policy cho phép CloudDocsEc2Role của tài khoản 861358180788 đọc, ghi và xóa object tại documents/*. Đối chiếu ARN role với T02/T05 và permission policy phía role; đây là truy cập bằng resource policy, không phải bằng chứng ứng dụng đã tự AssumeRole sang một role S3 khác.

![H03. Bucket policy chéo tài khoản](/images/workshop-evidence/03-hai-luong-s3/h03-s3-bucket-policy.png)

### H04. CORS đã lưu

Trong Permissions → CORS, cấu hình đã lưu cho phép GET, PUT, HEAD từ http://54.255.190.1; ExposeHeaders gồm ETag và Content-Disposition, MaxAgeSeconds là 300. Đây là origin của lần chụp; nếu đổi host hoặc cổng frontend phải đồng bộ lại.

![H04. CORS đã lưu](/images/workshop-evidence/03-hai-luong-s3/h04-s3-cors.png)

### H05. Mã hóa mặc định SSE-S3

Mở Properties → Default encryption. Encryption type là SSE-S3, sử dụng khóa do Amazon S3 quản lý; không mô tả cấu hình này thành SSE-KMS.

![H05. Mã hóa mặc định SSE-S3](/images/workshop-evidence/03-hai-luong-s3/h05-s3-default-encryption.png)

### H06. Object đã lưu trong documents/

Mở Objects → documents/ để đối chiếu tên key, loại tệp, kích thước và thời điểm cập nhật. Danh sách có cả key của dữ liệu thử cũ và mới; không suy ra tất cả object đều được tạo từ cùng một phiên bản ứng dụng.

![H06. Object đã lưu trong documents/](/images/workshop-evidence/03-hai-luong-s3/h06-s3-uploaded-object.png)

### H07. Thuộc tính một object

Chọn tệp DeCuongThucTapChuyenNganh.docx → Properties. Ảnh ghi nhận key trong documents/, dung lượng 36.9 KB, Storage class Standard và SSE-S3; Bucket Versioning đang Disabled. Thuộc tính này bổ sung minh chứng lưu trữ cho danh sách H06.

![H07. Thuộc tính một object](/images/workshop-evidence/03-hai-luong-s3/h07-s3-object-properties.png)
