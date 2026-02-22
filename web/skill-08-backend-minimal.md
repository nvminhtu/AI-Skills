# SKILL 08 — Backend Minimal Layer (Optional)

## Skill Identity
**Name:** Backend Minimal Layer
**Version:** 1.0.0
**Description:** Lightweight server-side support for features that cannot be handled purely on the client.
**Trigger Requirements:**
- "backend minimal"
- "file upload server"
- "api rest"
- "authentication basic"
- "rate limiting"

## 🎯 Mục tiêu
Hỗ trợ các tính năng như upload file lớn, lưu trữ database hoặc hệ thống tài khoản.

## 📂 Core Knowledge
- **REST API:** Thiết kế các endpoint chuẩn (GET, POST, PUT, DELETE).
- **File Upload Handler:** Xử lý multipart/form-data và lưu trữ cloud (S3/Cloudinary).
- **Rate Limiting:** Ngăn chặn spam và tấn công DDOS vào các API xử lý nặng.
- **Logging:** Theo dõi lỗi và hành vi người dùng bằng các log system.
- **Basic Authentication:** Quản lý User/Session bằng JWT hoặc Auth providers (NextAuth, Clerk).

## 🛡️ AI rules
- **Giữ backend mỏng nhất có thể:** Tránh đưa logic có thể xử lý ở client lên server.
- **An ninh là ưu tiên:** Luôn validate input và check quyền (policy) ở phía server.

---

## Summary Checklist
1. [ ] Backend có cơ chế giới hạn tần suất (Rate Limit) chưa?
2. [ ] Các thông tin nhạy cảm đã được ẩn bằng Environment Variables chưa?
3. [ ] API có trả về mã lỗi (Status Code) chuẩn không?
4. [ ] Dữ liệu người dùng có được bảo mật không?
