# SKILL 05 — Image & Media Processing Layer

## Skill Identity
**Name:** Image & Media Processing
**Version:** 1.0.0
**Description:** Web-based media processing engine. Focuses on client-side image manipulation and format conversion.
**Trigger Requirements:**
- "xử lý ảnh"
- "resize image browser"
- "watermark javascript"
- "base64 image"
- "canvas manipulation"

## 🎯 Mục tiêu
Build tool image/video xử lý trực tiếp trên browser không cần backend.

## 📂 Core Knowledge
- **Canvas Manipulation:** Chỉnh sửa pixel, filter màu, cắt ghép ảnh.
- **Image Resize/Compress:** Tối ưu dung lượng và kích thước ảnh bằng Canvas/OffscreenCanvas.
- **Watermark Engine:** Chèn text/logo bản quyền vào ảnh hàng loạt.
- **Base64 Conversion:** Chuyển đổi giữa file/blob và chuỗi base64.
- **GIF Splitting Logic:** Tách frame từ file GIF hoặc tạo GIF từ ảnh/video.
- **WASM Integration:** Sử dụng WebAssembly (như FFmpeg.wasm) cho các tác vụ media nặng.

## 🛡️ AI rules
- **Tối ưu bộ nhớ:** Giải phóng URL.createObjectURL() và canvas context sau khi dùng.
- **Xử lý bất đồng bộ:** Sử dụng Web Workers để tránh treo giao diện khi xử lý ảnh lớn.

---

## Summary Checklist
1. [ ] Ảnh sau khi nén có giữ được chất lượng chấp nhận được không?
2. [ ] Công cụ đóng dấu (watermark) có hỗ trợ nhiều vị trí không?
3. [ ] Việc xử lý ảnh có làm đơ (block) main thread của UI không?
4. [ ] Có hỗ trợ xuất ra nhiều định dạng (PNG, JPEG, WebP) không?
