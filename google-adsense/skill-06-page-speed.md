# SKILL 06 — Page Speed Optimization

## Skill Identity
**Name:** Page Speed & Core Web Vitals
**Version:** 1.0.0
**Description:** Technical optimization to ensure high performance while loading heavy ad scripts.
**Trigger Requirements:**
- "tối ưu tốc độ adsense"
- "lazy load ads"
- "core web vitals adsense"
- "preconnect google ads"
- "performance expert"

## 🎯 Mục tiêu
Web load cực nhanh để đạt Core Web Vitals (giúp ranking cao hơn và RPM tốt hơn).

## 📂 Core Knowledge
- **Lazy Load Ads:** Chỉ load mã quảng cáo khi người dùng cuộn đến gần vị trí đó.
- **Preconnect Google Ads:** Kết nối sớm tới domain `google-analytics.com` và `googlesyndication.com`.
- **Code Splitting:** Tách biệt code tool và code quảng cáo.
- **CDN:** Sử dụng Cloudflare hoặc Vercel Edge để phân phối assets.
- **Image Compression:** Luôn dùng WebP cho mọi layout images.
- **Remove Render Blocking:** Dùng `async` hoặc `defer` cho tất cả script bên thứ ba.

## 🛡️ AI rules
- **Quảng cáo luôn phải load sau nội dung chính (Late load).**
- **Sử dụng placeholder (min-height) cho khung ads để tránh nhảy trang (CLS).**

---

## Summary Checklist
1. [ ] Script AdSense đã được đặt `async` chưa?
2. [ ] Khung quảng cáo có chiều cao cố định để tránh Layout Shift không?
3. [ ] Đã cấu hình preconnect tới máy chủ Google chưa?
4. [ ] Điểm Lighthouse Mobile có đạt vùng Xanh (90+) không?
