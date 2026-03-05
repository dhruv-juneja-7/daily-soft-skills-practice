### Question - Increasing Streak

![alt text](Increasing_Streak.jpeg)

```sql
with lagged_amount as (
    select user_id, date, amount as curr_amount,
    lag(amount) over(partition by user_id order by date) as prev_amount
    from users
),
indicators as (
    select user_id, date, curr_amount, prev_amount,
    case when prev_amount is null or prev_amount < curr_amount then 0 else 1 end as ind
    from lagged_amount
),
groups as (
    select user_id, date, curr_amount, prev_amount, ind, sum(ind) over(partition by user_id order by date) as groups
    from indicators
),
streaks as (
    select user_id, count(groups) as streak_count
    from groups
    group by user_id, groups
)
select distinct user_id
from streaks
where streak_count >= 3
```

### Question

> Write an SQL query to display the records [in the Stadium table] with three or more rows with consecutive id’s, and the number of people is greater than or equal to 100 for each. Return the result table ordered by visit_date in ascending order.

```sql
with lagged_table as (
    select id, visit_date, people, lag(id) over(order by id) as lag_id,
    case when lag(id) over(order by id) is null or lag(id) over(order by id) + 1 = id then 0 else 1 end as ind
    from stadium
    where people >= 100
),
groupings as (
    select id, visit_date, people, sum(ind) over(order by id) as groups
    from lagged_table
),
grouped_data as (
    select groups, count(*) as count_all
    from groupings
    group by groups
    having count(*) >= 3
)
select id, visit_date, people
from groupings l
join grouped_data g on l.groups = g.groups
order by visit_date
```

```sql
with ranks as (
    select emp_id, work_date, row_number() over(partition by emp_id order by work_date) as rn,
    work_date - row_number() over(partition by emp_id order by work_date) as group_id
    from attendance
)
select emp_id, min(work_date) as start_date, max(work_date) as end_date, count(*) as streak_length
from ranks
group by emp_id, group_id
```

## LC 1454 Active Users

```sql
with rankings as (
    select id, login_date, row_number() over (partition by id order by login_date) as rn
    from logins
),
groupings as (
    select id, login_date, login_date - rn * Interval '1 day' as groups
    from rankings
),
streaks as (
    select id, min(login_date) as streak_start_date, max (login_date) as streak_end_date, count(*) as streaks
    from groupings
    group by id, groups
    having count(*) >= 5
)
select distinct a.id, a.name
from accounts a
join streaks s on a.id = s.id
```

## Suspicious Raid Transactions

> Correct but not optimised

```sql
with intervals as (
    select a.user_id, a.txn_time as start_time,
    b.txn_time, a.amount
    from transactions a
    join transactions b
    on a.user_id = b.user_id and (b.txn_time >= a.txn_time and b.txn_time <= a.txn_time + INTERVAL '10 minutes')
)
select user_id, start_time, max(txn_time) as end_time, count(*) as txn_count
from intervals
group by user_id, start_time
having count(*) >= 3

```

> Optimised using window functions

```sql
with burst_counts as (
    select user_id, txn_time,
    count(*) over (
        partition by user_id
        order by txn_time
        range between current row and interval '10 minutes' following
    ) as bursts,
    max(txn_time) over(
        partition by user_id
        order by txn_time
        range between current row and interval '10 minutes' following
    ) as end_time
    from transactions
)
select user_id, txn_time as start_time, end_time, bursts
from burst_counts
where bursts >= 3
order by user_id, start_time
```

### LC 1225 Report Contiguous Dates

```sql
with combined as (
    select 'failed' as period_state, date
    from failed
    union all
    select 'succeeded' as period_state, date
    from succeeded
),
markings as (
    select period_state, date,
    row_number() over (order by date) as rn,
    date - row_number() over (order by date) * INTERVAL '1 day' as streak_start_date
    from combined
),
groupings as (
    select period_state, date,
    concat(left(period_state, 1), streak_start_date) as groups
    from markings
)
select period_state, min(date) as start_date, max(date) as end_date
from groupings
group by period_state, groups
order by start_date
```

### Lesson

> In the above query the row_number will continue event if the period state changes, making two failed states with a gap a part of the same group.

> So, we have to use a partition by because we want the row_number (our local clock) to stop when the state is not the same while the actual dates ( the master clock) continues. This will treat two same period_states as part of different groups.

### LC 180 Consecutive Numbers.

```sql
with lag_table as (
    select id, num, lag(num) over(order by id) as lag_num
    from logs
),
indicators as (
    select id, num, lag_num,
    case when lag_num is null or num - lag_num = 0 then 0 else 1 end as ind
    from lag_table
),
groups as (
    select id, num, ind, sum(ind) over(order by id) as group_id
    from indicators
)
select distinct num
from groups
group by num, group_id
having count(*) >= 3
```

## learnings Day 4

> Report Contiguous Dates : Islands question. For this my main lesson was to understand the ;ogic using two clock method where the actual date is the master cl;ock and the row_number is the local clock. The local clock should reset on each partition while the master clock should move continuously

> Active Users : Island question. For this my main lesson was to think which cte are at each step and how will rach row look in them and write the column names in each cte beforehand

> Consecutive numbers : Island problem. I solved it using my lag value, indicator and rolling sum to find group id approach. Lesson here if the consecutive value is 3 or 4 then use lead, lag instead of making groups to make the query faster.
