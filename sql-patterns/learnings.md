# Week 1

## learnings Day 4

> LC 1225: Report Contiguous Dates : Islands question. For this my main lesson was to understand the logic using two clock method where the actual date is the master clock and the row_number is the local clock. The local clock should reset on each partition while the master clock should move continuously

> LC 1454: Active Users : Island question. For this my main lesson was to think which cte are at each step and how will rach row look in them and write the column names in each cte beforehand

> Consecutive numbers : Island problem. I solved it using my lag value, indicator and rolling sum to find group id approach. Lesson here if the consecutive value is 3 or 4 then use lead, lag instead of making groups to make the query faster.

## Learnings Day 5

> "Consecutive Numbers — LAG/LEAD approach. Find middle row where num = lag AND num = lead. Filter with WHERE not GROUP BY. **GROUP BY destroys consecutiveness.**"

## Learnings Day 6

Gaps and Islands two versions:

```sql

Version 1: value - rn
-> Use when dates, sequential integers like ids that keep on increasing
-> Example: attendance_streaks, contiguous date ranges

Version 2: rn_overall - rn_per_num

-> Use when: repeating values, status codes, same number consecutive
-> Example: LC 180 Consecutive Numbers, seat numbers, server downtime
```

> "Gaps and Islands is always about two clocks. When both clocks tick together the difference is constant — same island. When one clock skips the difference shifts — new island. Version 1 uses date and rn as the two clocks. Version 2 uses rn_overall and rn_per_num as the two clocks."

And remember in version 1 the master clock existed and we just have to create the local clock but in version 2 we have to create both the clocks

## Learnings Day 7

> One drill to add from next week: after writing each query, read it line by line top to bottom before running it. Give yourself 60 seconds of review. Catch your own errors before the interpreter does. In an interview this habit alone saves you from looking shaky on problems you actually know.

# Week 2

## Learnings Day 10

```
LC 1204 — Last Person to Fit in Bus
Pattern: Running Sum
Skeleton: SUM(value) OVER (ORDER BY sequence) AS cumulative
Find cutoff: WHERE cumulative <= limit, ORDER BY cumulative DESC, LIMIT 1
Key insight: Running sum accumulates state row by row.
Filter finds everything within limit.
Last row of that filtered set is the answer.
```

## Learnings Day 11

```
-- Step 1: Join lookup tables to get filter flags
-- Step 2: Filter on those flags
-- Step 3: Conditional aggregation for the metric
```

This pattern appears constantly in product company interviews — calculating rates, percentages, and ratios after filtering specific user segments.

---

## Add to learnings.md

```
LC 262 — Trips and Users
Pattern: Multi-join filtering + Conditional Aggregation
Skeleton:
  CTE: Join lookup table twice with different aliases to get flags
  Final: WHERE on flags, ROUND(SUM(CASE WHEN)/COUNT(*), 2)
Key insight: When same table is a lookup for two different
foreign keys, join it twice with different aliases.
Typo watch: be consistent with column names across CTEs.
```
