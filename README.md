# linkedin scraper python: How to Scrape LinkedIn Jobs Without Getting Blocked, Plus Real Pricing Math Before You Pick a Plan

Most people searching for a linkedin scraper python setup want the same two things: working code that pulls job listings or profile data, and a way to keep it running past LinkedIn's rate limits. This article covers both. You'll get the working approach for scraping public LinkedIn job data with plain Python, and then a hard look at what it actually costs to run this at scale through ScraperAPI — including a detail almost nobody mentions: LinkedIn requests bill at 30 credits instead of the standard 1, which changes which plan makes sense.

**Quick picks**

> - Just experimenting? The free trial gives you 5,000 credits for 7 days, no credit card needed. That's roughly 160 LinkedIn requests at LinkedIn's 30-credit rate.
> - Regular scraping of public job listings? Hobby at $49/month (100,000 credits, 20 concurrent threads) covers about 3,300 LinkedIn requests monthly.
> - Need profiles, jobs, and company pages on a schedule with no login? That's where a scraping API earns its keep over homegrown scripts.
>
> 👉 [Try ScraperAPI free — 5,000 credits, no credit card required](https://www.scraperapi.com/?fp_ref=coupons)

## Why scraping LinkedIn with Python is harder than most sites

LinkedIn combines three defenses that break naive scrapers: dynamic JavaScript rendering, aggressive rate limiting, and IP-based blocking. Miss any one of them and your script either gets empty HTML, a login wall, or a permanently banned IP. The official framing from ScraperAPI's own engineering blog puts it plainly — scraping LinkedIn data is tricky "because of its dynamic, JavaScript-rendered content, rate limiting, and IP-based blocking," and going at it at scale means handling IP rotation, proxy pools, CAPTCHAs, and correct headers yourself, or risking a lifetime ban.

This is why the open-source Python libraries you'll find on GitHub — `linkedin_scraper` by joeyism, the Playwright-based recruiter tools, the Scrapy spiders — all come with the same caveats. They work until LinkedIn changes a selector or flags your IP, and then you're maintaining stealth plugins, session files, and retry logic instead of doing actual work. For a weekend project that's fine. For anything you depend on, the maintenance cost usually exceeds the price of an API that handles it.

One important boundary before going further: the reliably legal path is scraping only what's publicly visible without logging in — public job listings, public company pages. Logging in and scraping profiles behind the auth wall is a different risk category entirely, and no tool changes that.

## The trick that makes a Python LinkedIn jobs scraper work without Selenium

The job listings page on LinkedIn doesn't paginate normally. There's no "next" button, and the URL never changes as you scroll — it's infinite scroll fed by fetch requests. ScraperAPI's tutorial documents what happens when you open Chrome DevTools, switch to the Network tab, and scroll: the browser calls an internal guest endpoint, `https://www.linkedin.com/jobs-guest/jobs/api/seeMoreJobPostings/search?...`, and that endpoint returns raw HTML with all the job data. No login, no headless browser required.

Three pagination details from that research are worth memorizing, because they're the difference between a working scraper and a broken one:

1. Changing `pageNum` does nothing. Ignore it.
2. The `start` parameter increments by 25 per page. Request start=0, start=25, start=50, and so on.
3. Incrementing `start` by 1 hides previous listings instead of paging — don't do it. At the time of the tutorial, start=975 was the last working page and start=1000 returned a 404.

From there it's standard Python: `requests` and `beautifulsoup4` (install with `pip3 install requests` and `pip3 install beautifulsoup4`), send the guest-endpoint URL through your scraper of choice, parse the HTML, and pull out the fields. The selectors the tutorial uses, which you can reuse directly:

- Job title: `h3` with class `base-search-card__title`
- Company: `h4` with class `base-search-card__subtitle`
- Location: `span` with class `job-search-card__location`
- Job URL: `a` with class `base-card__full-link`

The catch: hit that guest endpoint directly from your own machine and LinkedIn will start throttling or blocking you within a few hundred requests. That's the point where routing the same request through a scraping API matters. With ScraperAPI, you prepend your target URL with `http://api.scraperapi.com?api_key={YOUR_API_KEY}&url=` and the request goes out from their rotating pool instead of your IP. Since the guest endpoint returns plain HTML without heavy JS, you often don't even need rendering — just the proxy rotation and retry logic.

## ScraperAPI plans: full comparison

Every plan includes JS rendering, premium residential and mobile IPs, CAPTCHA handling, auto-parsing, custom sessions, DataPipeline access, unlimited bandwidth, and a 99.9% uptime guarantee. Annual billing takes 10% off every plan. Here's the complete lineup from the official pricing page:

| Plan | Price/mo (annual) | Credits | Concurrent threads | Geotargeting | Get it |
| --- | --- | --- | --- | --- | --- |
| Free trial | $0 for 7 days | 5,000 | — | — | [ Start the free trial](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | $49 ($44.10) | 100,000 | 20 | US & EU | [ Get Hobby](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 ($134.10) | 1,000,000 | 50 | US & EU | [ Get Startup](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 ($269.10) | 3,000,000 | 100 | Country-level | [ Get Business](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | $475 ($427.50) | 5,000,000 | 200 | Country-level | [ Get Scaling](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 ($877.50) | 10,500,000 | 300 | Country-level | [ Get Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 ($1,777.50) | 21,500,000 | 500 | Country-level | [ Get Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | 22,000,000+ | 500+ | Custom | [ Contact sales](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

There's also a permanent free plan — 1,000 credits/month with up to 5 concurrent connections — if the trial expires and you're still in evaluation mode. Worth noting: Hobby and Startup are geographically limited to US and EU exit nodes, which matters for LinkedIn because job listings, salary data, and even company visibility vary by location. If you need to view listings as they appear in, say, Japan or Brazil, you need Business or above for country-level targeting. The LinkedIn solutions page says geotargeting works from 150+ countries "included in all plans" — the pricing page's plan-by-plan breakdown is the more precise source, so treat US/EU-only on the low tiers as the operative limit.

## The real cost per LinkedIn request (this is the part most reviews skip)

Here's the math that decides your plan, and it's buried in the docs and pricing fine print rather than the headline numbers. A standard page costs 1 credit. LinkedIn costs **30 credits per request** — more than Google's 25, and among the most expensive sites on ScraperAPI's rate card. Amazon is 5. If a request also needs the `ultra_premium` proxy pool (documented as being for "really tough targets, such as LinkedIn"), that's 30 credits on its own, or **75 credits** when combined with JavaScript rendering.

Run that against the plans:

- Hobby's 100,000 credits ≈ 3,333 plain LinkedIn requests, or ~1,333 with rendering and ultra-premium.
- Startup's 1,000,000 credits ≈ 33,000 plain requests, or ~13,300 with the full treatment.
- Business's 3,000,000 credits ≈ 100,000 plain requests, or 40,000 rendered ones.

That reframes the pricing page entirely. A third-party review from July 2026 makes the same point about protected targets generally: "Budget against your real per-page cost, not the credit count on the pricing page." A plan that looks like $0.000149 per credit on paper can effectively cost 20–30 times that on hard targets once multipliers apply.

Two mitigating features help here. First, the dashboard includes a Domain Cost Estimator that tells you the exact credit cost of any URL before you burn quota, and you can set a `max_cost` parameter to cap any single request. Second, `ultra_premium` requests are cached by default, so repeat requests for the same URL don't re-bill at full rate. Before committing to a tier, it's worth seeing the current rates side by side: 👉 [Check ScraperAPI's pricing page with the 10% annual discount applied](https://www.scraperapi.com/pricing/?fp_ref=coupons)

Another wrinkle: when credits run out, scraping pauses by default until renewal. Hobby through Professional offer opt-in auto-upgrade to the next tier; Scaling and above can enable pay-as-you-go at a fixed per-credit rate with a monthly cap you set. If your job board syncs nightly, getting cut off mid-month is worse than paying slightly more per credit — decide which behavior you want before checkout.

## When you don't need the expensive settings

Not every LinkedIn task needs `ultra_premium`. The public jobs guest endpoint is comparatively soft — it's designed to serve logged-out visitors, and routing it through ScraperAPI's standard rotating pool with clean headers gets most requests through. The premium and ultra-premium pools (10x and 30x credit multipliers, 25x/75x with rendering) are for the hard cases: profile pages, heavily rate-limited search flows, or anything where blocks keep recurring.

There's one more legitimate cost saver: `output_format=markdown` or `output_format=text`. Instead of receiving HTML you parse with BeautifulSoup, you get LLM-ready text directly — useful if your pipeline feeds a model rather than a database, and it skips the parser-maintenance tax when LinkedIn tweaks its class names.

Where ScraperAPI genuinely isn't the right tool, according to the same independent review: session persistence beyond roughly an hour (the `session_number` sticky sessions cap out around there), fixed-IP requirements for ad verification, and social account management. And against competitors on price: Bright Data's entry point sits around $499/month with no free plan, while Apify starts near $29/month. If you're only pulling a few hundred job listings a month, a free-tier-capable competitor may be enough; the value case for ScraperAPI's $49–$299 range is volume plus not wanting to maintain your own rotation logic.

## FAQ

**Is scraping LinkedIn with Python legal?** Scraping publicly available data — job listings, public company pages — without logging in is generally permissible, as the official tutorial outlines. Scraping content behind the login wall is a different legal and risk situation. Stay on the public side.

**Do I need a LinkedIn account or cookies?** Not for the guest jobs endpoint described above. That's the whole appeal of the approach: no account, no session cookies, no risk to a real profile.

**How many LinkedIn pages can I scrape on the free trial?** 5,000 credits over 7 days. At 30 credits per LinkedIn request, that's around 160 requests — enough to validate your selectors and pagination logic, not enough for a production run.

**What happens if I exceed my plan's concurrency?** Requests come back with a 429 and are refused rather than queued, at no extra charge. Drop back within your thread limit and retry.

## Which setup fits your situation

- **Learning or prototyping:** plain Python against the guest endpoint, through ScraperAPI's free trial. 👉 [Sign up and get your API key in minutes](https://www.scraperapi.com/?fp_ref=coupons)
- **A job board, lead-gen tool, or research project pulling public listings weekly:** Hobby at $49/month handles ~3,300 LinkedIn requests; Startup at $149 handles ~33,000. 👉 [Compare plan prices and the 10% annual discount](https://www.scraperapi.com/pricing/?fp_ref=coupons)
- **Country-level geotargeting or scheduled DataPipeline jobs:** Business ($299) is the first tier with country targeting and unlimited analytics history.
- **Millions of requests with webhook delivery:** Professional and up, or the custom Enterprise trial with +300 threads and up to 50M credits.

One last check before you pay: open the Domain Cost Estimator in the dashboard and run one of your actual target URLs through it. If your LinkedIn flow needs rendering plus ultra-premium at 75 credits a page, size the plan to that number — and if the monthly volume is small enough that the free plan's 1,000 credits covers it, you may not need to pay at all.
