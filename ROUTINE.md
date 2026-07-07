# Morning Brief — Merged Routine Spec

**Canonical source of truth for the daily scheduled routine.** The routine prompt on
claude.ai/code points at this file; edits here take effect on the next run.

Runs daily, before market open (Jakarta time). Output is a single HTML brief plus a
Gmail draft ready to send.

Owner: nelsontube16@gmail.com
Repo: nelsontube16-ux/Finance
Branch: `claude/zen-lamport-5xwsw9`

---

## 1. What this routine replaces

This routine **supersedes and merges**:
- The prior _"Morning Brief"_ routine (finance-first) — kept as the spine.
- The prior _"Daily News Digest"_ routine — folded in as a short "World & Tech Beyond Finance" strip so nothing is missed but nothing duplicates.

**After the merge, the "Daily News Digest" routine on claude.ai/code should be deleted.**
(That deletion is a UI action — do it once, from the claude.ai/code routines panel.)

---

## 2. Sources to poll (in this order, in parallel where possible)

Financial spine:
- Wall St close: S&P 500, Dow, Nasdaq, VIX
- Asia-Pacific session: Nikkei, Kospi, Hang Seng, CSI300, ASX
- Indonesia: **IHSG close, sector heat-map, top 10 movers**, foreign flow, rupiah, BI reserves
- Rates: US 2y/10y, Fed funds, Fed speakers overnight
- FX: DXY, USD/IDR, USD/JPY, EUR/USD
- Commodities: Brent, WTI, gold, nickel, CPO
- M&A wire: last 24-48h announced deals ≥ $500m
- IPO pipeline: this week's pricings, next week's launches
- Sell-side calls: notable upgrades/downgrades, PT changes
- Indonesian-specific names: BBRI, BBCA, BMRI, TLKM, ASII, ICBP, MYOR, ANTM, MDKA, GOTO (default watchlist — user can override)

Broader digest (from the folded-in Daily News Digest routine, kept **headlines-only**):
- Top 3 world/geopolitics headlines not already covered above
- Top 2 tech/AI headlines with market implications
- Top 1 Indonesia domestic policy / macro headline

Personal:
- Gmail: unread + important, last 24h — 3-5 items worth surfacing
- Google Calendar: today's events (**only if the Calendar MCP is connected**; otherwise stub the section and remind the user to connect)

---

## 3. Output contract

Two artifacts, both required each run:

### A. HTML file
- Path: `output/(DATE) - Morning Brief.html` where DATE = `MMM D, YYYY` (e.g., `July 7, 2026`).
- Design: minimalist white background, editorial serif for body, mono for tags/labels, single accent color (`#b3261e`). No dark mode override; the brief is meant to read like a printed newspaper.
- Length target: **~14 minute read**. If content pushes past that, trim commentary before cutting facts.

### B. Gmail draft
- To: `nelsontube16@gmail.com`
- Subject: `Morning Brief — <DATE>` (e.g. `Morning Brief — Mon 07 Jul 2026`)
- Body: HTML mirror of the brief (inline `<style>`, no external assets). Also include a short 6-bullet TL;DR at the very top of the email so the phone preview shows something useful.
- The draft is created via the Gmail MCP `create_draft` tool. **The routine does not send** — the user hits Send from Gmail. (This is a hard constraint of the current MCP; upgrade to a `send`-capable Gmail integration later if truly hands-off is required.)

### C. Sections (in order)

1. **The Tape** — one-glance strip: Dow, S&P, Nasdaq, Kospi, Nikkei, Hang Seng, IHSG, Brent, WTI, gold, Fed funds, USD/IDR. Deltas colored green/red.
2. **The one thing to know** — a boxed feature: single most market-moving story of the last 24h in one paragraph.
3. **Macro & Central Banks** — Fed / BI / ECB / BoJ; jobs, CPI, PMI prints; rate expectations.
4. **Geopolitics** — Middle East, US-China, Europe/Russia. Three cards max, always closed with one investable theme.
5. **M&A** — 24-48h announced deals, top 3 by size or strategic importance. Always frame with "what this signals for deal comps."
6. **IPOs** — this week's pricings + one deep-dive on the biggest.
7. **Indonesia (Home Market)** — IHSG detail, sector heat-map, top gainers/losers, foreign flow, single ticker to watch today with an EV/EBITDA or P/E anchor.
8. **Street Calls** — analyst upgrades/downgrades/PT changes as a compact table.
9. **World & Tech Beyond Finance** _(digest folded-in)_ — 3 world + 2 tech + 1 Indonesian-domestic headlines, one line each. Purely informational; no commentary.
10. **Learn Today** — one crucial finance/IB/PE/M&A concept explained from zero. Rotate: valuation multiples → DCF mechanics → LBO structure → merger consequences (accretion/dilution) → deal process → capital structure → working capital → NWC adjustments → precedent transactions → comparable companies → WACC → terminal value → LBO returns bridge → covenants → PIK/mezz → earnouts → escrow / R&W insurance. Anchor the concept to a real event in today's tape whenever possible.
11. **Book of the Day** — micro-reading plan. Default rotation:
    - Weeks 1-9: _Investment Banking_ (Rosenbaum & Pearl) — one sub-section per day.
    - Then _Barbarians at the Gate_ (Burrough & Helyar) — 20 pages/day.
    - Then _Dealmakers_ (Cassidy) or _King of Capital_ (Carey & Morris).
    - Then _Distress Investing_ (Whitman & Diz).
    - Loop back with delta commentary.
12. **Δ Correlation with prior briefs** — flag: which themes strengthened / reversed, whether analyst PTs from prior briefs proved right, how the tracked watchlist moved vs. those calls. Reads the **previous day's brief file** in `output/` to compute deltas.
13. **Inbox (last 24h)** — 3-5 emails worth surfacing. Flag urgency clearly.
14. **Today's Agenda** — Calendar events. Stub with a "not connected" note if Calendar MCP is offline.
15. **For You** — 3-5 questions to sharpen tomorrow's brief. Rotate: sector focus, watchlist adjustments, macro overlay, delivery format, calendar events.
16. **Sources** — every claim linked. Grouped by category.

### D. Delivery

1. Save HTML to `output/(DATE) - Morning Brief.html`.
2. Create Gmail draft via `mcp__Gmail__create_draft` with HTML body.
3. `SendUserFile` the HTML back in-chat with `display: render`.
4. Commit + push to `claude/zen-lamport-5xwsw9`.
5. Notify user via `PushNotification` **only** if today's tape has a genuinely market-moving event (single-day index move > 3%, circuit breaker, macro surprise > 1σ, or a policy shift). Never notify on routine "all quiet" runs.

---

## 4. Style rules

- Voice: senior partner briefing an analyst. Direct. No hedging phrases like "it appears that."
- Every card ends with a **Why it matters** line — how a banker/PE analyst should act on the info.
- Use **EV / EBITDA** and **P / TBV** for banks, **EV / Revenue** for high-growth, **P / E** for stable industrials. Never mix.
- Numbers: comma thousands, 2-decimal % where meaningful.
- No emoji anywhere in the HTML.
- Filename date format is `MMM D, YYYY` per the user's original spec.

---

## 5. What was dropped vs. the Daily News Digest routine

- Long-form article summaries (redundant with the finance spine).
- Weather / sports (never relevant to the user's stated goal).
- Multi-paragraph "editorial takes" on non-financial news (replaced with the terse headlines-only strip).

## 6. What was added on top of the old Morning Brief

- The headlines-only World & Tech strip (Section 9).
- Automatic Gmail draft delivery to nelsontube16@gmail.com.
- Δ Correlation section that reads the prior brief file.
- Explicit rotating concept and book plans so nothing repeats.
- Compact table format for street calls (was prose).

---

## 7. Routine prompt to paste on claude.ai/code

> Run the Morning Brief for today. Follow the spec at `ROUTINE.md` on branch `claude/zen-lamport-5xwsw9` — read that file first, then execute end-to-end. Save the HTML, create the Gmail draft, commit, push, and deliver the file in-chat. If Google Calendar MCP is not connected, stub Section 14 and note it. Do not send push notifications unless a real market-moving event triggers the criteria in the spec.
