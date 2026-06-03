# [SPEC] How to Scrape a Website Without Getting Blocked

> **About this piece:** This is an original sample written to demonstrate technical,
> developer-audience writing in the style a web-scraping product would publish. It's a
> spec piece — written to show capability, not a client deliverable.
>
> _Mónica: this is a working draft scaffold. Read it, fact-check every technical claim,
> rewrite it fully in your own voice, and add real code you've tested. The point is to
> show YOUR writing for a dev audience — so make it yours. Notes to you are in `> blockquotes`._

---

If you've ever written a scraper that worked perfectly for ten minutes and then started
returning `403 Forbidden` on every request, you've met the wall. Websites don't love
being scraped, and most have defenses that spot automated traffic fast.

This guide walks through *why* you get blocked and the practical techniques to avoid it —
from the basics anyone can implement to the heavier infrastructure you'll eventually need.

> _Note to self: keep the intro to ~3 sentences max for a dev reader. They scanned the
> H1, they know the problem, get to the value._

## Why you're getting blocked in the first place

Before fixing it, it helps to know what you're up against. Sites flag scrapers using a
handful of signals:

- **Request rate.** A human doesn't load 200 pages in 4 seconds. A script does.
- **Missing or suspicious headers.** Real browsers send a rich set of headers; a bare
  `requests.get()` does not.
- **IP reputation.** Too many requests from one IP, especially a known datacenter IP,
  is an instant red flag.
- **No JavaScript execution.** Many bot checks rely on JS challenges a simple HTTP
  client never runs.

> _Note to self: this section is where I'd want a quick diagram. Add later._

## Technique 1 — Set a real User-Agent (and rotate it)

The single most common mistake: scraping with the default Python `requests` user-agent,
which announces "I am a bot" to every server.

```python
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) "
                  "Chrome/120.0 Safari/537.36"
}
response = requests.get(url, headers=headers)
```

> _Note to self: VERIFY this code runs before publishing. Rotate UAs from a list for
> anything beyond a handful of requests._

## Technique 2 — Slow down and randomize

Add delays between requests, and randomize them so the pattern doesn't look mechanical.

```python
import time, random
time.sleep(random.uniform(1, 4))
```

## Technique 3 — Rotate your IP with proxies

This is the big one. Once a site rate-limits or bans your IP, no amount of clever
headers helps. Rotating proxies spread your requests across many IPs so no single one
trips a limit. Residential proxies (real consumer IPs) are far harder to detect than
datacenter ones.

> _Note to self: this is where a product like ScraperAPI naturally fits — it handles
> proxy rotation, headers, and retries for you. In a real client piece this would be the
> soft product tie-in. Keep it useful, not salesy._

## Technique 4 — Handle JavaScript-rendered pages

If the content you need only appears after JS runs, a plain HTTP request returns an empty
shell. You'll need a headless browser (Playwright, Puppeteer) or an API that renders JS
for you.

## When to stop building this yourself

There's a point where maintaining your own proxy pool, browser farm, and retry logic
costs more engineering time than it's worth. That's usually the signal to reach for a
managed scraping API and get back to building your actual product.

## Takeaways

- Blocks come from rate, headers, IP reputation, and JS challenges.
- Start with real headers and randomized delays.
- Scale up to rotating residential proxies when you hit limits.
- Use a headless browser or rendering API for JS-heavy sites.
- Know when to buy instead of build.

> _Note to self — checklist before this goes in the portfolio:_
> - _Run every code block. No broken examples in front of engineers._
> - _Rewrite in my own voice; cut anything that sounds like AI-generic._
> - _Add one diagram or a real screenshot._
> - _Keep it honest: I'm demonstrating I can write the FORMAT, with SME/AI support on detail._
