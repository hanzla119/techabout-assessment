# Proposal: Fixing PKHosting's Shared Hosting & Domain Checkout Flow

## 1. The Problem in One Sentence
PKHosting's shared hosting checkout omits a domain action selector, silently treating all entered domains as externally owned ("owndomain") rather than offering registration or transfer—causing customers who believe they bought a domain to lose their names to squatters, fail onboarding, and demand immediate refunds.

---

## 2. The Proposed Change
Implement a unified, transparent **3-Way Domain Configuration Step** directly integrated into the shared hosting checkout flow:
1. **Register New Domain:** Includes real-time availability check, extension selector (.com, .pk, .com.pk), transparent line-item pricing (e.g. Rs 3,999/yr), and automated 100% discount credit when an eligible 3-year term is selected.
2. **Transfer Domain:** Prompts for EPP/auth code and outlines registrar unlock prerequisites.
3. **Use Existing Domain:** Explicitly clarifies that PKHosting will **not** register or bill for the domain (Rs 0 line item), providing clear nameserver instructions (`ns1.pkhosting.com` / `ns2.pkhosting.com`).
4. **Flow & Mobile Fixes:** Re-order steps into logical order (*1. Plan & Domain → 2. Account → 3. Payment*), eliminate mobile horizontal overflow, elevate the live chat bubble above the sticky bottom CTA, and replace dead disabled buttons with active click-to-focus inline validation.

---

## 3. Why Fix This First (Impact vs. Effort)
- **High Business Impact:** This is not a cosmetic layout bug; it is an active financial and trust leak at the exact point of transaction. It simultaneously:
  - **Eliminates Post-Purchase Churn:** Stops 100% of chargebacks and refund tickets caused by unfulfilled domain expectations.
  - **Drives Higher Average Order Value (AOV):** Automatically bundles high-margin domain registrations with 1-year and 2-year hosting orders.
  - **Restores Advertising Integrity:** Seamlessly honors the homepage promise of "Free .com with a 3-year term" without obscure technical workarounds.
- **Low Implementation Effort (2 to 3 days):** The backend `/api/checkout/cart/` already supports domain registration items (`productId: 1`, `allow_domain_register: true`). The fix requires only frontend state and component adjustments, reusing the existing domain selector logic already present in `?product=1`.

---

## 4. Measurement Thinking

### The Metric to Move & Rough Baseline
- **Primary Metric:** **First-Month Hosting Churn / Refund Rate** (due to onboarding domain setup failure).
  - *Rough Baseline:* Estimated at **12% to 18%** of new shared hosting signups (based on industry benchmarks for hosting providers with disconnected domain provisioning).
- **Secondary Metric:** **Average Order Value (AOV)** for new 1-year shared hosting customers.
  - *Rough Baseline:* Currently locked at **Rs 6,990** (because no domain registration add-on can be purchased during checkout).

### Success Threshold
- **Refund/Dispute Reduction:** A **≥70% drop** in domain-related refund requests and onboarding support tickets within 30 days of release.
- **AOV Increase:** A **≥35% increase in 1-Year Hosting AOV** (from Rs 6,990 to >Rs 9,500) as 40%+ of buyers bundle domain registration at checkout.
- **Funnel Completion:** A **≥15% relative improvement** in mobile checkout completion rate resulting from the removal of layout overflow and chat CTA obstruction.

### What Would Prove Me Wrong (Falsification Criteria)
- If A/B test data shows that >80% of PKHosting's hosting buyers strictly bring domains from external registrars, and introducing domain registration options increases checkout abandonment by >5% (due to added decision steps), the hypothesis is falsified.
- If refund tickets remain unchanged, proving that customer confusion stems from cPanel provisioning rather than unfulfilled domain expectations.

---

## 5. What We Need From TechAbout
1. Access to Google Analytics 4 checkout funnel drop-off metrics (`begin_checkout` vs `purchase`).
2. Current 90-day support ticket categorization tags regarding "domain not working" or "where is my domain".
3. Staging deployment verification access to test the updated Next.js checkout bundle against the live billing backend.
