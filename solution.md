Расчет rolling retention
WITH cohort_users AS (
    SELECT 
        id AS user_id,
        DATE_TRUNC('month', date_joined) AS cohort_month,
        date_joined::date AS registration_date
    FROM users
),
user_activity AS (
    SELECT DISTINCT
        ue.user_id,
        ue.entry_at::date AS activity_date
    FROM userentry ue
),
cohort_activity AS (
    SELECT 
        cu.user_id,
        cu.cohort_month,
        cu.registration_date,
        ua.activity_date,
        (ua.activity_date - cu.registration_date) AS days_since_registration
    FROM cohort_users cu
    LEFT JOIN user_activity ua ON cu.user_id = ua.user_id
),
retention_windows AS (
    SELECT 
        ca.user_id,
        ca.cohort_month,
        ca.registration_date,
        MAX(CASE WHEN ca.days_since_registration >= 0 THEN 1 ELSE 0 END) AS active_day_0,
        MAX(CASE WHEN ca.days_since_registration >= 1 THEN 1 ELSE 0 END) AS active_day_1,
        MAX(CASE WHEN ca.days_since_registration >= 3 THEN 1 ELSE 0 END) AS active_day_3,
        MAX(CASE WHEN ca.days_since_registration >= 7 THEN 1 ELSE 0 END) AS active_day_7,
        MAX(CASE WHEN ca.days_since_registration >= 14 THEN 1 ELSE 0 END) AS active_day_14,
        MAX(CASE WHEN ca.days_since_registration >= 30 THEN 1 ELSE 0 END) AS active_day_30,
        MAX(CASE WHEN ca.days_since_registration >= 60 THEN 1 ELSE 0 END) AS active_day_60,
        MAX(CASE WHEN ca.days_since_registration >= 90 THEN 1 ELSE 0 END) AS active_day_90
    FROM cohort_activity ca
    GROUP BY ca.user_id, ca.cohort_month, ca.registration_date
)
SELECT 
    TO_CHAR(cohort_month, 'YYYY-MM') AS cohort,
    COUNT(DISTINCT user_id) AS total_users,
    ROUND(100.0 * SUM(active_day_0) / COUNT(DISTINCT user_id), 2) AS day_0,
    ROUND(100.0 * SUM(active_day_1) / COUNT(DISTINCT user_id), 2) AS day_1,
    ROUND(100.0 * SUM(active_day_3) / COUNT(DISTINCT user_id), 2) AS day_3,
    ROUND(100.0 * SUM(active_day_7) / COUNT(DISTINCT user_id), 2) AS day_7,
    ROUND(100.0 * SUM(active_day_14) / COUNT(DISTINCT user_id), 2) AS day_14,
    ROUND(100.0 * SUM(active_day_30) / COUNT(DISTINCT user_id), 2) AS day_30,
    ROUND(100.0 * SUM(active_day_60) / COUNT(DISTINCT user_id), 2) AS day_60,
    ROUND(100.0 * SUM(active_day_90) / COUNT(DISTINCT user_id), 2) AS day_90
FROM retention_windows
GROUP BY cohort_month
ORDER BY cohort_month;


