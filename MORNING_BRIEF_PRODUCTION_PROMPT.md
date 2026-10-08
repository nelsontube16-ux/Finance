# Morning Brief v2 — Production Prompt (merged)

Copy everything between the `===` lines into claude.ai Scheduled Tasks, replacing the old "Daily Digest" prompt. Schedule (Asia/Shanghai): `30 7 * * *` (Morning Brief, daily), `0 14 * * 1-5` (Midday Flash, weekdays), `0 23 * * 1-5` (US Open Flash, weekdays). If one task can only hold one time, create three tasks with this same prompt.

===

Build Nelson's automated analyst-grade market brief and deliver it to his Gmail (nelsonsusanto12@gmail.com). Use only free sources and services. Do not use any paid service or paid API. No manual input from Nelson is needed or expected.

ENVIRONMENT NOTE: you run in an Anthropic cloud sandbox on Linux, not on Nelson's Windows PC. You have no access to his local files, local scripts, or local credentials, so do not try to read local paths or run PowerShell. Everything you need is inlined in this prompt. Delivery and data come from web search plus the Gmail and Google Calendar connectors.

CONTEXT ON NELSON: Indonesian student in Shanghai (FISF/Fudan), studying finance and preparing for CFA Level 1, reading "Financial Shenanigans", building a Qualcomm financial model, hunting for a finance internship. Side project: China-Indonesia sourcing-as-a-service (TikTok Shop/Shopee channel, vegetarian products niche). Trains for triathlon; his sessions are in Google Calendar (keywords: swim, bike, run, brick, long ride).

PORTFOLIO SOURCE: first run Gmail search_threads, query: from:nelsonsusanto12@gmail.com subject:"PORTFOLIO UPDATE", and read the newest match with get_thread. If found, its body is the current portfolio (same format as the default list below) and fully replaces the default list; say "Portfolio as of [that email's date]" in the Book Summary. If none is found or it cannot be parsed, use the default list below and say so.
DEFAULT PORTFOLIO:
- BBCA (IDX, Bank Central Asia): avg Rp 7,225 · 7 lots (700 shares)
- BACH (IDX, PT Bach Multi Global): avg Rp 715.24 · 132 lots (13,200 shares)
- BRMS (IDX, Bumi Resources Minerals): avg Rp 1,000 · 101 lots (10,100 shares)
- GOOGL (NASDAQ, Alphabet Class A): avg $396.18 · 1 share
- SLV (NYSE Arca, iShares Silver Trust): avg $94.99 · 0.30 share
- DRY POWDER (uninvested cash): Rp 1,000,000

STEP 0, DAY AND SLOT CHECK
Run: TZ=Asia/Shanghai date '+%Y-%m-%d %A %H:%M'
That output is today's date, day-of-week and time for the entire run. Use it everywhere a date is needed, never the sandbox's own local time. Pick the slot by hour:
- 06:00 to 11:59: MORNING BRIEF (full, all sections in Step 4)
- 12:00 to 17:59: MIDDAY FLASH (short)
- 18:00 to 05:59: US OPEN FLASH (short)
If the day is Sunday and the slot is MORNING BRIEF, this run also builds the WEEKLY REPORT (Step 6), appended to the same email.

STEP 1, DUPLICATE CHECK
Gmail search_threads, query: (in:sent OR in:draft) subject:("Morning Brief" OR "Midday Flash" OR "US Open Flash") after:YYYY/MM/DD with today's date. Ignore any subject containing "Preview" or "Setup".
- If an email with THIS slot's exact subject (Step 5) was already SENT today, do not send another. Stop and report the skip in Step 7.
- If only a leftover DRAFT for this slot exists and nothing was sent, build and send anyway, and note the stale draft in Step 7 so Nelson can delete it. Do not delete anything yourself.
- Otherwise proceed.
Also search: in:sent subject:("Morning Brief" OR "Midday Flash" OR "US Open Flash" OR "Daily Digest") newer_than:3d, read the subjects and snippets, and do not repeat a story already covered unless there is a material new development (then say what is new).

STEP 2, DATA GATHERING
Run web searches in parallel. Prefer credible primary and wire sources: Reuters, Bloomberg, FT, WSJ, Nikkei, Caixin, CNBC, CNBC Indonesia, Kontan, Bisnis Indonesia, Jakarta Post, central bank and exchange sites, investing.com, tradingeconomics.com, stockanalysis.com.
DATA RULE: every number must come from a source you actually found in this run. Never estimate, extrapolate or fill from memory, and this includes YTD, 1M and daily-change columns. If a number is not found, write "n/a". If a figure is older than the latest session, tag it "as of [date]". If sources conflict, use the one with a clear date and note the conflict briefly.

Markets (every slot, as relevant to the slot):
- Equities: S&P 500, Nasdaq, Dow, Nikkei 225, Hang Seng, Shanghai Composite, IHSG, DAX
- FX: DXY, USD/IDR, USD/CNH, EUR/USD, USD/JPY
- Rates: UST 2Y, UST 10Y, UST 30Y, INDOGB 10Y, Bund 10Y, JGB 10Y
- Commodities: Brent, WTI, gold, silver, copper, nickel (LME), coal (Newcastle), CPO (KPBN Inacom or Bursa Malaysia)
- Top movers: biggest gainers and losers of the last session in the S&P 500 and IHSG (3 each), with the reason for each move

Portfolio (every slot): latest close and previous close for BBCA, BACH, BRMS, GOOGL, SLV, plus any news, filings, rating or target changes in the last 24h. Flag any price older than 3 days.

News (2-3 varied searches per bucket, last 24-48h). Region does not matter; what matters is urgency and impact on markets or the economy now or later:
- Macro and central banks: Fed, BI, PBoC, ECB, BOJ and others; inflation, jobs, GDP, PMI prints; central bank speakers
- Geopolitics and politics: only what moves markets or will (US-China, Middle East/Hormuz, Russia-Ukraine, Taiwan, elections, sanctions, trade policy)
- Corporate and finance: earnings (today's reports first), M&A, guidance changes, rating changes, short-seller reports, earnings-quality and accounting red-flag stories, IPOs, defaults
- Indonesia: IDX foreign flow, LQ45 movers, OJK/BI statements, major emiten news, fiscal and regulatory moves
- Rumours: allowed only from credible outlets (e.g. "people familiar" reports from Reuters/Bloomberg/FT) and only if high-impact; label them "Unconfirmed".
- Business watch (include only when genuinely notable): Indonesia-China trade and tariffs, export-import rules, TikTok Shop/Shopee policy, plant-based/vegetarian trends in Indonesia or China.
Nothing is excluded by topic (crypto, politics, etc. are allowed) as long as it is credible and important to markets or the economy. Skip gossip with no financial angle.

Calendar ahead: today's economic data (time in CST, consensus, prior), today's and tomorrow's earnings (pre-market and after-hours separately), central bank meetings and speakers, Treasury/SUN auctions, scheduled geopolitical events.

STEP 3, CALENDAR PULL
Google Calendar list_events on the primary calendar, timeZone Asia/Shanghai, orderBy startTime: today 00:00 to 23:59, and today to 7 days ahead. Extract only summary + start (+ end) per event before using it; never dump raw output. Pick out triathlon sessions (swim, bike, run, brick, long ride), Qualcomm sessions, deadlines and anything else time-critical. If the calendar connector is not authorized, write one line "Google Calendar not accessible" and continue.

STEP 4, BUILD THE EMAIL (FT style, English)
Writing: English, analyst tone, short sentences, no filler, no em-dashes in body text. Each story: bold headline, source + date, 2-3 sentences on what happened, then one line starting "Read-through:" with the market or sector implication (not a personal angle).
Design: one HTML body for Gmail (Gmail strips SVG, scripts, external CSS and most <style> blocks, so everything is inline-styled tables). Background #FFF1E5, text #262A33, accent #990F3D, up #007A33, down #CC0000, neutral/amber #B07500, table header #F5E3D0. Georgia for headlines, Arial for body and tables. Max width 760px.
Visuals: pure HTML tables with inline styles only. No SVG, no images, no PNG, no charts as attachments. Make it look like a polished FT/Bloomberg newsletter, visual first, text second:
- Layout: nested tables, width 100% inside a centered 760px container; generous padding (16-24px per section); white cards (#FFFFFF) on the salmon background with a 1px #E6D3C2 border; 28-32px space between sections; section headers in Georgia 20px with a 2px #262A33 rule and a small uppercase #990F3D kicker label above.
- KPI tiles: a grid of white cards, 2 per row (3 per row overflows on phones), for S&P 500, IHSG, USD/IDR, UST 10Y, Brent, gold. Each card: a 4px top border in green/red by direction, small uppercase label, big number (22px bold), % change below with an ▲/▼ arrow, so direction never depends on color alone.
- Data tables: equities, FX, rates and commodities as separate stacked tables (never side by side), with LAST, %CHG and YTD where sourced; zebra rows (#FFFFFF / #FBF4EC), right-aligned tabular numbers, change cells in green/red text with ▲/▼.
- Heatmap: one grid of equal cells (indices + portfolio names), each cell's background tinted by % move: 3 shades of green and 3 of red (darker = bigger move, e.g. under 1%, 1-3%, over 3%) and #EFE5DA for flat; ticker and % printed inside each cell in readable contrast.
- Bars: horizontal bars built from two table cells (filled cell width = value share, empty cell = remainder), 14px tall, for allocation % and for P&L % per position, value labelled at the end of each bar.
- Range bar: for each portfolio position, a 52-week range bar (low to high) with a marker cell at the current price and the avg cost marked, so the position in its range is visible at a glance. Only if the 52-week range was sourced; otherwise omit the bar.
- Badges: CALL shown as a rounded pill (border-radius 12px, padding 3px 10px, white bold text) on green BUY/ADD, grey HOLD, amber TRIM/REVIEW, red SELL; ALERT as a red pill.
- Story cards: each story in its own white card with a 3px left border colored by section, headline in Georgia bold, source/date in small grey caps, "Read-through:" in bold #990F3D.
- Mobile: no fixed widths wider than 760px; font sizes no smaller than 12px; tables must not overflow on a phone (use percentages).
- Length: no cap. Gmail clips emails over ~102KB behind a "View entire message" link, and that is accepted. Because of that, keep the order of sections exactly as listed below so the most important content (Opening Shot, markets, portfolio) is always above any clip point.

MORNING BRIEF sections, in order:
1. Masthead: "Morning Brief · Nelson", date, time, slot
2. The Opening Shot: 3 bullets, max 2 sentences each: the biggest story, the biggest FX/rates move, the biggest portfolio-relevant signal
3. Markets Overnight: KPI tiles, the three data tables, the heatmap, then Top Movers (S&P 500 and IHSG, 3 up and 3 down each, with reasons)
4. Your Portfolio:
   - Book summary: IDX book in IDR (cost, market value, P&L, P&L %), USD book in USD (same), total AUM in IDR converted at the sourced USD/IDR rate (state the rate), dry powder, allocation % per position, sector exposure, currency exposure (IDR vs USD %), day change in value vs previous close
   - Position tables: IDX and USD separately (IDR only for IDX names, USD only for US names), max 5 columns so they fit a phone: TICKER (market value in small text below), AVG → LAST (price date below if not today's close), DAY %, P&L %, CALL. Share counts go in a footnote.
   - CALL is BUY, ADD, HOLD, TRIM, SELL or REVIEW, shown as a pill badge
   - Visuals: allocation bars, P&L % bars, and a 52-week range bar per position (see Visuals)
   - Per-position commentary: fundamentals (valuation vs peers, earnings momentum), technicals (trend, RSI or moving averages if sourced), upcoming catalyst, and a clear answer to "is it worth selling?" with the reason
   - EXIT DISCIPLINE: Nelson's goal is profit, and he wants to sell before a fall rather than ride it down. For every position give a take-profit level and an exit trigger (a price or technical level such as a break below support or a key moving average, or a named negative catalyst), both from sourced data. Say plainly when an exit trigger has been hit and recommend acting on it. For positions already at a loss, judge on what the stock is likely to do from here, not on the purchase price: if the trend and catalysts point lower, say cut; if a recovery case exists, state it and the level that would invalidate it. Never imply tops or bottoms can be timed reliably.
   - SIZING RULE: any BUY/ADD must fit inside the dry powder; state exact size and cost (e.g. "1 lot BBCA = 100 shares x Rp 6,050 = Rp 605,000"). If a name cannot be bought within the dry powder, say so and do not recommend it.
5. Top Stories: grouped Macro / Geopolitics and Politics / Corporate and Finance / Indonesia / Business Watch (only if notable). As many stories as are genuinely important; no fixed cap. Order by impact.
6. Day Ahead: economic data table with 3 columns (time CST; event with a one-line expected impact in small grey text below; consensus with prior in small grey text below), earnings today and tomorrow (one-line why it matters), central bank speakers and auctions, then "Your Calendar" with today's events and deadlines in the next 7 days.
7. Learning Corner: one CFA Level 1 concept, 150-250 words, teaching tone, tied to today's biggest market event with a "CFA tie-in" line (link to his Qualcomm model when it fits). If no event is a good hook, teach a standalone CFA L1 module (Ethics, Quant, Economics, FSA, Corporate Issuers, Equity, Fixed Income, Derivatives, Alternatives, Portfolio Management), rotating topics across days and weighting the heavier exam topics (Ethics, FSA, Equity, Fixed Income) more often. Nelson's exam date is not set, so there is no countdown or pacing schedule.
8. Footer: one line with the data timestamp. No "reply more/less" line.

MIDDAY FLASH and US OPEN FLASH sections: masthead with slot tag, 1-2 Opening Shot bullets, KPI tiles for the session that matters (Asia for midday, US pre-market/futures for US open), portfolio snapshot table with CALL, up to 3 breaking stories with Read-through, and a short "Next 12 hours" list. If a portfolio name has major news (move over 5%, suspension, earnings, rating change, corporate action), put it first and mark it "ALERT".

STEP 5, DELIVERY (Gmail connector; SMTP is not available here)
send_message to nelsonsusanto12@gmail.com, htmlBody = the HTML, subject:
- Morning: "Morning Brief — YYYY-MM-DD"
- Sunday morning: "Morning Brief + Weekly — YYYY-MM-DD"
- Midday: "Midday Flash — YYYY-MM-DD"
- US Open: "US Open Flash — YYYY-MM-DD"
If send_message fails for any reason, fall back to create_draft with the same To/Subject/htmlBody so the work is not lost, and say so in Step 7.
Then best-effort apply the Gmail label "Daily Digest" to the thread: list_labels for its ID, create_label if missing, then label_thread (fall back to label_message). If labeling fails with a permission error, skip it. The email itself is the success condition.

STEP 6, WEEKLY REPORT (Sunday morning only, appended under an h2 "Weekly Report" with the #990F3D accent)
All sections in English; the personal sections (d and e) in a supportive, reflective tone. "This week" = the 7 days ending today (Asia/Shanghai). If any single source is unavailable, note it briefly in that section and continue; never abort the whole weekly report.
  a. Week in Review: 1-week change for the main indices, FX, rates and commodities (sourced), the portfolio's week, the 3 biggest macro/market events and what they changed.
  b. Week Ahead: full calendar for the next 7 days (data releases with consensus, earnings, central bank meetings and speakers, auctions, scheduled geopolitical events), each with a one-line expected impact, plus a 2-3 sentence sourced outlook.
  c. Email Recap: Gmail search_threads, query newer_than:7d in:inbox, pageSize 30. Ignore noise (mass job alerts, promos, newsletters) unless clearly relevant. Flag only what is significant or needs follow-up: security alerts, personal emails, deadlines. For long emails use get_thread and strip HTML before summarizing.
  d. Recap Agenda: from the calendar, what happened this week (triathlon sessions, Qualcomm sessions, deadlines) and a preview of the next 7 days.
  e. Targets for Next Week, exactly 2 targets:
     - Fitness: based on next week's triathlon sessions in Google Calendar (swim/bike/run/brick/long ride), framed as a supportive commitment. If none are scheduled, say so gently.
     - Study: one specific, optional CFA L1 reading or Financial Shenanigans chapter tied to the week's biggest market theme, framed as an invitation to try, not an obligation.

STEP 7, NOTIFICATION
Send a PushNotification on MORNING BRIEF runs, and on FLASH runs only when there is a portfolio ALERT or a skip/failure to report. Wrap the body in <routine_summary> tags. The lead sentence is the phone banner; the rest is the email body. Cover: slot and whether it was SENT, skipped (already sent), or saved as a fallback draft; total portfolio P&L and day change; the 2-3 biggest market moves; any portfolio ALERT; key catalysts in the next 24h; any stale draft to delete; on Sundays the weekly highlights and the 2 targets. If PushNotification is unavailable, end the run with the same summary in <routine_summary> tags as the final message.

SUCCESS: the brief for this slot was sent to nelsonsusanto12@gmail.com, or correctly skipped because it was already sent, or saved as a fallback draft if sending failed. On Sunday mornings the weekly report is appended or each missing part is explicitly noted. A summary notification is the final step on Morning Brief runs and on any Flash run with an ALERT, skip or failure.

===
