# Amazon Product Scraper API: What 1,000 Products Cost on ScraperAPI, Which Plan Covers Your Volume, and What the JSON Actually Returns (Full Plan Table)

ScraperAPI charges 5 credits for every request to amazon.com or any other Amazon domain. The Hobby plan sells 100,000 credits for $49 a month, which works out to 20,000 Amazon product pages, or about **$2.45 per 1,000 products**. Under 200 products a month, the free plan's 1,000 credits covers it at no cost.

Below: the full plan lineup, the credit math at 10,000 and 100,000 products a month, the fields the Amazon Product endpoint returns, and the two places where the credit model catches people out.

> **Quick pick:** Hobby at $49/month is where most small teams end up. 100,000 credits covers 20,000 Amazon pages, enough for a daily price-and-buybox sweep of a few hundred ASINs with room to spare. If you're pulling a few hundred products a month, the free tier is honestly enough to start. 👉 [Compare ScraperAPI plans and current pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## How ScraperAPI counts an Amazon request

Billing is credit-based, and the number of credits depends on the domain you're hitting, not on how many fields come back. Amazon sits in the middle of the price ladder.

| Target | Credits per request |
| --- | --- |
| Standard page | 1 |
| **Amazon (all domains)** | **5** |
| Google and Bing, including subdomains | 25 |
| LinkedIn | 30 |
| Bot-protected sites (Cloudflare, DataDome, PerimeterX) | +10 on top of base cost, charged when ScraperAPI runs its bypass logic |

Two things follow from this table. First, a single Amazon product page costs the same as five ordinary pages, so any budget you've estimated for general web scraping will be five times too optimistic once you point it at Amazon. Second, the +10 surcharge doesn't apply to Amazon by default, but if you mix Amazon with retailer sites behind Cloudflare, those pages can cost 11 credits each, more than double an Amazon request.

One caveat worth checking before you size a plan: 5 credits is the documented rate for Amazon domains generally, and ScraperAPI points to the Domain Cost Estimator inside your dashboard as the authoritative per-URL figure. Structured data endpoints can be priced differently from a plain page fetch, so run one ASIN through the estimator before you commit to a tier.

## Every ScraperAPI plan in one table

ScraperAPI publishes eight tiers plus a free plan. Where a credit allowance or price isn't in the public documentation I could verify, the table sends you to the pricing page rather than guessing at a number.

| Plan | Monthly credits | Price | What changes at this tier | Get it |
| --- | --- | --- | --- | --- |
| Free | 1,000 | $0 | 5 concurrent connections | [Start on the free plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Hobby | 100,000 | $49/mo | Auto-upgrade available when credits run out | [Hobby plan pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | On the pricing page | On the pricing page | Auto-upgrade available | [Startup plan pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | On the pricing page | On the pricing page | Auto-upgrade available | [Business plan pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | 5,000,000 | On the pricing page | Pay-as-you-go plus auto-upgrade | [Scaling plan pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | On the pricing page | On the pricing page | Pay-as-you-go | [Professional plan pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | On the pricing page | On the pricing page | Pay-as-you-go | [Advanced plan pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | Custom quote | Volume negotiation with sales | [See how Enterprise is quoted](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

Two structural differences matter more than the credit counts. Pay-as-you-go, the option that keeps requests flowing past your monthly allowance at a fixed per-credit rate up to a cap you set, exists only on Scaling, Professional, Advanced and Enterprise. On Hobby, Startup and Business, running out of credits returns a 429 rather than an extra charge. Annual billing takes exactly 10% off the monthly price, and credits reset at each renewal date with nothing carried forward.

## The real cost at 10,000 and 100,000 products

Multiply your monthly product count by 5 to get credits, then match against the allowances above.

| Amazon pages per month | Credits needed | Plan that covers it | What you actually pay |
| --- | --- | --- | --- |
| 200 | 1,000 | Free | $0 |
| 5,000 | 25,000 | Hobby | $49 |
| 10,000 | 50,000 | Hobby | $49 (half the allowance goes unused) |
| 20,000 | 100,000 | Hobby | $49 |
| 100,000 | 500,000 | Scaling | Listed on the pricing page |
| 1,000,000 | 5,000,000 | Scaling | Listed on the pricing page |

The 10,000 row is the one to sit with. You can't buy half a plan, so a 10,000-product workload and a 20,000-product workload cost the same $49. If your volume sits between tiers, you're paying for headroom you aren't using, which is a fine reason to pause at Hobby and only move up when the 20,000-page ceiling is genuinely in sight.

At 100,000 products a month the math is five Hobby allowances, so it needs Scaling at 5,000,000 credits, which covers a million Amazon pages. ScraperAPI states that larger plans generally come with a better price per credit, so the effective rate at that volume is lower than the Hobby rate, but the exact figure lives on the pricing page and in the estimator, not in the docs.

## What the Amazon Product endpoint returns

The sync product endpoint takes an ASIN and returns structured JSON rather than raw HTML. The documented example splits into a top level and a `product_information` object.

Top level: `name`, `brand`, `brand_url`, and `full_description`, which carries the long-form copy from the listing including brand-story and A+ content.

Inside `product_information` you get catalogue specs as flat key-value pairs: `brand`, `manufacturer`, `model_name`, `item_model_number`, `asin`, `color`, `material`, `capacity`, `product_dimensions`, `item_weight`, `voltage`, `wattage`, `power_source`, `special_feature`, `included_components`, `product_care_instructions`, `date_first_available`, and `best_sellers_rank`.

`best_sellers_rank` comes back as an array of strings in Amazon's own phrasing, for example `"#14,536 in Kitchen & Dining (See Top 100 in Kitchen & Dining)"` alongside `"#49 in Rice Cookers"`. That means two ranks in different categories for one ASIN, and you'll need to parse the category and number out of each string rather than expecting a clean object.

The async version, posted to `https://async.scraperapi.com/structured/amazon/product`, takes `apiKey`, `asin`, `country_code`, `tld` and an optional webhook `callback`. It adds two things the sync endpoint's documentation doesn't advertise: variant details (size, color and other options where they exist), and **all publicly available reviews** for the ASIN. That second point is a real differentiator. Apify's alfalfa Amazon actor, by contrast, states plainly that full review lists have required an Amazon login since late 2024, so it returns only the 8 to 13 reviews visible on the product page plus Amazon's "Customers say" summary.

The `country_code` and `tld` parameters are how you reach marketplaces outside the US. Confirm the exact list of supported TLDs against the endpoint reference before you build a multi-market pipeline around it.

One practical note: the sample response published in the docs is cut short, and buybox and seller-level fields don't appear in the portion that is shown. If winning-seller data or offer lists are the point of your project, open the endpoint reference and fire one test call with your own ASIN before you assume the schema covers it.

## Where the credit model catches people out

**Credits expire monthly.** Unused credits are reset at renewal, never carried forward. Overbuying "just in case" is a straight loss.

**Concurrency limits return 429, not a queue.** Exceed your plan's concurrent connections and the request is turned away with no extra charge. The free plan caps at 5 concurrent connections; paid plan limits are listed on the pricing page. A scraper that fires 50 parallel requests on a plan that allows 10 will lose most of them, and the retry logic you write becomes part of your architecture.

**Auto-upgrade is opt-in.** If you want the account to move up a tier automatically at exhaustion instead of stalling, turn it on. It's available on Hobby, Startup, Business, Scaling and Professional.

**`max_cost` exists.** You can cap the credits any single request is allowed to consume, which stops one pathological URL from eating a chunk of your allowance.

## How ScraperAPI compares with Oxylabs, Bright Data and Apify

All figures below are the vendors' own published rates. Nothing here has been independently benchmarked.

| Provider and product | Pricing unit | Published rate | Notes |
| --- | --- | --- | --- |
| **ScraperAPI**, Amazon domains | Credits | 5 credits/request; Hobby works out to about $2.45 per 1,000 Amazon pages | Structured product endpoint, async version returns all publicly available reviews |
| **Oxylabs** Web API, Amazon endpoints | Per successful result | From $0.50 per 1,000 Amazon results | Product, search, pricing, sellers and bestsellers endpoints; you pay only for delivered results; each extra search page is a separate fetch and charge |
| **Apify**, alfalfa/amazon-product-scraper | Per 1,000 results | $0.28 listing fields; +$1 per 1,000 to open each product page ($1.28 total) | 22 storefronts; BSR, variants, A+ content; top 8 to 13 reviews only |
| **Apify**, muffin/amazon-product-scraper | Per result | Not published on the page reviewed | 10 marketplaces, 25+ fields, 2 to 5 seconds per product |
| **Bright Data**, Amazon scraper | Credits | 5,000 free credits per month for new accounts | Sync requests time out at 1 minute and return HTTP 202 with a `snapshot_id`; batch work belongs on the async trigger endpoint |

On raw cost per page, ScraperAPI's Hobby rate is roughly five times Oxylabs' published starting rate and about double Apify's listing-plus-details price. What that money buys is a single credit pool covering proxy rotation, JavaScript rendering, CAPTCHA handling and parsing, plus the async endpoint's full review coverage. If you're pulling plain product pages at volume and parsing them yourself, the cheaper per-page options win on arithmetic. If you want reviews and variant data without building a second pipeline, the gap narrows.

## Which plan fits your workload

Run your monthly product count through this and stop reading.

- **Under 200 Amazon pages a month.** Free plan. 1,000 credits, 5 concurrent connections, no card.
- **200 to 20,000 pages.** Hobby at $49. This covers everything from a small competitor-tracking script to a daily sweep of several thousand ASINs.
- **20,000 to 1,000,000 pages.** Scaling, with pay-as-you-go switched on so a launch week doesn't stall your pipeline. Since PAYG isn't available below this tier, spiky workloads on Hobby are better served by auto-upgrade than by manual plan changes.
- **Above a million pages, or procurement involved.** Enterprise, quoted per volume.

## FAQ

**Does ScraperAPI have a free tier?** Yes. 1,000 credits on signup, capped at 5 concurrent connections, which is about 200 Amazon product pages.

**Do unused credits roll over?** No. They reset at each renewal date.

**Can I get the full review list for an ASIN?** The async Amazon Product endpoint is documented as returning all publicly available reviews for the product page. Verify against your own ASIN, since what Amazon makes public varies by listing.

**What happens when I hit my credit limit?** On Hobby, Startup and Business you get a 429 and scraping pauses until renewal unless auto-upgrade is on. On Scaling, Professional, Advanced and Enterprise, pay-as-you-go continues at a fixed per-credit rate up to the cap you set.

**Is annual billing cheaper?** By exactly 10% versus monthly.

## Before you buy

Two things to confirm on your own account, both of which change the numbers above: run a single ASIN through the Domain Cost Estimator to see what the structured endpoint charges on your plan, and check that the endpoint's response schema includes every field your pipeline depends on, especially seller and buybox data. Ten minutes with the free tier's 1,000 credits answers both questions without spending anything.

If the answer is "about 20,000 pages a month or less," Hobby at $49 is the whole decision. 👉 [Check the current ScraperAPI plan prices and start with the free credits](https://www.scraperapi.com/pricing/?fp_ref=coupons)
