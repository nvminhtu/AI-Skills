# SKILL 07 — Performance Engineering

## Skill Identity
**Name:** Performance Engineering
**Version:** 1.0.0
**Description:** Advanced techniques to achieve < 1s load time and perfect Lighthouse scores.
**Trigger Requirements:**
- "tối ưu tốc độ web"
- "fast load time"
- "bundle optimization"
- "lazy loading"
- "performance web"

## 🎯 Mục tiêu
Đảm bảo website load dưới 1s và đạt độ mượt mà cao nhất.

## 📂 Core Knowledge
- **Code Splitting:** Chia nhỏ file JS để chỉ load những gì cần thiết cho trang hiện tại.
- **Lazy Loading:** Chì load hình ảnh và components khi chúng xuất hiện trên màn hình.
- **Dynamic Import:** Sử dụng `import()` để load các thư viện nặng (ví dụ: Highlight.js, Chart.js) theo yêu cầu.
- **Bundle Optimization:** Loại bỏ code thừa (Tree-shaking) và thu nhỏ (Minification) assets.
- **Debounce/Throttle:** Hạn chế số lần thực thi các hàm xử lý sự kiện liên tục (scroll, resize, input).
- **Caching Strategy:** Sử dụng Service Workers và Cache-Control headers hiệu quả.

## 🛡️ AI rules
- **Không import toàn bộ tool logic vào homepage:** Chỉ load danh sách tool ở trang chủ.
- **Tối ưu bundle size:** Luôn kiểm tra kích thước library trước khi cài đặt.

---

## Summary Checklist
1. [ ] Điểm Google Lighthouse (Performance) có trên 90 không?
2. [ ] Các thư viện nặng có được dán nhãn `dynamic import` không?
3. [ ] Hình ảnh đã được convert sang định dạng WebP và có lazy load chưa?
4. [ ] Trang chủ có nhẹ và load tức thì không?
