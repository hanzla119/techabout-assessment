# PKHosting Shared Hosting + Domain Checkout Fix (Proof Prototype)

An interactive standalone prototype fixing the "ghost domain" checkout trap on PKHosting.
It introduces a 3-way domain selector (Register New, Transfer, Own Domain), transparent live pricing breakdown (including the 3-Year Free .com credit), zero-overflow mobile layout, and interactive field validation.

## How to Run
- Direct: Open `proof/index.html` in any modern web browser (Chrome, Firefox, Safari, Edge).
- CLI HTTP Server: Run `python3 -m http.server 3000 --directory proof` then visit `http://localhost:3000`.
- Zero build steps, external network requests, or npm dependencies required.
