## 1454 Active Users

```sql
with group_id as (
    select id, login_date, login_date - row_number() over(partition by id order by login_date) as rn
    from logins
),
grouped_data as (
    select distinct id
    from group_id
    group by id, rn
    having count(*) >= 5
)
select a.id, a.name
from grouped_data g
join accounts a on g.id = a.id
order by id
```

## 1225 Report Contiguous dates

with combined_data as (
select 'failed' as type , date
from failed
union all
select 'succeeded' as type, date
from succeeded
),
groups as (
select type, date, date - row_number() over(partition by type order by date) as group_id
from combined_data
)
select type, min(date) as start_date, max(date) as end_date
from groups
group by type, group_id
order by start_date

## 180 Consecutive numbers

with lead_lag as (
select id, num, lead(num) over(order by id) as next_num,
lag(num) over(order by id) as prev_num
from logs
)
select distinct num
from lead_lag
where num = prev_num and num = next_num

## Attendance Query

> Question: Find each employee's consecutive attendance streaks — show the start date, end date, and length of each streak.

```sql
with groupings as (
    select emp_id, work_date, work_date - row_number() over(partition by emp_id order by work_date) as group_id
    from attendance
)
select emp_id, min(work_date) as start_date, max(work_date) as end_date, count(*) as streak_length
from groupings
group by emp_id, group_id
order by emp_id, start_date
```
