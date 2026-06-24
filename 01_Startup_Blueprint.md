# Startup Blueprint — Limitless ChargeHub
### A Smart, Multi-Source Power Outlet Charging Station for Africa
**Limitless Power Solution Concepts** · CAC BN 8752639 · TIN 33455754-0001
Document owner: Project Lead / CTO · Version 1.0 · June 2026

---

## 0. Executive summary

**What we are building.** *ChargeHub* — a secure, solar-first **smart charging station** where people pay (online or on-site) to charge phones, laptops, tablets and other low-power devices through **metered, individually-controlled smart outlets**, backed by an **online booking + payment + monitoring platform** and an **automatic three-source power supply** (solar PV + battery, national grid, generator) that always picks the cheapest available clean energy.

**Why now.** Nigeria runs on unreliable, expensive power. Students, commuters and small businesses constantly need to charge devices and have nowhere safe, cheap and available to do it. The team has *already built and won awards with the core product* and *already runs a CAC-registered energy company* — the gap is commercialisation, not invention.

**The unfair advantage.** Three de-risking assets already exist in-house:
1. A working, award-winning **smart-outlet + web platform** prototype (NSE Babafunso GEOTY 2020, 3rd; *SSRG IJEEE* 2021 publication).
2. A working **IoT hybrid power transition controller (HPTS)** — auto grid↔generator switching in 0.15 s, built by two of the founders.
3. A **registered company** with a business plan, brand and grant-readiness groundwork.

**The ask of this document.** A realistic 6-month path to a **revenue-generating pilot at the University of Ibadan**, structured so the same evidence package unlocks grant funding (ZE-Gen-style climate/energy-access programs), an accelerator place, and a first angel/convertible round.

**Headline targets (6 months):**
- 1 pilot station live (8–12 smart outlets, solar+grid+generator) at UI.
- ≥ 150 unique paying users; ≥ 65% repeat usage rate.
- ≥ 80% station uptime; every kWh metered and billed (zero "energy theft").
- 1 grant/competition submitted; 1 host-partnership MoU signed; data-room complete.

---

## 1. Vision & Mission

**Vision.** *Uninterrupted power, limitless possibilities* — to become West Africa's default network of clean, intelligent, pay-as-you-go energy-access points, starting with device charging and expanding into full distributed energy management.

**Mission.** To give people and businesses **safe, affordable, clean and always-available power on demand**, by combining renewable-first hybrid energy with IoT metering and a simple digital experience — turning every outlet into a measured, managed, monetisable service.

**Operating principles (carried over from the company's own docs):**
- *Load-first engineering* — measure real demand before sizing anything.
- *Hybrid by default* — no single point of failure (solar + storage + grid + generator).
- *Measurable impact* — every system reports kWh generated, diesel displaced, CO₂ avoided.
- *Local capacity* — Nigerian-built, Nigerian-maintained.

---

## 2. Product definition

### 2.1 What the customer gets
A **ChargeHub station**: a secure kiosk/cabinet (wall- or pole-mounted, or a small booth) with **8–12 individually-metered smart sockets** (Nigerian/UK 13 A type-G + USB-A/USB-C fast-charge ports), an on-unit **keypad + display**, and a **cloud-connected controller**. Users:

1. **Find** a station (map on web/app).
2. **See** live outlet availability and status.
3. **Book** a time slot and power allowance, or walk up.
4. **Pay** online (card / bank transfer / USSD / mobile money) — or with cash via an attendant during the pilot.
5. **Receive** a one-time access code (PIN) by SMS/email.
6. **Use** the outlet — enter the PIN on the station keypad to unlock the assigned socket for the paid duration and power cap.
7. **Get notified** before time/energy expires; can top up.

> This is exactly the flow the team already prototyped: the PIN encodes *access code + duration + socket ID + power threshold* (e.g. `14785|02|2|03`), the Arduino validates it against codes pulled from the web platform, energises the matching relay, meters consumption with an ACS712 current sensor, and cuts power on time-up, over-power, or 10-minute idle. We are productising a proven design, not prototyping a new one.

### 2.2 What the operator/admin gets
A management console to: provision stations & outlets, monitor live usage and outlet status, watch **energy by source** (solar/grid/gen) and battery state of charge, track **revenue**, manage users and pricing plans, and run a **maintenance dashboard** with fault alerts.

### 2.3 Product tiers / pricing plans
Re-use the prototype's tiered model (already built — see `assets/web_dashboard.jpg`), renamed for a public audience:

| Plan | Access time | Power cap | Typical use | Indicative price* |
|------|-------------|-----------|-------------|-------------------|
| **Quick** | 1 hr | low (≤15 W) | phone top-up | ₦150–250 |
| **Standard** | 3 hr | medium | phone + tablet | ₦400–600 |
| **Pro** | 5 hr | high (≤20 kW cap setting) | laptop work session | ₦700–1,000 |
| **Day** | 7 hr+ | high | vendor / all-day | ₦1,200–1,800 |

\*Indicative only — **set by the willingness-to-pay survey in doc 04**, not by guesswork. The team's 2021 paper modelled a sustainable tariff of ~3.5 ¢/kWh for Ibadan; per-session pricing above bundles convenience + security, not just energy.

### 2.4 What makes it different
- **Metered & secure by outlet** — solves "energy theft" and over-draw; every kWh is billed.
- **Renewable-first hybrid** — cheaper and cleaner than diesel-only kiosks; keeps running through grid outages.
- **Online booking + payment** — not just a USB bar; a managed service with accounts, history, notifications.
- **Handles real devices** — laptops and equipment, not only phones.

---

## 3. Technical architecture (summary — full detail in doc 02)

```
        ┌─────────────────────── ENERGY LAYER (HPTS) ───────────────────────┐
 Solar PV → Charge controller → Battery bank ┐
 National grid ───────────────────────────────┤→ Auto source-select → Inverter → AC bus
 Generator (auto start/stop) ─────────────────┘     (ESP32 + PZEM-004T sensors,
                                                      relays, priority logic)
        └────────────────────────────────────────────────────────────────────┘
                                   │ metered AC
        ┌─────────────────── OUTLET LAYER (Smart Sockets) ───────────────────┐
        │  Controller (Arduino Mega/ESP32) · keypad · LCD · SD log           │
        │  per-socket relay + ACS712 current sensor (energy metering)        │
        │  8–12 sockets (13A type-G) + USB-A/USB-C PD ports                  │
        └────────────────────────────────────────────────────────────────────┘
                                   │ Wi-Fi / Ethernet · MQTT/HTTPS
        ┌──────────────────────── CLOUD PLATFORM ────────────────────────────┐
        │  API + DB · booking engine · payment gateway (Paystack/Flutterwave) │
        │  PIN/access-code service · notifications (SMS/email) · OTA updates   │
        └────────────────────────────────────────────────────────────────────┘
                                   │
        ┌──────── USER WEB/APP ────────┐   ┌──────── ADMIN CONSOLE ───────────┐
        │ find · book · pay · monitor  │   │ stations · energy · revenue · M&E │
        └──────────────────────────────┘   └──────────────────────────────────┘
```

**Energy source-selection logic (priority + optimisation):**
`Solar/battery (if SoC healthy & sun) → Grid (if present & cheaper than gen) → Generator (last resort)`, with rules for source availability, marginal cost (₦/kWh), battery state of charge, live demand, and a "maximise renewable share" objective. This is the HPTS work, extended from binary grid↔gen switching into a true **energy-management controller**.

---

## 4. Business model

### 4.1 How we make money (pilot → scale)
1. **Per-session charging revenue** (core, day 1) — users pay per booking/plan.
2. **Operator/host revenue share** — for hosted stations on third-party land (campuses, malls, parks), split gross revenue (e.g. host 10–20%).
3. **Station-as-a-Service (CapEx-light expansion)** — sell/lease stations to hosts who want their own branded charging point, plus a SaaS management fee.
4. **Software/SaaS** — the booking + energy-management platform licensed to other micro-utility/EV-adjacent operators (later).
5. **Advertising / sponsorship** — screen + station branding (telcos, banks, FMCG) at high-traffic sites.
6. **Data & impact products** — anonymised energy-access + footfall analytics; carbon-credit aggregation once M&E is in place.

**Day-1 model for the pilot:** company owns + operates 1 station; revenue = per-session fees; host (UI) provides space + security in exchange for a small revenue share and student-impact reporting.

### 4.2 Unit economics (illustrative, pilot station — *assumptions stated*)
Assumptions: 10 outlets; ~6 paid socket-hours/outlet/day; blended ₦120/socket-hour net of payment fees; 26 operating days/month.

- Monthly gross ≈ 10 × 6 × ₦120 × 26 ≈ **₦187,000/month** at modest utilisation.
- At higher utilisation (10 socket-hrs/outlet/day, ₦150 blended) ≈ **₦390,000/month**.
- Variable cost is low (the energy is largely solar; grid/gen only as backup). Main OpEx: airtime/data SIM, payment fees (~1.5%), part-time attendant, maintenance reserve.
- **These are planning figures.** The market-validation survey (doc 04) sets real price points and the demand model converts them to a defensible projection for the financial model and investor deck.

### 4.3 Cost structure
- **Build (CapEx) per pilot station** — see §7 cost analysis.
- **OpEx** — connectivity, payment fees, attendant stipend, maintenance/parts reserve, cloud hosting, fuel (generator, minimised by solar-first logic).
- **Shared/back-office** — already lean (founders), per the company's existing business-structure doc.

---

## 5. Market opportunity

### 5.1 The problem (quantified)
- Nigeria has one of the world's largest populations with unreliable grid power; firms lose **$1,304–$9,402 per kW annually** to outages (team's 2021 paper, citing McLinn/Oseni). Power instability is a "silent tax" on every business and student.
- Smartphone and laptop penetration keeps rising while **public, secure, metered charging is essentially absent**. Existing solar charging kiosks are mostly **USB/phone-only, unmetered, and prone to energy theft** — the exact gaps this product closes (per the literature review in the team's paper).

### 5.2 Who we serve first (beachhead)
**University of Ibadan students & staff** (~30,000+ students). High device dependence, frequent campus power gaps, dense foot traffic, captive recurring demand, secure environment. (See doc 03 / §10 for the full siting justification.)

### 5.3 Expansion segments (adjacent, proven demand)
Innovation/tech hubs & co-working spaces · transport parks & bus terminals · markets & shopping complexes · gated estates · hospitals/clinic waiting areas · event venues · filling stations. Each shares: high footfall + unreliable power + people waiting with devices.

### 5.4 Market sizing approach (to be filled by validation)
Bottom-up: (target sites in SW Nigeria) × (outlets/site) × (sessions/outlet/day) × (price). We deliberately size *after* the survey rather than quoting a top-down TAM we can't defend — funders reward this (per the company's own grant-readiness notes: "specificity over ambition").

---

## 6. Revenue streams (prioritised)

| # | Stream | When | Margin | Notes |
|---|--------|------|--------|-------|
| 1 | Per-session charging | Pilot (M0) | High | Core; validated by survey |
| 2 | Host revenue-share stations | M6–12 | High | Campus/mall expansion, low CapEx for us |
| 3 | Station sale + SaaS | M9–18 | Med/High | Sell hardware + recurring platform fee |
| 4 | Platform SaaS licensing | M18+ | Very high | License booking/EMS to other operators |
| 5 | Advertising/sponsorship | M6+ | High | Telco/bank branding at busy sites |
| 6 | Carbon credits / impact data | M18+ | High | Needs M&E + verification (groundwork in §13) |

---

## 7. Cost analysis (pilot station bill-of-materials, re-priced 2026)

Derived from the team's 2021 paper (Table III) plus current Nigerian market reality. **FX assumption: ₦1,580/$.** Treat as a planning budget to be confirmed by quotes.

| Item | Spec / basis | Est. cost (₦) |
|------|--------------|---------------|
| Solar PV array | ~2–3 kWp (sized to Ibadan profile in paper) | 1,400,000 |
| Battery bank | LiFePO₄ ~5 kWh (modern swap for the paper's lead-acid) | 1,900,000 |
| Hybrid inverter + charge controller (MPPT) | 3–5 kVA | 950,000 |
| Smart-outlet cabinet (controller, relays, ACS712 sensors, keypad, LCD, SD, comms) | 10 outlets, per prototype BOM | 450,000 |
| HPTS source-switching module | ESP32 + PZEM-004T ×2, contactors/ATS, enclosure | 350,000 |
| Sockets, USB-PD modules, wiring, protection (breakers, RCD, SPD), earthing | safety-critical | 300,000 |
| Enclosure / kiosk / mounting / signage | weatherproof, lockable | 500,000 |
| Connectivity (router/SIM), cloud setup (year 1) | 4G + hosting | 200,000 |
| Installation, certification, contingency (15%) | labour + buffer | 950,000 |
| **Total estimated pilot CapEx** | | **≈ ₦7.0M (~$4,400)** |

**6-month operating budget (lean):** connectivity ₦15k/mo, payment fees ~1.5% of revenue, attendant stipend ₦40k/mo, maintenance reserve ₦25k/mo, software/cloud ₦20k/mo, misc ₦20k/mo → **≈ ₦120k/month** ex-revenue. Funded by founder capital + grant/competition + early revenue.

> A leaner "MVP-first" path (§14) can cut first-spend to ~₦2.5–3.5M by starting **grid+generator+small battery** and adding solar in month 4 once demand is proven — recommended to de-risk capital.

---

## 8. Risk analysis

| Risk | L | I | Mitigation |
|------|---|---|------------|
| **No real demand / wrong price** | M | H | Run doc-04 validation *before* full build; MVP-first; pre-sell day passes |
| Vandalism / theft of hardware | M | H | Secure host site (UI), lockable enclosure, attendant in pilot, asset tagging, insurance |
| Payment integration friction (failed transfers) | M | M | Use Paystack/Flutterwave (proven in NG); offer USSD + cash fallback in pilot |
| Solar underperformance / battery degradation | L | M | Conservative sizing, MPPT, LiFePO₄, monitor SoC, grid/gen backup |
| Generator/grid switching fault (safety) | L | H | HPTS proven design; certified install; RCD/SPD/earthing; phase-isolation; pro electrician sign-off |
| Connectivity loss → can't validate PINs | M | M | Offline PIN cache on SD (already in prototype); store-and-forward sync |
| Key-person dependency (4-person team) | M | H | Documentation, RACI (doc 06), cross-training, version control, IP assignment |
| FX / component cost inflation | H | M | Local sourcing where possible, buy critical imports early, 15% contingency |
| Regulatory ambiguity | L | M | Stay under captive/embedded threshold; counsel review (doc 08) |
| Funding delay | M | H | MVP-first keeps burn low; competitions + revenue bridge to grant/angel |

---

## 9. Competitor analysis

| Category | Examples | Their gap | Our edge |
|----------|----------|-----------|----------|
| USB solar charging kiosks | Generic solar phone-charging tables/carts | Phone-only, unmetered, no booking, theft-prone | Metered per-outlet, laptops too, online booking + payment, secure access codes |
| Diesel/grid "charging corners" | Informal vendors charging phones for a fee | Dirty, unreliable, cash-only, no accountability | Clean solar-first, digital payments, audited energy |
| Co-working / café power | Free outlets at cafés | Tied to a purchase, scarce, not a service | Standalone, always-on, pay-for-what-you-use |
| EV charging players | Emerging NG EV charging startups | Targeting cars, capital-heavy, thin near-term market | Device charging = today's mass market; same platform extends to EV later |
| Smart-socket OEMs | Imported Wi-Fi smart plugs | Consumer home use, no public-billing/booking layer | Public-access billing, energy management, multi-source supply |

**Positioning statement:** *The only renewable-first, fully-metered public charging network with online booking and payment, built for African power realities — by a team that already built and published the technology.*

---

## 10. Pilot deployment plan (University of Ibadan)

**Why UI (justification):**
- **Secure & access-controlled** campus → low vandalism risk, reliable attendant presence.
- **Consistent, dense demand** → 30,000+ device-dependent students/staff; frequent power gaps.
- **Team home-ground** → founders are UI Electrical/Electronic Engineering alumni; existing faculty relationships ease siting, support and credibility (the 2021 paper is UI-affiliated).
- **Documented solar resource** → ~5 kWh/m²/day GHI for Ibadan (team's own paper) → solar-first economics work.
- **Scalable template** → succeed at one faculty/hall, replicate across halls of residence, library, faculties, then to other campuses.

**Pilot structure:**
- **Site:** high-traffic indoor/sheltered spot — e.g. a hall-of-residence common area, the Kenneth Dike Library forecourt, or a faculty café. Secure power closet for battery/inverter.
- **Footprint:** 1 station, 10 smart outlets + USB-PD, solar+battery+grid+generator backup.
- **Partnership:** MoU with the hall/faculty or UI's innovation/works unit — space + security in return for revenue share + a student-impact report.
- **Phases:** see roadmap §17.
- **Success metrics:** uptime ≥80%, ≥150 paying users, ≥65% repeat rate, every kWh billed, NPS ≥ +30, payback trajectory consistent with the financial model.

---

## 11. Scaling strategy

1. **Replicate within UI** (months 6–12): 3–5 stations across halls/library/faculties — proves a multi-station network + central console.
2. **Multi-campus** (12–24): other SW Nigerian universities (OAU, UNILAG, LAUTECH, FUTA) via the same host-revenue-share template.
3. **Off-campus high-footfall** (18–30): transport parks, malls, markets, estates — Station-as-a-Service to hosts.
4. **Platform play** (24+): license the booking + energy-management software to third-party operators; add EV-charging SKUs on the same stack.
5. **Geographic** (30+): pan-African pilots (aligned with the company's existing "Expand" phase), DFI debt + carbon finance.

**Scaling enablers:** standardised station BOM, OTA firmware, modular HPTS, multi-tenant cloud, trained installer/maintenance crew, host-partner playbook.

---

## 12. Investment readiness strategy

**Narrative:** "Award-winning, *de-risked* hardware + registered company + clear beachhead → fund the network, not the invention."

**Raise plan (sequenced, mostly non-dilutive first):**
1. **Now–M3:** competitions + small grants + founder capital → build MVP pilot (target ₦3–7M).
2. **M3–M6:** accelerator (climate/energy-access) → cash + mentorship + network.
3. **M6–M12:** angel / convertible note (₦20–60M) → 3–5 station network + first hires, *triggered by pilot evidence*.
4. **M18+:** blended/project finance + climate grant for multi-site rollout.

**Data-room (build now — doc 08):** CAC cert, TIN, status report, founder CVs, business plan, this blueprint, financial model (3-scenario, 5-yr — *to build*), technical architecture, pilot results, IP/assignment docs, cap table, M&E framework.

**What investors will test:** real demand (doc 04 evidence), unit economics, team's ability to execute (already shown via working product + publication), and a credible path beyond one station.

---

## 13. Grant readiness strategy

The company's `GRANT_READINESS.md` already maps ZE-Gen-style criteria; the ChargeHub satisfies them concretely:

- ✅ **Direct fossil-fuel (diesel/generator) displacement** — solar-first logic minimises generator runtime.
- ✅ **Safe, flexible, affordable, reliable, low-carbon** — all four are explicit design goals.
- ✅ **Multi-sector applicability** — education, commercial, public, humanitarian.
- ✅ **Hybrid AC+DC + storage + grid-charge backup** — designed in (HPTS + battery).
- ✅ **Scalability & modularity** — station network model.
- ✅ **Local supply chain + capacity building** — Nigerian build/maintenance + technician training.
- ✅ **M&E** — kWh, diesel displaced, CO₂ avoided, users served, jobs (instrument from day 1).
- ⏳ **To prepare:** detailed financial model, 2 letters of support (host + LGA/faculty), safeguarding/gender-inclusion policy, formal M&E framework.

**Target programs:** ZE-Gen Accelerator, Energy Catalyst, GIZ EnDev, REA/SEFA-adjacent energy-access funds, climate-tech accelerators, university innovation grants. Apply with a **named pilot in one named place** + measurable targets.

---

## 14. MVP-first recommendation (capital de-risking)

Rather than build the full solar station first, **prove demand cheaply, then add solar**:
- **MVP (M0–M2):** grid + generator + small battery + 8 smart outlets + web booking/payment. CapEx ~₦2.5–3.5M.
- **Validate (M2–M4):** measure real utilisation, price tolerance, repeat rate at UI.
- **Solarise (M4–M6):** add PV + larger battery once demand is evidenced — and fund it with the grant/competition won on the back of early traction.
This sequence matches lean-startup practice and the company's own "specificity over ambition" funding principle.

---

## 15. Partnership strategy

| Partner type | Examples | Value to us | Value to them |
|---|---|---|---|
| Host site | UI hall/faculty, library, campus works unit | Space, security, captive demand | Revenue share, student service, impact story |
| Payments | Paystack / Flutterwave | Cards, transfer, USSD, mobile money | Transaction volume |
| Connectivity | MTN / Airtel IoT SIMs | Reliable station comms | ARPU, IoT showcase |
| Components/solar | Local solar distributors (Tier-1 panels, certified inverters) | Trade pricing, support | Volume, reference deployments |
| Accelerator/grant | ZE-Gen, Energy Catalyst, GIZ, REA | Capital, mentorship, network | Portfolio impact, SDG metrics |
| Academic/R&D | UI Faculty of Technology | Lab access, student talent, credibility | Research output, placements |
| Sponsors/advertisers | Telcos, banks, FMCG | Branding revenue | Targeted reach at busy sites |

---

## 16. Regulatory considerations (summary — detail in doc 08)
- **Electricity licensing:** a single ≤100 kW behind-the-meter solar+storage system on private/institutional land typically falls under **captive/embedded generation** and does not need a NERC generation licence — *confirm with counsel and current EA 2023 / state regulations before energising.*
- **Electrical safety & standards:** install to NEMSA/IEE wiring standards; RCD/earthing/SPD; certified electrician sign-off; periodic inspection.
- **Payments & data:** comply with CBN payment rules (use licensed gateways) and NDPR (Nigeria Data Protection) for user data.
- **Consumer/metering:** transparent pricing display; accurate metering (the ACS712-based metering already supports this).
- **Tax & corporate:** TIN active; keep VAT/CIT obligations current; insurance for equipment/public liability.

---

## 17. Operational plan & 6-month execution roadmap

**Team mode:** weekly stand-up + bi-weekly build review; everything in version control + a shared drive; RACI per doc 06.

### Month-by-month

**Month 0 — Foundation & validation (in parallel)**
- Kickoff meeting (doc 05); confirm roles (doc 06); set up repos, drive, bank, NDAs/IP assignment (doc 08).
- Launch customer-discovery: survey + 15–20 interviews at UI (doc 04).
- Re-quote BOM; finalise MVP-first vs full-solar decision; secure host MoU conversation.
- Pull the old PHP platform + Arduino code out of the archive; audit reusability (doc 02).

**Month 1 — MVP build (hardware + software)**
- Rebuild smart-outlet controller (Arduino Mega/ESP32, relays, ACS712, keypad/LCD, SD) from the proven prototype; modernise comms (Wi-Fi/MQTT).
- Stand up cloud platform v2: refactor booking/PIN engine; integrate Paystack/Flutterwave; SMS/email notifications.
- Integrate HPTS source-switching module (grid+gen+battery).
- Lock host site + siting agreement.

**Month 2 — Integration, safety, MVP pilot soft-launch**
- End-to-end test (book → pay → PIN → unlock → meter → cutoff → notify).
- Electrical safety certification; enclosure + signage.
- **Soft-launch MVP** at UI with limited outlets; start collecting real usage + payment data.

**Month 3 — Iterate + funding push**
- Fix top issues from soft-launch; tune pricing from real data.
- Submit first grant/competition with early-traction evidence; apply to an accelerator.
- Build the 3-scenario financial model + investor data-room.

**Month 4 — Solarise + scale-readiness**
- Add PV + battery (full hybrid); verify solar-first switching reduces grid/gen use.
- Harden monitoring, admin console, maintenance dashboard, OTA.
- Start mobile-app MVP (or PWA) per doc 02.

**Month 5 — Full pilot operations**
- Run full station at target utilisation; track all KPIs; gather testimonials + case study.
- Begin host conversations for stations #2–#3 (UI replication).

**Month 6 — Evidence, raise, plan v2**
- Publish pilot results + impact (kWh, diesel displaced, CO₂, users served).
- Close grant/accelerator outcome; open angel/convertible conversations.
- Lock the 12-month scaling plan (§11) and budget.

### Gantt (textual)
```
Task \ Month             M0   M1   M2   M3   M4   M5   M6
Validation / discovery   ███  ██
Company/legal/banking    ███  █
Outlet hardware rebuild       ███  ██
Cloud platform v2        ░    ███  ██
HPTS energy module            ██   ██
Safety cert + install              ███
MVP soft-launch                   ███  ██
Pricing/iterate                        ███  █
Grant/accelerator push                 ███  ██
Solar + battery add                         ███  █
Mobile app MVP                              ░    ███  ██
Full pilot ops                                   ███  ███
Results + fundraise                                   ██  ███
```

---

## 18. KPIs / how we measure success

| Dimension | Metric | 6-month target |
|---|---|---|
| Demand | Unique paying users | ≥ 150 |
| Stickiness | Repeat-usage rate | ≥ 65% |
| Reliability | Station uptime | ≥ 80% |
| Integrity | % energy metered & billed | ~100% (zero theft) |
| Sustainability | Renewable share of energy delivered | ≥ 60% |
| Impact | Diesel litres displaced / CO₂ avoided | tracked monthly |
| Commercial | Monthly net revenue trajectory | toward unit-economics breakeven |
| Funding | Grant/accelerator submitted; MoU signed | ≥ 1 each |
| Satisfaction | NPS | ≥ +30 |

---

## 19. Appendix — provenance of claims
- Smart-outlet design, PIN scheme, metering, tariff/LCC/carbon economics: *Olufajo, Ogundipe, Atanda, Fakolujo — "Sustainable Smart Power Outlet Controller with Online Energy Management System for Public Charging Stations," SSRG IJEEE 8(5):16–23, 2021.*
- Award: *Design & Prototype Implementation of Smart Power Outlets with an Online Management System — NSE Babafunso Graduate Engineer of the Year, 3rd, Aug 2020.*
- HPTS switching performance (3.8 s gen start, 0.15 s transfer, 95.2% efficiency): *First draft — IoT-Based Hybrid Power Transition Controller for Automated Source Switching* (team document).
- Company registration, vision/mission, units, grant-readiness: *Limitless Power Solution Concepts* business plan + `docs/` (BUSINESS_STRUCTURE, GRANT_READINESS) + brand site.
- Smart-meter expansion path: *Limitless Team — Lagos Smart Meter Hackathon 2020* submission.
