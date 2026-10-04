# TechAbout Assessment Submission: PKHosting Funnel Optimization

**Candidate Assessment:** General Role — Pick One Product, Find One Real Problem, Prove the Fix  
**Selected Product & Flow:** PKHosting.com Shared Hosting & Checkout Funnel (`/hosting/personal/` → `/checkout/?product=2`)  
**Core Problem Solved:** The "Ghost Domain" Trap — Missing Domain Registration in Shared Hosting Checkout

---

## Deliverables Summary

| File | Purpose | Brief Compliance |
|---|---|---|
| [`audit.md`](./audit.md) | 2-page detailed audit, choice rationale, and 10-finding evidence table | Exact name, 2 pages, verified timestamps & DevTools evidence |
| [`proposal.md`](./proposal.md) | 1-page proposal: 1-sentence problem, proposed fix, why first, metrics, falsification | Exact name, 1 page, impact vs effort, falsification criteria |
| [`proof/`](./proof/) | Standalone working prototype, screenshots, and under-10-line README | Exact name, standalone interactive HTML, runs with 1 command |
| [`notes.md`](./notes.md) | 10-line reflection: next 5 days, what couldn't be verified, timing, AI disclosure | Exact name, exactly 10 lines |
| `README.md` | Setup, decisions, tradeoffs, tests, and future improvements | Explains engineering decisions, tests, tradeoffs |

---

## How to Run the Standalone Proof

The proof is implemented as a production-grade, zero-dependency standalone web application in `proof/index.html`.

### Option 1: Direct File Open
Double click or open `proof/index.html` in any modern web browser (Google Chrome, Mozilla Firefox, Safari, Microsoft Edge).

### Option 2: Local HTTP Server
Run from the root of the project:
```bash
python3 -m http.server 3000 --directory proof
```
Then navigate to: `http://localhost:3000`

---

## Key Decisions

1. **Focusing on the Shared Hosting Order Funnel:** Rather than surveying four distinct sites broadly, we honed in on PKHosting's primary customer acquisition engine. Shared hosting is where first-time Pakistani entrepreneurs make purchase decisions. Any leak here directly impacts cash flow and reputation.
2. **Prioritizing Finding #1 ("The Ghost Domain Trap"):** Customers ordering hosting expect to register a domain. Currently, PKHosting silently assigns `action: "owndomain"` for all 1-year and monthly orders. When customers discover their site does not resolve because the domain was never registered, they request immediate refunds. Fixing this yields immediate ROI (higher AOV + lower churn).
3. **Stand-Alone Build Track Implementation:** We built an authentic, high-fidelity HTML/CSS/JS prototype with zero external bundle dependencies. It demonstrates:
   - 3-Way Domain Action Selector (`Register New`, `Transfer In`, `Use Existing`).
   - Dynamic Term Pricing with 10x annual discount and 3-Year Free .com credit.
   - Elimination of mobile horizontal scroll overflow.
   - Clearance of floating chat widgets from the primary checkout CTA.
   - Click-to-focus interactive validation (abandoning the dead disabled button anti-pattern).

---

## Tradeoffs

- **Single-File Vanilla vs Framework:** We chose a single-file, zero-dependency solution over a full Next.js/React project to ensure the reviewer can evaluate and run it instantly on any OS without `node_modules` installation, version conflicts, or build tool friction.
- **Client-Side WHOIS Simulation vs Live API:** Because live domain availability queries require authenticated registrar API keys (or third-party RDAP rate limits), the prototype includes a realistic debounced client-side simulator demonstrating user feedback states without risking third-party network outages during evaluation.

---

## Testing & Verification

The prototype was rigorously verified using automated headless Chrome (Puppeteer) test suites:
- **Mobile Viewport Test (375×812 iPhone):** Verified `document.documentElement.scrollWidth === window.innerWidth` (0 horizontal scroll overflow).
- **Console Inspection:** 0 JavaScript errors or unhandled rejections during full checkout lifecycle.
- **Form State Validation:** Verified that entering invalid email or missing domain triggers smooth scroll-to-focus, field highlighting, and button shake without blocking the user permanently.
- **Order Preparation:** Verified that selecting a 3-Year plan with `.com` automatically reflects the Rs 3,999 promotional credit, while selecting `.pk` provides an informative advisory.

---

## What 5 More Days Would Improve

1. **Direct Next.js Integration:** Submit a clean Pull Request to PKHosting's repository updating the existing `app/(frontend)/checkout/page.tsx` component and unifying it with `app/(frontend)/checkout/domain/`.
2. **A/B Testing Infrastructure:** Deploy a split-test experiment via Cloudflare Workers to measure conversion rate uplift against the legacy control flow.
3. **Real-Time RDAP Domain Availability:** Hook the domain input up to an asynchronous PKNIC / ICANN RDAP microservice with instant suggestion chips for alternate available TLDs.
