# Product Audit: PKHosting Shared Hosting & Checkout Funnel

## 1. Product Choice and Rationale: Why Narrow Beats Broad
Out of TechAbout's four products, **PKHosting.com** was selected—specifically auditing the flagship **Personal Shared Hosting Page (`/hosting/personal/`) and its "Order Now" Checkout Flow (`/checkout/?product=2`)**.

Narrow beats broad because this single conversion funnel represents PKHosting’s core acquisition engine for Pakistani small businesses, freelancers, and students. A broad, surface-level audit across four domains would only yield cosmetic observations. Auditing this specific path end-to-end—simulating real purchases on both desktop (1280×900) and mobile (375×812 iPhone viewport) using Chrome DevTools, network timing analysis, and client bundle deobfuscation—uncovered systemic breakdowns where real money and trust leak out of the business daily.

---

## 2. Findings Table (Observed 2026-10-04, Pakistan Standard Time)

| # | What Was Seen | URL | Time Seen (PKT) | Verifiable Evidence | Business Cost | Severity | Effort |
|---|---|---|---|---|---|---|---|
| **1** | **"Ghost Domain" Trap:** Shared hosting checkout provides no domain action selector (Register vs Transfer vs Own), silently defaulting to "owndomain". | `https://www.pkhosting.com/checkout/?product=2` | 2026-10-04 19:48:52 | `checkout-page.js` line 56490: selector is gated by `np ? (...) : null` where `np = tQ.type === "domain"`. POST to `/api/checkout/cart/` with domain returns `"domain":{"action":"owndomain"}`. | High churn, 100% refund/chargeback rate when customers realize their desired domain was never registered. Domain squatting vulnerability. | **P0 (Critical)** | Medium (2–3 days) |
| **2** | **Silent Exclusion of `.pk` TLDs:** Advertised "Free Domain with 3-Year Plan" silently excludes `.pk` without explanation, while displaying a confusing generic .com disclaimer. | `https://www.pkhosting.com/checkout/?product=2` | 2026-10-04 20:11:38 | `checkout-page.js`: `function U(e){ return t.endsWith(".com") && 2===t.split(".").length; }`. Screenshot `checkout_3y_pk_domain.png` shows Rs 16,077 total with Rs 0 domain discount. | Alienates domestic Pakistani businesses wanting `.pk` domains; drives them to local competitors (Nexus, Webx, PakNIC). | **P1 (High)** | Low (1 day) |
| **3** | **Mobile Horizontal Overflow:** Viewport breaks horizontally at 375px width, generating an unsightly horizontal scrollbar and clipping form borders. | `https://www.pkhosting.com/checkout/?product=2` | 2026-10-04 19:58:38 | Screenshot `before_mobile.png`. Headless Chrome measurement: country code flex wrapper (`w-[7.5rem]`) and unconstrained grid items force body scroll beyond viewport. | Drop-off in mobile checkout conversions (where mobile represents >75% of web traffic in Pakistan). | **P1 (High)** | Very Low (0.5 day) |
| **4** | **Sticky CTA & Live Chat Collision:** The mobile floating Crisp/chat bubble directly occludes and touches the sticky bottom "Place order" button. | `https://www.pkhosting.com/checkout/?product=2` | 2026-10-04 19:59:05 | Screenshot `before_mobile.png`. Floating button (`bottom: 1.25rem`) directly collides with bottom container on 375px viewport. | Accidental chat opens instead of order completions ("rage clicks") right at the final conversion point. | **P1 (High)** | Very Low (1 hr) |
| **5** | **Marketing CTAs Strip Billing Period:** Annual plan cards advertise "Rs 6,990/yr", but clicking "Order Now" links to `/checkout/?product=2` without `&period=1Y`. | `https://www.pkhosting.com/` | 2026-10-04 19:39:06 | Landing page HTML: `<a href="/checkout/?product=2">Order Now</a>`. Checkout initializes with `initialPeriod: "1M"` (Rs 699/mo). | 90% reduction in upfront cash collection (Rs 699 vs Rs 6,990) and suppressed customer lifetime value (LTV). | **P2 (Medium)** | Very Low (15 mins) |
| **6** | **Dead / Disabled Submit Button:** Primary CTA remains disabled (`disabled:cursor-not-allowed`) without active inline field feedback. | `https://www.pkhosting.com/checkout/?product=2` | 2026-10-04 20:09:22 | DevTools inspection: `btns[0].disabled === true`. Condition `let n8 = eE || eI || !nQ || !nd || !nM ...` provides zero scroll-to-focus on click. | Abandonment due to perceived checkout freeze when minor validation rules fail silently. | **P1 (High)** | Low (1 day) |
| **7** | **Promo Code Hard-Lock:** Entering any invalid or non-applicable coupon permanently disables the checkout button until manually cleared. | `https://www.pkhosting.com/checkout/?product=2` | 2026-10-04 19:48:49 | `checkout-page.js`: `nz = !!eD && null != ej && (_(ej.discount) ?? 0) === 0`, causing `n8` to disable submission. | Checkout abandonment caused by promotional traps. Shoppers testing codes get blocked. | **P2 (Medium)** | Very Low (2 hrs) |
| **8** | **Inverted Step Prerequisite:** Step 1 requires domain to calculate total, but Domain input is hidden down in Step 3 beneath Account creation. | `https://www.pkhosting.com/checkout/?product=2` | 2026-10-04 19:45:19 | Step 1 badge: `<p>Enter domain to prepare order</p>`. Step 2 forces account credentials before user knows domain status or price. | Cognitive friction and drop-off before account creation. Violates standard e-commerce configuration hierarchy. | **P2 (Medium)** | Medium (2 days) |
| **9** | **Missing Domain Availability Check:** Checkout allows entering taken domains (`google.com`) without a WHOIS query. | `https://www.pkhosting.com/checkout/?product=2` | 2026-10-04 19:48:01 | `checkout-page.js` regex `z(e)` only tests syntax `/^[a-z0-9][a-z0-9-]*(\.[a-z0-9][a-z0-9-]*)+$/i`. Zero registry check. | Manual support workload to resolve orders for unavailable domains; payment reconciliation costs. | **P2 (Medium)** | Medium (2 days) |
| **10** | **Unsorted 240+ Country Picker:** Mobile fast-checkout contains an unindexed native select with 240+ countries. | `https://www.pkhosting.com/checkout/?product=2` | 2026-10-04 19:41:38 | `<select aria-label="Country dialling code">` contains 240+ raw `<option>` elements from Afghanistan to Zimbabwe. | Form fatigue and accidental country changes on mobile touchscreens. | **P3 (Low)** | Very Low (1 hr) |

---

## 3. Deep Dive: The Critical Money & Trust Leaks

### A. The "Ghost Domain" Trap (Finding #1)
The single most damaging defect is the complete absence of a domain configuration selector when ordering hosting. 

When a customer orders standalone domain registration (`/checkout/?product=1`), the UI renders clean buttons: `Register New` and `Transfer In`. However, in `checkout-page.js`, this component is wrapped with:
```javascript
np = tQ?.type === "domain"
{np ? ( <DomainOrderTypeSelector /> ) : null}
```
Because Personal Hosting has `tQ.type === "hosting"`, `np` evaluates to `false`. The entire domain selector is removed from the DOM, leaving only a bare text input: `Domain to host`.

When the user types `mybrandname.com`, the client POSTs to `/api/checkout/cart/` with no domain action specified. The backend defaults to:
```json
"domain": {
  "action": "owndomain",
  "owndomain_sld": "mybrandname",
  "owndomain_tld": ".com"
}
```
The cart line item reads: `Personal Hosting for mybrandname.com Rs 6,990`.
**The customer believes they have registered their brand and purchased hosting.** In reality, PKHosting has only created a cPanel vhost for an external domain. The domain remains completely unregistered on the open market.

**The Fallout:**
1. **Financial:** 100% refund rate or payment dispute when the user discovers their site does not resolve.
2. **Reputational:** If a third party or competitor registers the domain in the interim, PKHosting incurs permanent brand damage and potential legal liability.
3. **Lost Revenue:** PKHosting leaves legitimate domain registration revenue (Rs 3,999/yr for `.com` or Rs 4,699 for `.pk`) on the table by failing to offer registration during checkout.
