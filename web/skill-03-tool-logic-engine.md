# SKILL 03 — Tool Logic Engine

## Skill Identity
**Name:** Tool Logic Engine
**Version:** 1.0.0
**Description:** Client-side processing engine for utility tools. High focus on performance, regex, and pure functions.
**Trigger Requirements:**
- "logic xử lý tool"
- "tool engine"
- "regex processing"
- "data conversion tool"
- "string manipulation engine"

## 🎯 Mục tiêu
Xây engine xử lý logic client-side.

## 📂 Core Knowledge
- **String Manipulation:** Truncate, slugify, camelCase, kebab-case, v.v.
- **Regex Engine:** Tìm kiếm, thay thế và trích xuất dữ liệu bằng Regular Expressions.
- **Encoding/Decoding:** Hỗ trợ Base64, URL Encoding, HTML Entities.
- **Data Parsing:** Chuyển đổi qua lại giữa JSON, CSV, XML.
- **Random Generators:** Tạo mật khẩu, chuỗi ngẫu nhiên, UUID.
- **Date & Time Utilities:** Định dạng, tính toán khoảng cách thời gian (Moment/date-fns style).
- **Math Calculation:** Các công thức tính toán logic phức tạp.

## 🛡️ AI rules
- **Logic thuần function (Pure functions):** Cùng một input luôn cho ra cùng một output.
- **Không phụ thuộc UI:** Logic phải hoạt động độc lập với framework UI (React/Vue/v.v.).
- **Testable:** Có thể viết unit test dễ dàng.

---

## Summary Checklist
1. [ ] Các hàm xử lý có phải là pure functions không?
2. [ ] Logic có bị dính chặt vào UI component không?
3. [ ] Các biểu thức Regex đã được tối ưu và kiểm tra chưa?
4. [ ] Có xử lý các trường hợp biên (edge cases) cho việc parse dữ liệu không?
