# SKILL 10 — Deployment & DevOps Layer

## Skill Identity
**Name:** Deployment & DevOps
**Version:** 1.0.0
**Description:** Production-ready deployment workflows, CI/CD, and scaling infrastructure.
**Trigger Requirements:**
- "deploy web"
- "devops workflow"
- "ci/cd pipeline"
- "edge deployment"
- "environment variables"

## 🎯 Mục tiêu
Deploy website lên production ổn định, bảo mật và có khả năng tự động hóa cao.

## 📂 Core Knowledge
- **Git Workflow:** Quản lý nhánh (main/dev), Pull Requests và Semantic Versioning.
- **Environment Variables:** Quản lý API keys và cấu hình riêng biệt cho từng môi trường (Dev/Staging/Prod).
- **CI/CD:** Tự động build, test và deploy (GitHub Actions, Vercel, Netlify).
- **CDN Integration:** Phân phối nội dung tĩnh qua hệ thống mạng toàn cầu để giảm độ trễ.
- **Edge Deployment:** Chạy logic xử lý gần người dùng nhất (Vercel Edge Functions / Cloudflare Workers).

## 🛡️ AI rules
- **Tuyệt đối không hardcode API keys hoặc secrets.**
- **Luôn kiểm tra build thành công ở local trước khi push lên Git.**

---

## Summary Checklist
1. [ ] Môi trường Production đã được cấu hình đầy đủ Env chưa?
2. [ ] Pipeline CI/CD có thông báo lỗi khi build thất bại không?
3. [ ] Website có hỗ trợ SSL (HTTPS) không?
4. [ ] Backup dữ liệu (nếu có backend) đã được thiết lập chưa?
