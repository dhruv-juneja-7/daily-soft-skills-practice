## Question - 1

Over the last three weeks, daily active users are flat, but revenue is down ~12%. Leadership is concerned and wants an explanation and next steps by the end of the day.

## Answer - 1

**Clarifying Questions**

1. Which product are you referring to?
2. “Is the revenue decline uniform, or is it concentrated in specific segments like existing vs new users, platform, or geography?
3. What metrics do you use to track DAU and revenue?
4. Compared to when this renveue drip was seen and was there a recent change that happened during this timeline?

**Interviewer Responses (Simulated)**

**Product context**

This is a B2C subscription-based mobile app in the personal finance category. Users can browse content for free; premium features require a paid subscription.

**Segmentation**
The revenue decline is not uniform.

It is primarily concentrated in existing users, not new users.

The drop is more pronounced on iOS than Android.

Geography-wise, it is largest in North America.

**Metric definitions**

DAU = users who open the app at least once in a day (logged-in users only).

Revenue = subscription revenue, recognized daily (no ads, no one-time purchases).

**Timing and changes**

Revenue started declining about three weeks ago.

Around the same time, the team rolled out a pricing experiment on iOS for renewals, but DAU did not change materially.

**Step 1 Reframed Question**

> Let me restate the problem to ensure alignment. DAU has remained flat, but subscription revenue is down 12% over the last three weeks, primarily driven by existing iOS users in North America, coinciding with a pricing experiment.

**Step 2 — Decompose the Metric (Core Skill)**

Since DAU is flat, revenue decline must come from downstream drivers.

For a subscription business:

Revenue ≈

DAU
× Paid Conversion Rate
× Average Revenue per Paying User (ARPPU)
× Retention Rate (or Renewal Rate)

You should explicitly say:

“Since DAU is flat, I’d decompose revenue into conversion, pricing, and retention effects to isolate the driver.”

**Step 3 - Funnel + RCA framing**

Given the decline is concentrated among existing iOS users and coincides with a pricing experiment, renewal behavior and effective pricing are my top hypotheses.

**Answer till now (from Steps 1,2, and 3)**
To restate the problem: DAU is stable, yet revenue has declined by 12% over three weeks, mainly in existing iOS users in North America, coinciding with a pricing experiment.

Since DAU is constant, I’ll break revenue into its drivers: subscription conversion rate, renewal rate, and average price per paid user. Given the timeline and iOS focus, I would prioritize two hypotheses. First, the experiment may have reduced renewal rates for existing subscribers. Second, the pricing change might have lowered effective ARPU. My immediate next step would be to compare renewal rates pre- and post-experiment and segment ARPU changes by cohort. This will isolate if the pricing test directly impacted retention or yield, allowing us to pinpoint next steps.

**Step 4: Validation + Decision (This is what was missing)**

This is the key sentence pattern:

“To validate this, I would **_ .
If this is true, the implication would be _** .”

**Final Answer**

To restate the problem, DAU is flat but revenue is down 12%, primarily among existing iOS users in North America, coinciding with a pricing experiment.

Since DAU is unchanged, I’d decompose revenue into subscription conversion, renewal rate, and average revenue per paying user.

Given this is concentrated in existing users and aligns with a pricing change, my top hypothesis is that the experiment negatively impacted renewal rates. A secondary hypothesis is that it reduced effective ARPU.

To validate the first hypothesis, I’d compare renewal rates for users in the experiment versus control during the same period. If renewal dropped meaningfully, the decision would be to roll back or modify pricing for existing users.

If renewal is stable but ARPU declined, I’d assess whether the lower price achieved any offsetting benefits, such as longer retention or higher lifetime value.

My goal is to quantify impact and recommend whether to pause, adjust, or scale the experiment.

**Reasons for ARPU reduction**

ARPU can decline if users are paying less per subscription due to pricing or plan mix changes.

## Question - 2

“Over the past two weeks, the number of completed checkouts on our e-commerce app has dropped by 18%, but traffic to the app is unchanged. Leadership wants to know why this is happening and what to do next.”

## Answer - 2

> Calrifying Questions

- Are we seeing this across all product categories, or is it concentrated in specific categories like fashion or electronics?
- Is the decline seen for a particular segment of users like existing vs new customers or a particular geography.
- Were there any recent product, UX, or operational changes that coincided with the decline?

> Interviewer Responses

- Product category

  This is a general marketplace e-commerce app selling fashion, electronics, and home goods. No category-level changes were intentionally made.

- Segmentation

  The checkout drop is much higher for new users than existing users.

  It is more pronounced on Android than iOS.

  Geography-wise, it’s strongest in India, with other regions relatively stable.

- Recent changes

  About two weeks ago, the team redesigned the checkout flow to reduce steps.

  No price changes or promotions were launched during this period.

> Next Steps:

1. Restate the problem

2. Identify where in the funnel the drop occurs

3. List your top hypotheses

4. Say how you would validate the first hypothesis and why

> Restate the problem

Let me restate the problem to ensure alignment. Checkout completions are down 18% over the last two weeks, while traffic is unchanged. The decline is driven primarily by new users on Android in India and coincides with a checkout flow redesign.

> Anchor to the funnel

Given traffic is flat, this is a mid-to-lower funnel issue, likely within the checkout flow itself.

> State 2–3 High-Probability Hypotheses

Based on facts (not imagination):

- UX friction introduced in the redesigned checkout, disproportionately affecting new users

- Android-specific bugs or performance issues in the new flow

- Payment failures or reduced payment option visibility, especially in India
