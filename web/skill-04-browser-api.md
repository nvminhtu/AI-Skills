# SKILL 04 — Browser API Integration

## Skill Identity
**Name:** Browser API Integration
**Version:** 1.0.0
**Description:** Deep integration with native browser capabilities to create powerful client-side tools.
**Trigger Requirements:**
- "browser api"
- "file reader"
- "clipboard api"
- "localstorage"
- "canvas api"
- "biến browser thành tool"

## 🎯 Mục tiêu
Tận dụng toàn bộ browser capabilities để xử lý dữ liệu ngay tại máy người dùng.

## 📂 Core Knowledge
- **FileReader API:** Đọc nội dung file (text, dataURL, binary) trực tiếp từ máy người dùng.
- **Clipboard API:** Đọc/Ghi dữ liệu vào clipboard (copy-paste automation).
- **LocalStorage/SessionStorage:** Lưu trữ trạng thái và cấu hình người dùng bền vững.
- **Drag & Drop API:** Trải nghiệm kéo thả file/element mượt mà.
- **Canvas API:** Vẽ, chỉnh sửa và xuất hình ảnh thủ công.
- **Blob & Download Handling:** Tạo file ảo từ dữ liệu (JS object/String) và kích hoạt tải xuống.
- **Web Crypto API:** Mã hóa/Giải mã dữ liệu an toàn ngay tại client.

## 🛡️ AI rules
- **Ưu tiên client-side processing:** Giảm tải cho server và tăng tính riêng tư.
- **Không gửi data lên server nếu không cần:** Tuyệt đối giữ dữ liệu người dùng tại chỗ (offline-first).

---

## Summary Checklist
1. [ ] Dữ liệu có được xử lý 100% tại trình duyệt không?
2. [ ] Có cơ chế fallback cho các trình duyệt cũ không hỗ trợ API mới không?
3. [ ] Các trạng thái cấu hình có được lưu vào LocalStorage không?
4. [ ] Quá trình tải file (download) có hoạt động ổn định không?
