# Market dashboard API (research only, not trade advice)

Static files, refreshed with the dashboard (hourly publish, 8:38 AM to 10:38 PM ET). Base URL:
https://nikdep.github.io/market-dash-lkoa20/api/

Every JSON file is `{"as_of": ..., "count": N, "data": [...]}` plus a few file-level fields. `as_of` is the newest
data timestamp in that file (ET, ISO 8601), not the build time. Every JSON has a CSV twin with the same name
(nested fields flattened with dots, lists space-separated). Missing values are `null` in JSON and empty in CSV;
nothing is estimated or filled in. Prices come from Yahoo's chart endpoint (unofficial). Times are America/Toronto
(same clock as New York).

## candidates.json : boom candidates (ranked)
| field | meaning |
|---|---|
| rank | 1 = biggest upside if the narrative plays out |
| ticker | Yahoo symbol (e.g. `TOU.TO`, `3006.TW`) |
| direction | `gain` / `lose` (sign of jev_impact) |
| narrative, event_id | story behind it (E01-E12 tracked events, P01-P12 proposed) |
| trigger, catalyst | what would set it off; catalyst text (dated version in catalysts.json) |
| jev_impact | -1..+1 Jev score (direction x directness x size x not priced in) |
| jev_priced_in | 0..1 Jev read of how much the market already expects |
| jev_order, jev_horizon | first/second order link; days/weeks/months/years |
| rank, status, filter_reason | v4 combined ranking: `confirmed` (ranked by combined.up_risk_ratio) then `waiting` (by combined score) then `filtered` (watchlist, with reason) |
| combined.* | combined_score, confirmed_date/how (5%+ day on 1.5x volume or close above 20-day high), stop/stop_how/risk_pct (story-wrong stop: tighter of 1% below pre-call low and 2x ATR14), upside_est_pct/src (half 52-wk high + half analyst mean, capped), up_risk_ratio, analysts_used, days_since_call |
| rank_upside | rank under the v3 upside-first score |
| rank, sort_score, cap_tier, size_w, strip, low_float | v5 (small caps first): confirmed by up_risk_ratio x size_w, waiting by combined score x size_w (size_w: < $2B 2.0, $2-10B 1.4, $10-200B 0.85, > $200B 0.6 and strip = `large caps`); under $2M/day traded -> filtered; low_float = float under $250M or under 50% of shares |
| rank_combined_v4 | rank under the v4 combined ranking (before the small-cap weighting) |
| boom_score | v4 combined score |
| upside_score | v3 upside score: 0.45 payoff vs size + 0.25 room to run + 0.10 dated catalyst + 0.20 trend, x0.6 late call, x0.75 price broken, x0.97-1.03 source grade |
| upside.bull_pct, upside.bull_src | bull-case % move and its basis (analyst high target, or back to the 52-week high) |
| upside.base_pct, upside.below_52w_high_pct, upside.to_target_pct, upside.n_analysts, upside.short_pct_float, upside.days_to_cover | Yahoo (yfinance) analyst/52-week/short data |
| upside.catalyst_date, upside.catalyst_label, upside.days_to_catalyst | nearest dated catalyst (company earnings date from Yahoo or a confirmed story date) |
| upside.above_ma20, upside.rs10_pct, upside.late_call, upside.price_broken | trend and penalty flags |
| upside.up_payoff, up_room, up_catalyst, up_trend, size_factor | score parts (0..1) |
| upside.why_pop, upside.what_happen, upside.up_tags | plain-English lines and warning tags |
| source_grade, first_caller | source quality A-D (tiebreaker only) and the first account to call it |
| echo_score, rank_echo | echo-aware score/rank (Oct 2 v2) = boom_score_old x echo_mult |
| boom_score_old, rank_old | previous score/rank (impact, small-cap and first-order boost, already-ran penalty), kept for comparison |
| echo_mult | independence x earliness x crowding x outside-confirmation multiplier |
| rank_tags | short human-readable reasons (independent sources / early / crowded / outside confirmation) |
| n_independent, n_accounts, n_copycats, n_story_posts, n_quote_or_reply_posts | each account once per story; quotes/replies 0.3, copycats (first post within 24h of another account) 0.5 |
| prior_run_1m_excess_pct | excess vs SPY over the 21 trading days before the base post (>= 20% = late, penalised) |
| crowded | true if a burst of posts on the ticker/theme in the last 7 days (>= 3x prior 4-week rate) |
| outside_confirmation | `tg` AzazelNews mention, `qq_buy` QuiverQuant insider/politician purchase (30 days before post to now), `polymarket` odds >= 50% for the supportive outcome |
| polymarket | related Polymarket odds (supportive outcome), informational |
| ret_since_post_pct, spy_since_post_pct, excess_since_post_pct | daily closes, first close after the post to latest close |
| base_post_et, base_handle | the post the baseline is measured from |
| market_cap_usd_b, avg_dollar_vol_m | size and liquidity |
| main_risk | one-line risk |
| base_date, base_close, last_date, last_close, currency | price at the post (first close at/after it) and latest close, listing currency |
| chg_per_share | last_close - base_close |
| usd_1k_now, usd_1k_spy_now | hypothetical: what $1,000 bought at base_close is worth now; same in SPY over the same dates |

## catalysts.json : dated triggers per ticker
| field | meaning |
|---|---|
| ticker | Yahoo symbol |
| date | `YYYY-MM-DD` when the date is exact, else null |
| date_precision | `exact`, `window`, `month`, `rolling`, `past`, `unknown` |
| window_start, window_end | date range when only a window is known (e.g. "late Oct" = Oct 21-31) |
| event, event_id | what happens; tracked event id |
| direction | how the ticker is expected to move if the story plays out: `gain` / `lose` / `mixed` / null |
| source | `confirmed date` (company/regulator page), `exposures.csv`, `boom_candidates.csv` |
| note | context |

## handles.json : account records (daily + intraday)
| field | meaning |
|---|---|
| handle, source | X handle (no @) or `AzazelNews (Telegram)`; `x` / `telegram` |
| rank, watch, watch_score | ordering; `watch=true` = followed by the post alert; score = mean of (daily % beat SPY - 50%) and (+1h hit rate - 50%), each used only with n >= 3 |
| posts, first_post, last_post, narratives | activity |
| jev.testable / predictions / factual / opinion | Jev triage counts of the account's posts |
| daily.* | named tickers from the first close after each post to the latest close vs SPY (assumes bullish mention) |
| daily.usd_invested_1k, usd_pnl_1k, usd_spy_pnl_1k | hypothetical: $1,000 into each priced post x ticker (tickers_priced of them) at the first close after the post, held to the latest close: total stake, $ profit/loss, and the same $1,000 per call in SPY |
| intraday.<h>.usd_pnl_1k / usd_spy_pnl_1k | hypothetical $ profit/loss of $1,000 per post x ticker bought at base_price and sold at horizon h; same in SPY |
| intraday.<h>.n / mean_excess_pct / median_excess_pct / hit_rate_pct | h in `5m`, `15m`, `1h`, `1d`, `close`; excess = ticker minus SPY over the same minutes; hit = excess > 0 |
| intraday_rows, intraday_intervals | post x ticker pairs and bar sizes used (`1m:23 5m:3`) |

## posts.json : latest posts, newest first
| field | meaning |
|---|---|
| source | `x` (posts you sent), `x-followed` (a followed account's timeline, e.g. @QuiverQuant), `x-liked` (a market post Rocketman liked on X), `x-alert` (new post found by the watcher), `telegram` (AzazelNews stock picks, last 30 days) |
| handle, post_id, url | |
| posted_at_et, posted_epoch | post time (X: decoded from the post ID, to the second) |
| sent_at_et | when you sent it into the Grok chat |
| tickers | tickers named in the post (cashtags; Telegram: matched symbols) |
| text | post text, cut at 400 characters |
| jev.kind | `testable_prediction`, `factual_claim`, `opinion_or_value`, `rhetoric_or_insult`, `question_or_hypothetical` |
| jev.kind_conf, specificity (0-4), domain, confidence (0-3), deadline | Jev triage |
| jev.testable | `yes` (prediction, conf >= 0.6, specificity >= 2), `review`, `no` |
| narrative_id, narrative, engagement.* | grouping and likes/reposts/views/followers at fetch time |
| since_post | JSON only: per ticker `{ticker, symbol, currency, base_date, base_close, last_date, last_close, chg_per_share, ret_pct, spy_pct, excess_pct}` (first close at/after the post -> latest close) |
| since_post_text | same as one readable line (CSV too) |

## reactions.csv (X posts) and reactions_telegram.csv (AzazelNews) : one row per post x ticker (CSV only)
`interval` = bar size used (finest available for the post's age: 1m < ~30 days, 5m < ~60 days, 60m older).
`base_time_et`/`base_price` = open of the first bar at/after the post (next open if posted outside the session; `post_in_session`).
`ret_<h>` / `ex_<h>` = % return and excess over SPY for h in 5m, 15m, 1h (trading time, rolls into the next session), 1d (next session's close), close (same session's close).
`price_1h` / `price_1d` / `price_close` = implied price at that horizon (base_price x (1 + ret)); $ change per share = price_h - base_price.
`vol_spike` = volume in the first 15 min (5m bars) or 60 min (60m bars) after base / average of the same time-of-day window over up to 20 prior sessions (`vol_baseline_days`).
`data_note` explains blanks. Use it for event studies: no look-ahead, every return starts after the post.

Limits: tiny samples (tens of posts per account at most), daily candidates are multi-week ideas, no pre/post-market bars,
and SPY excess is blank when SPY isn't trading at the base time (Asian listings, crypto).
