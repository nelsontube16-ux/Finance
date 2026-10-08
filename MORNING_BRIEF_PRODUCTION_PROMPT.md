# Morning Brief v2 — Production Prompt

**Instructions:** Copy the entire block below (between the `===` lines) and paste it into claude.ai → Scheduled Tasks, replacing the current "Daily Digest" prompt. Set schedule to **3 times per day Asia/Shanghai time**: `30 7 * * *`, `0 14 * * *`, `0 23 * * *`.

===

Build Nelson's automated analyst-grade Morning Brief and deliver it to Gmail (nelsonsusanto12@gmail.com). FT-style HTML. Free sources only, no paid API, no manual input.

ENVIRONMENT: Anthropic cloud sandbox on Linux. No access to Nelson's local files/scripts/credentials. Everything inlined here. Delivery via Gmail connector, calendar via Google Calendar connector.

PROFILE: Indonesian finance student at FISF/Fudan Shanghai, CFA L1 candidate, reading Financial Shenanigans, running Qualcomm financial model exercise, side project China-Indonesia sourcing (TikTok Shop vegetarian niche), training triathlon (keywords in Google Calendar: swim, bike, run, brick, long ride).

PORTFOLIO (hardcoded, update manually in this prompt if changed):
- BBCA (IDX): avg Rp 7,225 · 7 lots (700 shares)
- BACH (IDX, PT Bach Multi Global): avg Rp 715.24 · 132 lots (13,200 shares)
- BRMS (IDX, Bumi Resources Minerals): avg Rp 1,000 · 101 lots (10,100 shares)
- GOOGL (NASDAQ): avg $396.18 · 1 share
- SLV (NYSE Arca, iShares Silver Trust): avg $94.99 · 0.30 share
- DRY POWDER (uninvested cash): Rp 1,000,000 total

SIZING RULE: Any BUY/ADD call must fit inside the dry powder. State the exact size and cost (e.g. "1 lot BBCA = 100 shares × Rp 6,050 = Rp 605,000"). If a name can't be bought within the dry powder (e.g. 1 GOOGL share > Rp 1jt), say so and suggest HOLD or a fundable alternative instead. Show dry powder as a line in the Book Summary.

STEP 0 — DAY & SLOT CHECK
Run: `TZ=Asia/Shanghai date '+%Y-%m-%d %A %H:%M'`
That is today's date, day-of-week, and time. Decide slot by hour:
- 06:00–11:00 → MORNING BRIEF (full version, all sections)
- 12:00–17:00 → ASIA MIDDAY FLASH (portfolio snapshot + IDX/HSI intraday + top 3 breaking news only)
- 18:00–23:59 → US OPEN FLASH (US pre-market + overnight news + portfolio USD leg)
If Sunday AND slot is MORNING → also append WEEKLY REPORT at the bottom.

STEP 1 — DEDUP CHECK
Gmail search_threads: `in:sent subject:("Morning Brief" OR "Midday Flash" OR "US Open Flash") after:YYYY/MM/DD` with today's date. Ignore any subject containing "Preview". If an email with THIS slot's exact subject (see STEP 5) was already sent today, STOP and push notification "already sent, skipped". Otherwise proceed.

STEP 2 — DATA GATHERING (parallel WebSearches)
Prioritize credible sources: Bloomberg, Reuters, FT, WSJ, Nikkei, Caixin, CNBC Indonesia, Kontan, Bisnis Indonesia, investing.com, tradingeconomics.com, exchange sites. Pull most recent close/intraday for each. Tag "data as of [date]" where lag exists. If a specific number not findable, write "n/a" — never fabricate. This applies to every cell, including YTD, 1M and daily-change columns: only fill them from a source you actually found, never estimate.

**Markets (always):**
- Equities: S&P 500, Nasdaq, Dow, Nikkei 225, Hang Seng, Shanghai Composite, IHSG, DAX
- FX: DXY, USD/IDR, USD/CNH, EUR/USD, USD/JPY
- Rates: UST 2Y, UST 10Y, UST 30Y, INDOGB 10Y, Bund 10Y, JGB 10Y
- Commodities: Brent, WTI, Gold, Silver, Copper, Nickel (LME), Coal (Newcastle), CPO (KPBN Inacom)

**Portfolio prices (always):** BBCA, BACH, BRMS, GOOGL, SLV — pull latest close + previous close. If a price looks stale (>3 days old), note it inline.

**News (vary queries 2-3 per bucket):**
- Macro: Fed/BI/PBoC/ECB/BOJ actions, inflation prints, jobs data, central bank speakers
- Geopolitics: US-China, Middle East/Iran/Hormuz, Ukraine, Taiwan, elections with market impact
- Corporate: earnings (focus today's reports), M&A, major rating changes, unusual options flow, short-seller reports
- IDX: foreign flow, LQ45 movers, corporate actions, OJK/BI statements, major Indonesia emiten news
- Portfolio names: scan each ticker for breaking news in last 24h

**Economic calendar today:** Pull from investing.com/tradingeconomics, convert all times to CST Asia/Shanghai, include consensus + prior.

**Earnings calendar today & tomorrow:** Highlight pre-market and after-market separately.

STEP 3 — CALENDAR PULL
Google Calendar list_events for today 00:00–23:59 Asia/Shanghai, primary calendar. Extract events with keywords: swim, bike, run, brick, long ride (triathlon training), plus Qualcomm, DEADLINE, review, meeting. Also list_events for next 7 days to spot upcoming deadlines.

STEP 4 — BUILD HTML (FT style)
Palette: background #FFF1E5 (FT salmon), text #262A33, accent #990F3D (FT pink), green up #007A33, red down #CC0000, amber #B07500, table header bg #F5E3D0. Serif headlines (Georgia), sans-serif body (Arial). Max width 760px.

**Sections (MORNING BRIEF — full):**
1. **Masthead:** "Morning Brief · Nelson" + date + time + slot tag
2. **The Opening Shot:** 3 bullets, max 2 sentences each, must capture biggest overnight story + biggest FX/rates move + biggest portfolio-relevant signal
3. **Markets Overnight:** 3 tables (Equities / FX & Rates / Commodities) with LAST, CHG, %CHG, YTD, color-coded
4. **Your Portfolio:** Book summary (IDX total P&L, USD total P&L), position tables (IDX + USD separately) with AVG, LAST, SIZE, MKT VAL, P&L, CALL (BUY/HOLD/SELL/REVIEW with color), then per-ticker commentary paragraph (fundamental + technical + catalyst + specific action)
5. **Top Stories:** Grouped Macro / Geopolitics / Corporate / IDX. Each story: bold headline, source + date, 2-3 sentences, "Read-through:" analyst commentary line
6. **Day Ahead:** Economic data table (time CST, event, consensus, prior) + Earnings today + Your Calendar (triathlon training sessions, Qualcomm sessions, deadlines)
7. **Learning Corner:** 1 CFA L1 concept tied to today's biggest macro/market event. 150-250 words, teaching tone, include "CFA tie-in" link to Nelson's Qualcomm modeling practice when possible. If news is light, use a standalone CFA L1 module from the Reading list (Economics, Fixed Income, Equity, Corporate Finance, FRA, Portfolio Mgmt, Ethics)
8. **Footer:** Minimal, no "reply more/less" line

**Sections (MIDDAY/US FLASH — short):**
Masthead with slot tag + 1 Opening Shot bullet + Portfolio snapshot table only + 3 stories max + Day Ahead reminder. No Learning Corner.

**Sections (SUNDAY WEEKLY REPORT — appended to Morning Brief):**
- Header "Laporan Mingguan" with #990F3D accent
- **Week in Review:** Market performance recap (1-week change for all tables above), sector rotation summary, 3 biggest macro events of the week
- **Week Ahead:** Full economic calendar next 7 days, earnings calendar, Fed/central bank speakers, geopolitical events scheduled
- **Email Recap:** Gmail search newer_than:7d in:inbox, extract significant (security alerts, personal, deadlines), ignore noise
- **Calendar Recap:** Last 7 days triathlon sessions attended + Qualcomm hours logged + next 7 days preview
- **3 Personal Targets:**
  - Fitness: pull next week's triathlon sessions from Google Calendar (swim/bike/run/brick/long ride) and frame as commitment
  - CFA Study: suggest a specific CFA L1 reading tied to the week's biggest market theme
  - Kebiasaan: supportive one-liner only

STEP 5 — DELIVERY
Gmail send_message to nelsonsusanto12@gmail.com. Subject:
- Morning: "Morning Brief — YYYY-MM-DD"
- Midday: "Midday Flash — YYYY-MM-DD"
- US Open: "US Open Flash — YYYY-MM-DD"
- Sunday Morning: "Morning Brief + Weekly — YYYY-MM-DD"
htmlBody = the HTML. If send fails, create_draft fallback.
Then label the thread "Daily Digest" (Label_3). If label fails, skip.

STEP 6 — PUSH NOTIFICATION
Always send PushNotification wrapped in <routine_summary>. Lead sentence = banner. Include: slot sent/skipped, portfolio total P&L delta vs yesterday, 1-2 biggest moves of the day, any critical catalyst in next 24h (central bank, earnings, deadline).

SUCCESS: brief sent, labeled, push fired. On Sunday morning, weekly report appended.

===

**Suggested cron (claude.ai Scheduled Tasks, Asia/Shanghai timezone):**
```
30 7 * * *     Morning Brief (full)
0 14 * * *     Asia Midday Flash
0 23 * * *     US Open Flash
```

**To update portfolio in prompt:** Edit the "PORTFOLIO (hardcoded)" block when you add/trim/sell. Avg price = your effective cost basis per share.
