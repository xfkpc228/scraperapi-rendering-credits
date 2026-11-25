# JavaScript rendering API: what one render costs, which ScraperAPI plan covers your volume, and why rendered pages come back partial (plans and credits compared)

A JavaScript rendering API loads a page in a real browser, lets the scripts run, and hands you back the finished HTML instead of the empty shell. At ScraperAPI you switch it on with `render=true`, and each rendered request costs **10 API credits** against the 1 credit a standard page costs. Two decisions follow from that number: how many credits your monthly page volume actually burns, and what to check in the response before you trust what came back. Both are covered below, along with the limits that catch people out after they have already paid.

> Most readers fit **Hobby**: 100,000 credits a month is about 10,000 rendered pages at 10 credits each, with 20 concurrent threads. Move up when your rendered volume crosses that, or when you need country-level geotargeting, which starts at Business. The free tier gives you **1,000 credits and 5 concurrent connections** to test with.
> 👉 [See current ScraperAPI plans and credit allowances](https://www.scraperapi.com/pricing?fp_ref=coupons)

## All ScraperAPI plans, with rendered-page capacity

Rendered pages are calculated at 10 credits per request on a standard page. Sites with their own credit cost (Amazon, Google, LinkedIn, anything behind bot protection) consume more, so treat the third column as a ceiling for ordinary pages rather than a promise.

| Plan | Credits/month | Rendered pages/month | Threads | Price | Sign up |
| --- | --- | --- | --- | --- | --- |
| Hobby | 100,000 | ~10,000 | 20 | See pricing page | [Get Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | 1,000,000 | ~100,000 | 50 | See pricing page | [Get Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 | ~300,000 | 100 | See pricing page | [Get Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | 5,000,000 | ~500,000 | 200 | **$475/month**, or $427.50/month billed annually ($5,130/year) | [Get Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | 10,500,000 | ~1,050,000 | 300 | See pricing page | [Get Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 | ~2,150,000 | 500 | See pricing page | [Get Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 22,000,000+ | ~2,200,000+ | 500+ | Custom | [Get Enterprise](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Two more parameters decide which of those rows you can actually use, and neither is in the main table:

| Plan | Geotargeting | Analytics history | What happens when credits run out |
| --- | --- | --- | --- |
| Hobby | US & EU | 30 days | Auto-upgrade to next plan |
| Startup | US & EU | 30 days | Auto-upgrade to next plan |
| Business | Country-level | Unlimited | Auto-upgrade to next plan |
| Scaling | Country-level | Unlimited | Auto-upgrade or pay-as-you-go |
| Professional | Country-level | Unlimited | Auto-upgrade or pay-as-you-go |
| Advanced | Country-level | Unlimited | Pay-as-you-go |
| Enterprise | Country-level | Unlimited | Pay-as-you-go |

Geotargeting is the line that matters more than price for a lot of teams. Hobby and Startup cap you at US and EU exit points. If your scraping depends on seeing a page as a visitor from Brazil, Japan or Germany, your cheapest usable plan is Business regardless of how few pages you render.

## What happens inside one render request

ScraperAPI routes the URL through a Chromium instance, executes the page's JavaScript and returns the resulting HTML. That is the whole surface of a basic call. The interesting part is everything around it.

An HTML body on its own doesn't tell you whether the render succeeded. Well-built rendering services based on Playwright publish a request and response contract you can use as a checklist when you size up any vendor, ScraperAPI included:

- **On the request side:** the target URL (percent-encode it when it carries query strings or a hash), a user agent string, an optional ad-block switch, a wait window with a minimum and maximum, and a set of headers to forward to the target by name.
- **On the response side:** `statusCode`, `headers`, `body`, `metrics`, `resolvedUrl` (the redirect the renderer caught), `timeout`, and `error`.

You are not buying HTML. You are buying the metadata that lets your pipeline distinguish a good page from a half-rendered one.

### The two fields that decide whether to trust a result

`timeout: true` means the page hit your time limit. Rendering continues, but the engine squeezes every remaining timeout down to a minimum, so late-loading content is likely missing. `error` being non-null means the payload you received is **partial**, even if the body looks plausible.

Any API that returns only an HTML string, with no separate signal for timeout, partial payload or redirect target, forces you to guess. Ask before you buy:

1. Does the response expose the final URL after redirects, separate from the URL you sent?
2. Can the API distinguish a timeout from a hard error from a soft 404 page?
3. Can you forward custom headers to the target site?
4. Can you bound the wait time with a range instead of a single fixed delay?
5. Can it reach and hold a login-gated page? Services that support this typically fill the login form's text and password inputs, submit, and return the session cookie plus the post-login URL and body. ScraperAPI's session support is listed as free, with no extra credit cost.

## The credit maths that decides your plan

JS rendering is the expensive switch. A standard page costs 1 credit; a rendered one costs 10. Combining it with premium proxies pushes that to **25 credits**, and with ultra premium proxies to **75 credits**. Premium proxies on their own are 10, and a screenshot is 10.

Some domains carry their own cost on top of the request type: Amazon, Walmart and eBay at 5 credits, Google and Bing at 25, LinkedIn at 30, and bypassing Cloudflare, DataDome, PerimeterX/HUMAN, Akamai or Cloudflare Turnstile adds 10. Every response reports what the request actually cost in the `sa-credit-cost` header, you can price any URL in advance with the Domain Cost Estimator in the dashboard, and `max_cost` caps what a single scrape is allowed to spend.

At the verified Scaling price of $475/month for 5,000,000 credits, a plain rendered page works out at roughly **$0.00095**. At 75 credits for ultra premium plus rendering, the same 5,000,000 credits cover about 66,000 pages instead of 500,000. That is the same plan and the same bill, with an 8x difference in output, which is why checking the per-domain cost before you commit beats estimating afterwards.

👉 [Compare the plans that fit your rendered page volume](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Which plan you actually need

- **Under 10,000 rendered pages a month, US/EU targeting is enough:** Hobby. It is the plan most people should start on, and 20 threads is more than a single-page-at-a-time pipeline needs.
- **10,000 to 100,000 rendered pages:** Startup. Note that this is the tier people skip past, then get moved into anyway by auto-upgrade when Hobby's credits run dry mid-month.
- **You need country-level geotargeting at any volume:** Business, on geotargeting alone.
- **300,000 to 500,000 rendered pages:** Scaling at $475/month. Annual billing drops it to $427.50/month and saves $570 a year, which is worth taking only if you already know your volume is stable.
- **Over a million rendered pages:** Professional or Advanced. Enterprise exists for 22M+ credits and custom terms.
- **You render occasionally and scrape mostly static pages:** stay on the lowest tier you can and let the 1-credit standard requests carry the bulk of the work.

## The concurrency limit that isn't on the pricing table

Plan threads run from 20 to 500+, but rendered requests have a separate ceiling: a **burst limit of 10 render starts per second by default**. If each rendered request takes 25 seconds, 10 starts per second means up to 250 renders in flight at once, which is well above what the smaller plans sell you in threads. The burst limit is the one that bites during backfills, and raising it is an Enterprise conversation.

## When a JavaScript rendering API is the wrong purchase

If the data is in the HTML source, you are paying 10x for nothing. Fetch the page normally and parse it.

ScraperAPI's own documentation recommends using rendering only when you need it, and states plainly that it increases latency and can reduce success rates, which lowers the number of requests you get through the API. That is an honest description of every headless-browser pipeline, not a weakness specific to this vendor.

A one-off export of a few hundred pages doesn't need a subscription either. A local headless browser on your own machine handles that for free. The API earns its money when you need volume, retries, proxy rotation and bot-protection bypass in the same request.

Rendering also skips things by default to stay fast: images, tracking scripts and other unnecessary resources are not loaded. Occasionally the data you want sits behind one of those deferred scripts. That is what `wait_for_selector` is for, and it only works when `render=true` is in the same request. Without it, the parameter is ignored.

## FAQ

**Does `render=true` cost more than a normal request?**
Yes. 10 credits against 1 for a standard page. With premium proxies it is 25, with ultra premium proxies 75.

**Can I render only part of my traffic?**
Yes, it is a per-request parameter. Route only the URLs that need JavaScript through it and keep the rest at 1 credit.

**How many rendered requests can I start per second?**
10 by default. Higher burst limits are available on Enterprise.

**Can I click, type or scroll before the page is captured?**
Yes. The JavaScript Rendering Instruction Set covers click, input, scroll and waiting for events before the response is returned.

**What happens if I hit my credit limit mid-month?**
Requests pause until credits renew. On Hobby through Professional you can enable auto-upgrade to move to the next plan automatically. On Scaling and above, pay-as-you-go keeps requests running at a fixed per-credit rate up to a monthly cap you set. Annual plans can also renew early for a fresh batch of credits.

## Before you check out

Match the plan to your rendered page count first, then check two things that change the answer. Confirm the geotargeting tier you need, because Hobby and Startup stop at US and EU. And price your actual target domains in the Domain Cost Estimator before you pick a credit level, since a rendered request to a protected or premium domain costs several times the 10-credit baseline and can turn a comfortable plan into a tight one in a week.

Start on the smallest plan that covers your monthly volume. The 1,000 free credits and 5 concurrent connections are enough to run your real URLs through the API, read the `sa-credit-cost` header on each response, and see what a month of your workload would actually cost.

👉 [Start with ScraperAPI's free credits and pick a plan](https://www.scraperapi.com/pricing?fp_ref=coupons)
