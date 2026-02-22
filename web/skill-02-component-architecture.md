# SKILL 02 — Component Architecture Layer

## Skill Identity
**Name:** Component Architecture Layer
**Version:** 1.0.0
**Description:** Scalable architecture for large web applications (100-500+ pages). Implements Atomic Design and strict folder patterns.
**Trigger Requirements:**
- "cấu trúc dự án web"
- "component architecture"
- "scaling web app"
- "atomic design"
- "layout system"

## 🎯 Mục tiêu
Tạo cấu trúc tool có thể scale 100–500 pages.

## 📂 Core Knowledge
- **Component-based Architecture:** Chia nhỏ giao diện thành các phần độc lập và có khả năng tái sử dụng.
- **Atomic Design Principle:** Phân loại components thành Atoms, Molecules, Organisms, Templates, và Pages.
- **Reusable UI Components:** Xây dựng thư viện UI dùng chung (Buttons, Inputs, Modals).
- **Layout System:** Phân tách rõ ràng các phần Header / Sidebar / ToolLayout.
- **Route-based Page Structure:** Quản lý routing logic chặt chẽ.
- **Dynamic Metadata:** Tự động hóa SEO/Meta tags cho từng tool/page riêng biệt.

## 📁 Folder Pattern
Mỗi tool/trang nên tuân thủ cấu trúc:
`/tools/[category]/[tool-name]`

## 🛡️ AI rules
- **Mỗi tool = 1 isolated component.**
- **Không hardcode strings hay config.**
- **Tách UI và logic riêng (Separation of Concerns).**

---

## Summary Checklist
1. [ ] Cấu trúc thư mục có đúng mẫu `/tools/[category]/[tool-name]` không?
2. [ ] Các component UI có khả năng tái sử dụng không?
3. [ ] Phần logic xử lý dữ liệu đã được tách ra khỏi phần hiển thị (UI) chưa?
4. [ ] Metadata đã được cấu hình động cho từng trang chưa?
