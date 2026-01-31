**Business Question-1**

For each calendar day, calculate the conversion rate from signup to purchase within 7 days.

Definitions:

A user is considered converted if they make at least one purchase within 7 days of their signup.

The conversion rate for a day is:

```
(# of users who signed up that day and converted within 7 days)
/
(total # of users who signed up that day)
```

Assumptions:

Each user signs up at most once.

Purchases can occur at any time after signup.

Ignore users who have not yet had a full 7-day window.

```
table name: user_events
columns: user_id, event_time, event_type (signup, purchase)
```

**Answer-1**

```sql
with max_window_date as (
    select max(cast(event_time as date)) - INTERVAL '7 days' as max_event_date
    from user_events
),
signup as (
    select cast(event_time as date) as event_date, user_id as signed_user_id
    from user_events
    where event_type = 'signup' and cast(event_time as date) <= (select max_event_date from max_window_date)
    group by cast(event_time as date), user_id
),
purchases as (
    select s.event_date as event_date, user_id as purchased_user_id
    from user_events u
    join signup s on u.user_id = s.signed_user_id and ( cast(event_time as date) >= s.event_date and
    cast(event_time as date) <= (s.event_date + INTERVAL '7 days'))
    where event_type = 'purchase'
    group by s.event_date, user_id
)
select s.event_date as signup_date, count(distinct signed_user_id) as signed_users, count(distinct purchased_user_id) as purchased_users,
round((count(distinct purchased_user_id)*1.0)/count(distinct signed_user_id),2) as conversion_rate
from signup s
left join purchases p on s.event_date = p.event_date and s.signed_user_id = p.purchased_user_id
group by s.event_date
order by s.event_date
```

**Summary**

> I first create a signup cohort by day, excluding users who haven’t had a full 7-day observation window. Then I join purchases back to signups to check whether each user made at least one purchase within 7 days of signup. Finally, I aggregate by signup date to calculate the daily conversion rate.

**Business Question-2**

For each day, calculate the percentage of users who returned the next day (Day-1 retention).

Definitions:

A user is active on a day if they have at least one session that day.

A user is retained on Day-1 if they are active on day D and also active on day D+1.

Retention is calculated as:

```
(# of users active on day D who are also active on day D+1)
/
(# of users active on day D)
```

Assumptions:

Multiple sessions per user per day are possible.

Use calendar days (not rolling 24-hour windows).

```
table name: sessions
columns: user_id, session_start, session_end
```

**Answer-2**

```sql

with user_day_activity as (
    select cast(session_start as date) as session_date, user_id
    from sessions
    group by cast(session_start as date), user_id
)

select u1.session_date, count(distinct u1.user_id) as active_users, count(distinct u2.user_id) as retained_users,
round(((count(distinct u2.user_id)*1.0)/count(distinct u1.user_id)),2)
from user_day_activity u1
left join user_day_activity u2 on u1.user_id = u2.user_id and u2.session_date = u1.session_date + INTERVAL '1 day'
group by u1.session_date
```

**Summary**

> “I collapse the data to one row per user per day, self-join on the next day to identify returning users, and aggregate by day to compute retention.”

**Business Question-3**

For each day, calculate the 7-day rolling average revenue per active user (ARPU).

Definitions:

A user is active on a day if they placed at least one order in the last 7 days (including that day).

Revenue includes all orders in the 7-day window.

```
ARPU is defined as:

(total revenue in the 7-day window)
/
(# of distinct active users in the same 7-day window)
```

Assumptions:

Users can place multiple orders per day.

Use calendar days.

If there are zero active users, ARPU should be NULL.

````
table name = orders
column names = order_date, order_id, user_id, order_amount

**Answer-3**

```sql
with reporting_date as (
    select order_date as reporting_date
    from orders
    group by order_date
),
rolling_orders as (
    select r.reporting_date,
    o.user_id,
    o.order_amount
    from reporting_date r
    left join orders o on o.order_date between (r.reporting_date - INTERVAL '6 days') and r.reporting_date
)
select reporting_date,
count(distinct user_id) as active_users,
sum(order_amount) as total_revenue,
round((sum(order_amount)*1.0)/count(distinct user_id),2) as rolling_7_day_ARPU
from rolling_orders
group by reporting_date
order by reporting_date
````

**Summary**

> I constructed a day-level table where each row represents a reporting day, joined to all the orders in its 7-day lookback window. Then I calculated revenue by summing all the order amounts but users are counted using COUNT(DISTINCT user_id) within the 7-day window. Finally I divide revenue by active users to compute roliing ARPU.

**Business Question - 4**

For each day, calculate the conversion rate from view → purchase within the same day.

Definitions:

A user is counted as viewed on a day if they have at least one view event that day.

A user is counted as converted if they have at least one purchase event on the same day, regardless of order.

Conversion rate:

(# of users with both view and purchase on that day)
/
(# of users with view on that day)

Assumptions:

Users can have multiple events per day.

Events can occur in any order within the day.

Ignore users who only purchased without viewing.

```
table name: events
columns: user_id, event_name (view, add_to_cart, purchase), event_time
```

**Answer - 4**

```sql
with user_events as (
    select cast(event_time as date) as event_date, user_id,
    case(when max(event_name) = 'view' then 1 else 0 end) as view_flag,
    case(when max(event_name) = 'purchase' then 1 else 0 end) as purchase_flag
    from events
    group by cast(event_time as date), user_id
)
select event_date,
count(distinct (case when view_flag = 1 then user_id end)) as total_viewed_users,
count(distinct (case when view_flag = 1 and purchase_flag = 1 then user_id end)) as total_purchased_users,
round((count(distinct (case when view_flag = 1 and purchase_flag = 1 then user_id end))*1.0)/count(distinct (case when view_flag = 1 and purchase_flag = 1 then user_id end)),2) as same_day_conversion_rate
from user_events
group by event_date
order by event_date

```

**Summary**

> “I first collapse the data to one row per user per day, flagging whether the user had a view and whether they had a purchase that day. Then, for each day, I count users with a view as the denominator and users with both view and purchase as the numerator to compute the conversion rate.

**Business Question - 5**

For each day, calculate the number of active subscribers.

Definitions:

A user is active on a day if:

start_date ≤ day, and

(end_date is NULL or end_date ≥ day)

Count distinct users.

Use calendar days.

Assumptions:

A user can have multiple subscription records over time.

Subscriptions do not overlap for the same user.

You may assume the data spans multiple months.

```
table_name: subscriptions
columns: user_id, start_date, end_date, plan_type
```

**Answer - 5**

with calendar_table as (
select generate_series (
(SELECT min(start_date) from subscriptions),
(SELECT MAX(COALESCE(end_date, CURRENT_DATE)) from subscriptions),
INTERVAL '1 day'
) as reporting_date
)
select reporting_date, count(distinct user_id) as active_users
from calendar_table c
left join subscriptions s on c.reporting_date >= start_date and (end_date >= c.reporting_date or end_date is null)
group by reporting_date

**Summary**

> I’d create a calendar table covering the full date range. Then I’d join each calendar day to subscriptions where the day falls between the subscription’s start and end dates, treating NULL end dates as still active. Finally, I’d count distinct users per day to get active subscribers.

**Business Question - 6**

Find the top 10 employees by average daily working hours for a given month (say, September 2025).

Definitions & Assumptions

A working day starts with a punch_in and ends with a punch_out.

An employee can have multiple punch-in / punch-out pairs in a day (e.g., breaks).

Working hours for a day = sum of all (punch_out − punch_in) intervals for that day.

```
Average daily working hours =

(total working hours in the month)
/
(number of days the employee worked in that month)
```

Ignore incomplete pairs (e.g., punch_in without a punch_out).

Use calendar days based on event_timestamp.

**Answer - 5**

```sql
with arranged_data as (
select employee_id,
event_timestamp as in_time,
event_type as in_type,
lead(event_timestamp) over(partition by employee_id order by event_timestamp) as out_time,
lead(event_type) over(partition by employee_id order by event_timestamp) as out_type
from employee_attendance
where cast(event_timestamp as date) between '2025-09-01' and '2025-09-30'
),
hours as (
select employee_id, sum(EXTRACT(EPOCH FROM (out_time - in_time))/3600.0) as total_hours_spent, cast(in_time as date) as in_date
from arranged_data
where out_type = 'punch_out' and in_type = 'punch_in'
group by employee_id, cast(in_time as date))
select employee_id, avg(total_hours_spent) as avg_hours_spent
from hours
group by employee_id
order by avg_hours_spent desc
limit 10
```

**Summary**
I first filter events to the target month and order them by timestamp per employee. Then I use LEAD to pair each punch-in with the next event and keep only valid punch-in to punch-out pairs. I sum these intervals to get total working hours per employee per day. Finally, I average daily hours per employee for the month and rank them to get the top 10.

**ALTERNATIVE APPROACH**

```sql
WITH in_times AS (
    SELECT
        employee_id,
        event_timestamp AS in_time,
        LEAD(event_timestamp) OVER (
            PARTITION BY employee_id
            ORDER BY event_timestamp
        ) AS next_in_time,
        CAST(event_timestamp AS DATE) AS in_date
    FROM employee_attendance
    WHERE event_type = 'punch_in'
      AND event_timestamp >= '2025-09-01'
      AND event_timestamp <  '2025-10-01'
),
out_times AS (
    SELECT
        employee_id,
        event_timestamp AS out_time,
        CAST(event_timestamp AS DATE) AS out_date
    FROM employee_attendance
    WHERE event_type = 'punch_out'
      AND event_timestamp >= '2025-09-01'
      AND event_timestamp <  '2025-10-01'
),
matched_intervals AS (
    SELECT
        i.employee_id,
        i.in_date,
        i.in_time,
        MIN(o.out_time) AS out_time
    FROM in_times i
    LEFT JOIN out_times o
      ON i.employee_id = o.employee_id
     AND o.out_time > i.in_time
     AND (i.next_in_time IS NULL OR o.out_time < i.next_in_time)
    GROUP BY
        i.employee_id,
        i.in_date,
        i.in_time
),
daily_hours AS (
    SELECT
        employee_id,
        in_date,
        SUM(EXTRACT(EPOCH FROM (out_time - in_time)) / 3600.0) AS daily_hours
    FROM matched_intervals
    WHERE out_time IS NOT NULL
    GROUP BY employee_id, in_date
)
SELECT
    employee_id,
    AVG(daily_hours) AS avg_daily_hours
FROM daily_hours
GROUP BY employee_id
ORDER BY avg_daily_hours DESC
LIMIT 10;
```
