# Google AdSense Mastery & Monetization (S4 Edition)

## Skill Identity
**Name:** Google AdSense Specialist (S4)
**Version:** 1.0.0
**Description:** Expert-level integration and optimization of Google AdSense. Focuses on approval speed, high CPM placement, and policy compliance.
**Trigger Requirements:**
- "gắn quảng cáo" (add ads)
- "google adsense"
- "kiếm tiền website" (make money from website)
- "tối ưu hóa quảng cáo" (ad optimization)
- "adsense approval"

## 💡 The S4 Philosophy (AdSense Edition)
> **"Content is the Bait, Ads are the Harvest."**

Getting AdSense is easy, but getting **high revenue** without ruining user experience is the art. 
The goal is to maximize **RPM (Revenue per Thousand impressions)** while keeping the site **Safe & Fast**.

---

## 1. The "Fast Approval" Blueprint
*How to get approved in the first try.*

### ✅ Step 1: Quality Over Quantity
- **Must have:** At least 15-20 high-quality, original articles (800+ words).
- **Avoid:** "Thin content" or AI-generated fluff without manual editing.
- **Critical Pages:** About Us, Contact, Privacy Policy, Terms of Service (Must exist!).

### ✅ Step 2: Clean UI/UX
- Use a fast, mobile-responsive theme.
- No broken links.
- Clear navigation menu.

---

## 2. High-Leverage Placement Strategy
*Where to put ads for maximum clicks (CTR).*

### 💰 The "Golden Zones"
1.  **Above the Fold:** Large rectangle (336x280) or leaderboard (728x90) right under the header.
2.  **Inside Content:** Every 3-4 paragraphs (In-feed/In-article ads).
3.  **Below Post:** The "Engagement Zone" when users finish reading.

### 🛡️ Smart Ad Types
- **Auto Ads:** Enable them but use **Ad Load control** (don't let Google over-clutter your site).
- **Multiplex Ads:** Great for the sidebar or footer to look like "Recommended Content".
- **Anchor Ads:** Highly effective on mobile (stick to the bottom).

---

## 3. The "Anti-Ban" Shield (Compliance)
*Never lose your account.*

- ❌ **NEVER** click your own ads (even to "test").
- ❌ **NEVER** ask friends to click.
- ❌ **NEVER** use bots or "traffic exchanges".
- 🛡️ **Ads.txt:** Always implement `ads.txt` correctly to avoid "Revenue at risk" warnings.
- 🛡️ **Invalid Traffic:** Use plugins like "AdSense Invalid Click Protector" (AICP) for WordPress if needed.

---

## 4. Technical Implementation (The Expert Way)

### 🚀 Optimize Speed
- Load AdSense scripts with `async`.
- Don't load 10+ ads on a single page; it kills your SEO rankings.

### 🛠️ Example Integration (React/Next.js)
```javascript
// Components/AdBanner.js
import { useEffect } from 'react';

const AdBanner = ({ slot }) => {
  useEffect(() => {
    try {
      (window.adsbygoogle = window.adsbygoogle || []).push({});
    } catch (e) {
      console.error(e);
    }
  }, []);

  return (
    <ins className="adsbygoogle"
         style={{ display: 'block' }}
         data-ad-client="ca-pub-XXXXXXXXXXXXXXXX"
         data-ad-slot={slot}
         data-ad-format="auto"
         data-full-width-responsive="true"></ins>
  );
};
```

---

## 5. The "Kill List" (Common Mistakes)
- ❌ **Ads on Login/Error pages:** Policy violation.
- ❌ **Ads near Images:** Trying to "trick" users into clicking an image but hitting the ad instead.
- ❌ **Overlapping Content:** Ads that cover the text or buttons.

---

## Summary Checklist
Before applying/deploying:
1.  [ ] Is **Privacy Policy** and **Contact** page live?
2.  [ ] Is **Ads.txt** uploaded to the root directory?
3.  [ ] Are ads placed in the **Golden Zones**?
4.  [ ] Did you check mobile responsiveness with ads enabled?
5.  [ ] Is the content 100% compliant with Google's policies?

**Next Step:** Monitor **CTR** and **RPM** in the AdSense Dashboard after 14 days and tweak placements.
