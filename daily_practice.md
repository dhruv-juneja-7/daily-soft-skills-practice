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

```

**Summary**

> I constructed a day-level table where each row represents a reporting day, joined to all the orders in its 7-day lookback window. Then I calculated revenue by summing all the order amounts but users are counted using COUNT(DISTINCT user_id) within the 7-day window. Finally I divide revenue by active users to compute roliing ARPU.
