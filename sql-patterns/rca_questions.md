### Question 1

> Why did Zomato delivery times spike in Bangalore last week?"

Here's a polished version you can use as a reference:

---

### RCA Question: Why did Zomato delivery times spike in Bangalore last week?

**Process: Clarify → Decompose → Hypothesize (ranked) → Validate → Recommend**

---

**Clarify**

Before diagnosing, I'd ask:

1. **Scale** — How much did delivery time increase? (5 mins vs 30 mins is very different) And vs. what baseline — prior week, same week last month, or other cities like Hyderabad/Pune?
2. **When** — Was this all week or specific windows? Weekday evenings vs. weekend afternoons point to very different causes
3. **Metric definition** — Are we looking at mean, median, or p95? A median spike = systemic issue; p95 spike = outlier-driven
4. **Which segment** — All of Bangalore or specific zones? All restaurant types or specific cuisines?
5. **Source** — Is this from internal ops data or customer complaints? Both together is most reliable

---

**Decompose the Metric**

Delivery Time = `Order Acceptance` + `Prep Time` + `Partner Assignment` + `Pickup Wait` + `Transit Time` + `Drop-off`

I'd pull each component separately for last week vs. the baseline period. This tells me _where_ the time was lost before I hypothesize _why_.

> e.g., If transit time is normal but partner assignment spiked → driver shortage problem, not a traffic problem

---

**Hypotheses (ranked by likelihood)**

1. 🥇 **Driver supply-demand mismatch** — A festival (Holi was last week), IPL match, or competitor incentive campaign could have pulled drivers off the platform or caused an order surge without supply keeping up
2. 🥈 **Traffic / weather spike** — Bangalore had heavy rain or a major road closure, increasing transit time across all zones
3. 🥉 **Restaurant-side bottleneck** — High-volume restaurants understaffed due to a holiday, causing prep times to spike and queue up delivery partners
4. **Zone concentration** — A new locality or event venue drove order density into a zone without enough assigned partners
5. **App / GPS issue** — Partners navigating inefficiently due to a recent app update or map data error _(less likely to cause a week-long spike but worth checking)_

---

**Validate**

| Hypothesis            | Data to Pull                                          | Signal to Look For                                      |
| --------------------- | ----------------------------------------------------- | ------------------------------------------------------- |
| Driver shortage       | Driver online hours, supply/demand ratio by zone      | Supply down >15% vs. baseline                           |
| Traffic/weather       | Transit time by zone + external weather/traffic API   | Transit time up, correlates with rain/closures          |
| Restaurant bottleneck | Prep time per restaurant, partner idle time at pickup | Specific restaurants showing 2x prep time               |
| Zone concentration    | Order heatmap by zone, partner density map            | Mismatch between order density and partner availability |
| App/GPS bug           | App version logs, route deviation data                | Spike correlating with a specific app release           |

---

**Recommend**

Once validated, the fix follows directly:

- **Driver shortage** → Surge incentives, emergency driver recruitment, demand throttling in oversaturated zones
- **Traffic/weather** → Adjust estimated delivery time (EDT) displayed to users, trigger rerouting algorithms
- **Restaurant bottleneck** → Flag underperforming partners, enforce prep-time SLAs, consider temporary order caps
- **Zone imbalance** → Rebalance driver allocation, pre-position partners in high-demand zones
- **App bug** → Hotfix + rollback, post-mortem on release process

---
