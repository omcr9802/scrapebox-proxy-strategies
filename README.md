# scrapebox proxies: how to choose rotating vs sticky IPs so Google harvesting stops getting blocked

Most people searching for ScrapeBox proxies aren't shopping for a provider yet. They've already hit the wall: the harvester runs for ten minutes, Google starts throwing CAPTCHAs, and the free proxy list they spent an hour testing has maybe forty live entries left, half of which fail the next morning.

The provider question matters, but it comes second. What actually decides whether your harvest runs clean is the *type* of IP sitting behind the proxy entry, and whether that type matches the ScrapeBox module you're running. Get that wrong and you'll blame the provider for something no provider can fix.

## What ScrapeBox actually asks a proxy to do

ScrapeBox isn't one tool. It's a pile of modules that share a proxy connection, and they pull in opposite directions.

- **URL harvesting** fires thousands of small, independent requests at search engines. Each one needs a different IP, or at least a rapidly cycling one. Data per request is tiny; request volume is enormous.
- **PageRank lookups, indexed page checks, email scraping** behave the same way. Lots of cheap, disposable requests.
- **Comment posting, trackback and form submission** are session work. If the IP changes halfway through a submission, the request dies or gets flagged. These modules want the same IP across a whole sequence.
- **Keyword harvesting and add-ons** usually fall somewhere between the two.

That split is the whole game. ScrapeBox's own proxy page has been describing it for years in datacenter terms: "exclusive" proxies harvest more URLs from Google and handle more PageRank lookups because you're the only tenant, while "shared" proxies are "more suited towards posting, and scraping data from sites with no strict query limits." Same logic, different vocabulary. Residential rotating IPs are simply a third category that page never covered, because when it was written, residential networks weren't priced for bulk SEO work.

Worth knowing before you buy anything: ScrapeBox ships with its own proxy harvester and tester. It comes with 22 built-in sources that publish daily proxy lists, and you can add your own. The tester handles private proxies that need a username and password, can check whether a proxy actually returns Google results, filters by country, port and speed, supports a custom test URL with a custom success string, and lets you keep only Google-passed or anonymous entries. The Automator plugin can even loop the whole thing around the clock, saving fresh working proxies to a file.

So proxies are optional in ScrapeBox. What's not optional is IP diversity the moment you point it at a search engine.

## The decision that actually changes your results: rotating vs sticky

ScrapeBox's proxy manager expects a list of entries in `IP:Port` or `IP:Port:Username:Password` format. It rotates that list itself according to your Usage Time setting, which is how long it may keep using one entry before switching to the next.

A rotating backconnect endpoint works differently. It's a single entry that swaps the exit IP behind the scenes, often on every request. To ScrapeBox, it looks like one proxy. To the target site, it looks like thousands of different visitors.

Mixing these up is where most setup disasters come from.

| ScrapeBox job | IP behaviour you want | Why |
| --- | --- | --- |
| Google/SERP URL harvesting | Rotate on every request | One IP sending hundreds of queries gets blocked within minutes |
| Keyword scraping, indexed page checks | Rotate, or short sticky windows | Requests are independent; speed matters more than continuity |
| Comment posting, form submission | Sticky for the whole sequence | A mid-session IP swap kills the submission or flags it |
| Account-based tasks, multi-account work | One dedicated IP per account | Sessions must stay clean and separated |
| Long overnight harvests | Large rotating pool with auto reload | IPs drop out; the run shouldn't stop when they do |

Practical rule: if the module completes a task in a single request, rotate. If it completes a task across several requests, stay sticky.

## The tester gotcha that makes people think their proxies are dead

Here's the part that wastes the most support tickets. ScrapeBox's built-in proxy tester returns false negatives on backconnect and rotating endpoints. Decodo's own ScrapeBox setup guide warns about it directly: don't check proxy status inside the Proxy Editor, because ScrapeBox doesn't support checks on backconnect proxies, and you'll get negative results from proxies that work fine.

So when you paste a rotating endpoint in and every entry shows as failed, that usually isn't the provider's fault. It's the tester asking for a static connection and getting a moving target.

Three ways to verify honestly instead:

1. Run a real job and watch Harvester Status. If it reads "Proxies Enabled" and URLs are coming back, the proxy is working.
2. Use the tester's custom test option with a URL and a unique string from the response page, which sidesteps the connection model entirely.
3. Test static endpoints. ScrapeBox's tester is genuinely useful for a list of individual IP:Port entries.

One more thing while you're in the proxy editor: if you've pasted SOCKS5 addresses, select them all, hit Modify, and choose "Mark all Proxies as Non-Socks proxies." You should then see an "N" in the S column for each entry. Skip this and connections fail in ways that look random.

## How 9Proxy's two models map onto ScrapeBox work

9Proxy sells residential IPs two ways, and the split lines up unusually well with the rotating-vs-sticky problem above.

**Residential by GB** is the pay-per-traffic model. You buy a GB balance, generate unlimited endpoints from the dashboard, and pay only for data consumed. Authentication is username/password or IP whitelisting. No app install required. Sessions are controlled entirely through the username string, which is where it gets useful for ScrapeBox:

`subaccount-country-us-city-newyork-sst-15-ssid-device1`

Leave out `sst` and `ssid` and you get rotating IPs, one fresh exit per request. Add `sst-15` and the IP holds for fifteen minutes. Add `ssid` and you can spin up multiple sticky IPs from one configuration, which matters when you're running parallel tasks that must not share an IP. Country, state, city, ZIP and ISP targeting are all embedded the same way. GB balances last 180 days on standard plans, with no expiry at all on Enterprise.

**Residential by IPs** is the pay-per-IP model, and it's the one that produces plain `IP:Port` entries ScrapeBox understands natively. You get unlimited bandwidth while an IP is active, IPs you bought never expire, and each forwarded IP stays live anywhere from a few hours to roughly 24 hours depending on the IP. The catch is setup: this model requires the 9Proxy desktop app (Windows or Mac, with an equivalent CLI on Linux), which forwards selected IPs to local ports like `127.0.0.1:60000`. You then paste those local ports into ScrapeBox.

There's also a Today List in the app that reuses proxies you've already forwarded in the last 24 hours. The documentation is explicit that forwarding from the Today List doesn't consume new IPs, which is a real cost saver on repeat harvests that need the same geography rather than fresh IPs.

Both models cover 20 million-plus residential IPs across 90+ locations, and both handle HTTP(S) and SOCKS5.

## Setting it up in ScrapeBox, step by step

**Using the GB-based plan (recommended for harvesting)**

1. In the 9Proxy dashboard, open Residential Proxies → GB → Proxy Generator.
2. Set your authentication method, targeting, and session mode. Rotating for harvesting, sticky (`sst`) for posting.
3. Generate your endpoints and copy the host and port.
4. Open ScrapeBox, find Select Harvester and Proxies, tick Use Proxies, click Edit.
5. Paste your entries in `host:port:username:password` format. Build the username with your targeting suffix, use your sub-user password as the password.
6. Click Save so the list is locked in, then close the editor.
7. Set your timeout. 30–60 seconds is the usual range; check your provider's response times before going higher.
8. Turn on Auto Reload so ScrapeBox recovers the list during long sessions.
9. Confirm Harvester Status reports "Proxies Enabled" before you start.

**Using the IP-based plan (for sticky, IP-per-task work)**

1. Install the 9Proxy app and import the IPs you need, filtering by country, state, city or ISP.
2. Forward each IP to a port in your configured range. On Linux the CLI shortcut is `9proxy proxy -c US -p 60000`, which binds a US residential IP to local port 60000.
3. Copy the resulting list of `127.0.0.1:port` entries into ScrapeBox's proxy editor, adding proxy authentication if you enabled it.
4. Mark entries as non-SOCKS if you're forwarding SOCKS5 ports.
5. Re-forward from the Today List for repeat sessions instead of spending new IPs.

Keep thread counts realistic. These are real residential connections with consumer-grade uptime, not idle servers in a datacenter. If your old datacenter list ran happily at 200 threads, expect to dial that down and let IP diversity do the work instead of raw concurrency.

## Where each 9Proxy plan fits a ScrapeBox workflow

**One-off harvest or a first test.** The 5 GB pack at $15. SERP pages are light compared to media or product pages, so a GB-based balance stretches further on plain harvesting than people expect, and you're not committing to a subscription to find out whether a target is scrapable.

**Regular SEO work, a few tools.** The 50 + 5 GB pack at $105, or 100 GB at $150. Enough room for keyword harvesting, rank checks and the occasional email scrape without topping up mid-month.

**Comment posting, account work, anything session-based.** Straight to the IP-based packages. 100 IPs at $24 gives you a clean pool of individual ports, and because IPs never expire, you can re-forward from the Today List later instead of buying again. If you're posting at any volume, 500 IPs at $72 is the tier where the per-IP cost drops meaningfully.

**Agency work with mixed jobs.** The bundles exist for exactly this. 1,500 IPs plus 50 GB for $180 covers session work and volume harvesting under one purchase, and the traffic side is valid for 180 days so uneven project schedules don't burn balance.

**Teams.** Enterprise GB packages remove the 180-day expiry and add team mode: one owner plus up to five members, shared bandwidth with no expiration, per-member traffic controls and activity logs. If several people are running ScrapeBox, this beats buying separate balances.

## Current 9Proxy pricing, all packages

One thing to know before you compare these numbers against older reviews: 9Proxy adjusted pricing for its IP-based and bundle packages on 1 June 2026, and confirmed that GB-based prices were left unchanged. Reviews from before that date still show the old IP rates. Everything below is the post-adjustment structure.

**Residential by IPs** (pay per IP, unlimited bandwidth while active, IPs never expire)

| Package | Total | Effective per IP | Purchase |
| --- | --- | --- | --- |
| 100 IPs | $24 | $0.24 | 查看 100 IP 套餐 — no, English. → |

Apologies — let me render that properly:

| Package | Total | Effective per IP | Purchase |
| --- | --- | --- | --- |
| 100 IPs | $24 | $0.24 | Start with the 100 IP package |
| 500 IPs | $72 | $0.144 | Compare the 500 IP package |
| 1,000 IPs + 500 bonus | $126 | $0.084 | See the 1,500 IP package |
| 2,500 IPs | $210 | $0.084 | Check the 2,500 IP package |
| 5,000 IPs | $360 | $0.072 | View the 5,000 IP package |
| 15,000 IPs | $720 | $0.048 | Look at the 15,000 IP package |
| 25,000 IPs | $863 | $0.035 | Review the 25,000 IP package |
| 50,000 IPs | $1,438 | $0.029 | See the 50,000 IP package |

**Business IP packages** (high-volume tiers)

| Package | Total | Effective per IP | Purchase |
| --- | --- | --- | --- |
| 100,000 IPs | $2,300 | $0.023 | Get pricing on the 100,000 IP package |
| 200,000 IPs | $4,140 | $0.021 | Get pricing on the 200,000 IP package |
| 500,000 IPs | $8,625 | $0.018 | Get pricing on the 500,000 IP package |

**Residential by GB** (pay per GB, unlimited endpoints, rotating or sticky)

| Package | Total | Effective per GB | Validity | Purchase |
| --- | --- | --- | --- | --- |
| 5 GB | $15 | $3.00 | 180 days | Grab the 5 GB pack and test one harvest |
| 50 GB + 5 GB bonus | $105 | $2.10 | 180 days | Pick up the 50 GB pack |
| 100 GB | $150 | $1.50 | 180 days | Take the 100 GB pack |
| 200 GB | $200 | $1.00 | 180 days | Move to the 200 GB pack |
| 1,000 GB | $800 | $0.80 | 180 days | Scale on the 1,000 GB pack |
| 2,000 GB | $1,500 | $0.75 | 180 days | Check the 2,000 GB pack |

**Enterprise GB packages** (no expiry on balance)

| Package | Total | Effective per GB | Validity | Purchase |
| --- | --- | --- | --- | --- |
| 3,000 GB | $2,160 | $0.72 | No expiry | See the 3,000 GB Enterprise pack |
| 6,000 GB | $4,200 | $0.70 | No expiry | See the 6,000 GB Enterprise pack |
| 10,000 GB | $6,800 | $0.68 | No expiry | See the 10,000 GB Enterprise pack |

**Bundles** (IPs plus bandwidth in one purchase)

| Package | Total | What's included | Purchase |
| --- | --- | --- | --- |
| Starter bundle | $30 | 100 IPs + 5 GB | Try the Starter bundle |
| Popular bundle | $180 | 1,500 IPs + 50 GB | Compare the Popular bundle |
| Pro bundle | $720 | 5,000 IPs + 500 GB | Check the Pro bundle |

A couple of notes on reading that. There's no monthly subscription anywhere in this model; you buy a balance and draw it down. The headline figures you'll see in 9Proxy's marketing, $0.015 per IP and $0.68 per GB, are the floor rates that only apply at the largest tiers, not the entry prices. Enterprise and reseller arrangements are quoted separately. And bundles follow the same 180-day rule for traffic while the IP portion doesn't expire.

## What's worth knowing before you pay

No self-serve free trial. Reviews vary on this, but the consistent picture is that 9Proxy doesn't run an open trial on the site; availability depends on promotions and you generally have to ask support. Plan on buying a small package to evaluate it. There is a 60-second policy covering proxies that fail right at activation, which refunds or replaces a dead IP, but it isn't a satisfaction guarantee.

The IP-based model requires the desktop app. One reviewer lists this as a downside for multi-device workflows compared to browser extensions. The app is the reason a single forwarded IP stays clean and countable, but if you work across machines, the dashboard-only GB model is simpler.

The transparent Proxy Program feature, which routes an application's traffic without the app knowing a proxy exists, is Windows-only and doesn't yet work on x86 or ARM builds.

Residential IPs churn by design. A ScrapeBox workflow that assumes the same IP will be there tomorrow will break. This is the trade you're making: residential IPs get blocked far less often than datacenter ranges on search engines, but they're less stable, so Auto Reload and a healthy pool size matter more than they did with your old datacenter list.

Finally, honest framing on the category. If your ScrapeBox usage is mostly comment posting against sites with no strict query limits, a cheap shared datacenter plan may still be the better buy. Residential bandwidth is priced for situations where IP reputation is the bottleneck, which in practice means search engine harvesting, SERP checks and anything hitting aggressive anti-bot systems.

## Quick answers

**Does ScrapeBox need paid proxies?** No. It runs without them and includes a harvester with 22 proxy sources. But public proxies get blocked by Google quickly, and the time you spend testing them is usually worth more than a small GB package.

**Can I point comment posting at one rotating endpoint?** Not reliably. Use sticky sessions or forwarded IPs so a single submission completes from a single IP.

**Why does ScrapeBox's tester show every rotating proxy as failed?** Because the tester checks backconnect endpoints as if they were static connections. Verify by running a job or using a custom test URL instead.

**How many proxies do I need for harvesting?** Fewer than you'd think if they rotate quickly, more than you'd like if each one is fixed for a session. Thread count and IP diversity per minute are the numbers that matter.

**Do unused IPs expire?** On the IP-based model, no. On GB plans, 180 days on standard tiers and never on Enterprise.

## The short version

Match the IP behaviour to the ScrapeBox module and most of your blocking problems disappear. Rotating residential for harvesting, sticky or dedicated IPs for posting, and a pool big enough that individual IPs dropping out doesn't stall the run.

For that split, 9Proxy's GB model handles the rotating half with per-request IP changes, city and ISP targeting, and no app install, while the IP-based model gives you countable, never-expiring IPs for the session work. 👉 Open the 9Proxy sign-up and pick the smallest package that covers your current ScrapeBox workload — you can scale into bundles or Enterprise later once you know your actual GB burn per harvest.
