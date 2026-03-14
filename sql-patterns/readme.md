# SQL Patterns — Advanced Query Techniques for Product Analytics

A structured collection of SQL mental models, solved problems, and documented learnings built during a deliberate transition into product analytics.

Each solution is paired with the thinking process behind it — not just the answer, but the pattern that makes the answer reusable across different problems.

---

## Why This Repository Exists

Most SQL practice focuses on getting the right answer. This repository focuses on recognising the right pattern — so that any new problem can be classified and solved in under 10 minutes without starting from scratch.

This is the approach used by senior analysts in product companies: pattern recognition first, implementation second.

---

## The 5-Pattern Library

Every problem in this repository maps to one of five core patterns:

| Pattern | When to Use | Key Technique |
|---------|-------------|---------------|
| **Gaps & Islands V1** | Consecutive dates or sequential IDs | `date - ROW_NUMBER()` |
| **Gaps & Islands V2** | Repeating values — same number or status consecutive | `rn_overall - rn_per_num` |
| **Running Sum** | Something starts and ends — find state at any moment | `SUM(value) OVER (ORDER BY time)` |
| **Window Ranking** | Top N per group, leaderboards, percentiles | `DENSE_RANK() OVER (PARTITION BY group)` |
| **Conditional Aggregation** | Rates, ratios, percentages per segment | `SUM(CASE WHEN condition THEN 1 ELSE 0 END) / COUNT(*)` |

---

## Problems Solved

### 1. Employee Attendance Streaks
**Business Question:** How long are employees maintaining consistent attendance? Where are the gaps?

**Pattern:** Gaps & Islands V1

**Real-world application:** User engagement tracking — find how long users maintain a daily habit in an app (Duolingo streaks, fitness app check-ins, e-learning platforms).

```sql
WITH groupings AS (
    SELECT emp_id, work_date,
           work_date - ROW_NUMBER() OVER (PARTITION BY emp_id ORDER BY work_date) AS group_id
    FROM attendance
)
SELECT emp_id,
       MIN(work_date) AS streak_start,
       MAX(work_date) AS streak_end,
       COUNT(*)       AS streak_length
FROM groupings
GROUP BY emp_id, group_id
ORDER BY emp_id, streak_start;
```

**Mental model:** Two clocks — master clock (date) moves continuously, local clock (row_number) resets per partition. Constant difference = same streak. Shifted difference = new streak.

---

### 2. Active Users — 5 Consecutive Login Days
**Business Question:** Which users are genuinely engaged vs. occasional visitors?

**Pattern:** Gaps & Islands V1 with deduplication

**Real-world application:** Defining "active user" cohorts for retention analysis. Identifying power users for referral programs.

```sql
WITH deduped AS (
    SELECT DISTINCT user_id, login_date FROM logins
),
groupings AS (
    SELECT user_id, login_date,
           login_date - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) AS group_id
    FROM deduped
),
streaks AS (
    SELECT user_id
    FROM groupings
    GROUP BY user_id, group_id
    HAVING COUNT(*) >= 5
)
SELECT DISTINCT a.id, a.name
FROM accounts a
JOIN streaks s ON a.id = s.user_id;
```

**Key insight:** Always deduplicate before applying ROW_NUMBER — duplicate dates on the same user break the group_id calculation silently.

---

### 3. Report Contiguous Date Ranges
**Business Question:** What were the continuous periods of system success vs. failure?

**Pattern:** Gaps & Islands V1 with UNION ALL

**Real-world application:** SLA reporting, system uptime tracking, campaign active periods.

```sql
WITH combined AS (
    SELECT 'failed'    AS report_type, fail_date    AS report_date FROM failed
    UNION ALL
    SELECT 'succeeded' AS report_type, success_date AS report_date FROM succeeded
),
groupings AS (
    SELECT report_type, report_date,
           report_date - ROW_NUMBER() OVER (PARTITION BY report_type ORDER BY report_date) AS group_id
    FROM combined
)
SELECT report_type,
       MIN(report_date) AS start_date,
       MAX(report_date) AS end_date
FROM groupings
GROUP BY report_type, group_id
ORDER BY start_date;
```

**Key insight:** UNION ALL first to combine two tables into one event stream, then apply the standard skeleton. PARTITION BY the type column so each status type forms its own islands.

---

### 4. Consecutive Numbers
**Business Question:** Find values that repeat consecutively — detect patterns in sequential data.

**Pattern:** LAG/LEAD (small fixed N) or Gaps & Islands V2 (variable N)

**Real-world application:** Fraud detection — find transactions with the same amount repeated consecutively. Quality control — detect consecutive failed readings.

```sql
-- LAG/LEAD approach (cleaner for exactly 3 consecutive)
WITH neighbors AS (
    SELECT num,
           LAG(num)  OVER (ORDER BY id) AS prev_num,
           LEAD(num) OVER (ORDER BY id) AS next_num
    FROM logs
)
SELECT DISTINCT num
FROM neighbors
WHERE num = prev_num AND num = next_num;
```

**Decision rule:** Use LAG/LEAD when consecutive count is small and fixed (3, 4). Use Gaps & Islands V2 when consecutive count is variable or large.

---

### 5. Last Person to Fit in Bus (Weight Limit Problem)
**Business Question:** At what point does a cumulative threshold get crossed?

**Pattern:** Running Sum — cutoff variant

**Real-world application:** Budget allocation — last project that fits within quarterly budget. Inventory — last item before warehouse capacity is reached. Feature rollout — last user cohort before server load limit.

```sql
WITH cumulative AS (
    SELECT person_name,
           SUM(weight) OVER (ORDER BY turn) AS cumulative_weight
    FROM queue
)
SELECT person_name
FROM cumulative
WHERE cumulative_weight <= 1000
ORDER BY cumulative_weight DESC
LIMIT 1;
```

**Mental model:** Running sum = current state at every moment. Filter for all valid states. The last valid state is the answer.

---

### 6. Human Traffic of Stadium
**Business Question:** Which consecutive periods had sustained high engagement?

**Pattern:** Gaps & Islands V1 — ID based with pre-filter

**Real-world application:** Identifying sustained high-traffic periods on a platform. Finding peak engagement windows for notification timing. Detecting consistent high-usage periods for capacity planning.

```sql
WITH groupings AS (
    SELECT id, visit_date, people,
           id - ROW_NUMBER() OVER (ORDER BY id) AS group_id
    FROM stadium
    WHERE people >= 100
),
streaks AS (
    SELECT group_id
    FROM groupings
    GROUP BY group_id
    HAVING COUNT(*) >= 3
)
SELECT g.id, g.visit_date, g.people
FROM groupings g
JOIN streaks s ON g.group_id = s.group_id
ORDER BY visit_date;
```

**Key insight:** Filter BEFORE creating groups — only consecutive high-attendance days matter. Use `id - rn` not `date - rn` because consecutiveness is defined by row sequence, not calendar dates.

---

## Key Mental Models Documented

### The Two Clock Model
Every Gaps & Islands problem has two clocks:
- **Master clock** — moves continuously (date, overall row number)
- **Local clock** — resets per partition (row number per group)

When both clocks tick together → same island. When one skips → new island.

**Version 1** (date/id based): Master clock exists in the data. Only build the local clock.
**Version 2** (repeating values): Neither clock exists. Build both from scratch using `rn_overall - rn_per_num`.

### The Law of Constant Differences
If the difference between two clocks is constant across consecutive rows — they belong to the same group. The moment the difference shifts, a new group starts.

### The Pattern Classification Decision Tree
```
Does something START and END?
→ Yes → Running Sum

Are rows CONSECUTIVE or forming STREAKS?
→ Sequential data exists (date/id) → Gaps & Islands V1
→ Repeating values → Gaps & Islands V2

Need TOP N per GROUP?
→ Window Ranking (DENSE_RANK)

Need RATE, RATIO, PERCENTAGE?
→ Conditional Aggregation
```

### Think in Rows, Not Logic
Before writing any query, sketch the exact column names and sample row values at each CTE stage on paper. Gaps in logic appear on paper — not halfway through writing SQL.

---

## Common Error Patterns (and how to avoid them)

| Error | Example | Fix |
|-------|---------|-----|
| CTE name typo | `from grouings` | 60-second line-by-line review before running |
| Missing GROUP BY column | `GROUP BY report_type` without `group_id` | Every non-aggregated column must be in GROUP BY |
| Wrong table in final SELECT | `FROM attendance` instead of `FROM groupings` | Final SELECT always queries the last CTE |
| Using `date - rn` when value repeats | LC 180 Consecutive Numbers | Ask: is the value sequential or repeating? |
| Duplicate dates breaking ROW_NUMBER | LC 1454 Active Users | Always DISTINCT before ROW_NUMBER on date columns |

---

## Tools Used
- **Platform:** LeetCode, db-fiddle.com
- **Database:** PostgreSQL / MySQL
- **Reference:** binhhoang.io/blog/gaps-and-islands, datalemur.com

---

## About
Built as part of a structured transition from CRM analytics into product analytics. Problems selected to mirror real interview questions at product companies (Zomato, Razorpay, MakeMyTrip, PolicyBazaar).

*This repository is updated weekly as new patterns are learned.*
