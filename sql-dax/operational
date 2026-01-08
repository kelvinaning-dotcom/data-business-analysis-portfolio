/* ============================================================
   Operational Performance Analytics
   Table: operational_cases
   Purpose: Workload, backlog, cycle time, and closure metrics
   Dialect: PostgreSQL
   ============================================================ */

-- 1) Basic counts: total and closed cases
SELECT
  COUNT(*) AS total_cases,
  SUM(CASE WHEN status = 'Closed' THEN 1 ELSE 0 END) AS closed_cases,
  ROUND(
    SUM(CASE WHEN status = 'Closed' THEN 1 ELSE 0 END)::numeric / NULLIF(COUNT(*), 0) * 100, 2
  ) AS closure_rate_pct
FROM operational_cases;

-- 2) Average cycle time (days) for closed cases
SELECT
  ROUND(AVG((closed_date::date - created_date::date)), 2) AS avg_cycle_time_days
FROM operational_cases
WHERE status = 'Closed'
  AND closed_date IS NOT NULL;

-- 3) Monthly workload trend (cases created per month)
SELECT
  DATE_TRUNC('month', created_date)::date AS month_start,
  COUNT(*) AS cases_created
FROM operational_cases
GROUP BY 1
ORDER BY 1;

-- 4) Monthly closures trend (cases closed per month)
SELECT
  DATE_TRUNC('month', closed_date)::date AS month_start,
  COUNT(*) AS cases_closed
FROM operational_cases
WHERE status = 'Closed'
  AND closed_date IS NOT NULL
GROUP BY 1
ORDER BY 1;

-- 5) Backlog snapshot (open cases) by category and priority
SELECT
  category,
  priority,
  COUNT(*) AS open_cases
FROM operational_cases
WHERE status = 'Open'
GROUP BY category, priority
ORDER BY open_cases DESC;


-- 6) Cycle time by category (identifies bottlenecks)
SELECT
  category,
  COUNT(*) AS closed_cases,
  ROUND(AVG((closed_date::date - created_date::date)), 2) AS avg_cycle_time_days,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY (closed_date::date - created_date::date)) AS median_cycle_time_days,
  MAX((closed_date::date - created_date::date)) AS max_cycle_time_days
FROM operational_cases
WHERE status = 'Closed'
  AND closed_date IS NOT NULL
GROUP BY category
ORDER BY avg_cycle_time_days DESC;


-- 7) Cycle time by priority (does "High" actually move faster?)
SELECT
  priority,
  COUNT(*) AS closed_cases,
  ROUND(AVG((closed_date::date - created_date::date)), 2) AS avg_cycle_time_days
FROM operational_cases
WHERE status = 'Closed'
  AND closed_date IS NOT NULL
GROUP BY priority
ORDER BY
  CASE priority
    WHEN 'High' THEN 1
    WHEN 'Medium' THEN 2
    WHEN 'Low' THEN 3
    ELSE 4
  END;

-- 8) SLA compliance example (e.g., SLA = 7 days for High priority)
-- Adjust thresholds as needed.
SELECT
  priority,
  COUNT(*) AS closed_cases,
  SUM(CASE
        WHEN priority = 'High'   AND (closed_date::date - created_date::date) <= 7  THEN 1
        WHEN priority = 'Medium' AND (closed_date::date - created_date::date) <= 10 THEN 1
        WHEN priority = 'Low'    AND (closed_date::date - created_date::date) <= 14 THEN 1
        ELSE 0
      END) AS within_sla,
  ROUND(
    SUM(CASE
          WHEN priority = 'High'   AND (closed_date::date - created_date::date) <= 7  THEN 1
          WHEN priority = 'Medium' AND (closed_date::date - created_date::date) <= 10 THEN 1
          WHEN priority = 'Low'    AND (closed_date::date - created_date::date) <= 14 THEN 1
          ELSE 0
        END)::numeric
    / NULLIF(COUNT(*), 0) * 100, 2
  ) AS sla_compliance_pct
FROM operational_cases
WHERE status = 'Closed'
  AND closed_date IS NOT NULL
GROUP BY priority
ORDER BY sla_compliance_pct DESC;

-- 9) Workload by analyst/team (capacity signal)
SELECT
  analyst,
  COUNT(*) AS total_cases,
  SUM(CASE WHEN status = 'Closed' THEN 1 ELSE 0 END) AS closed_cases,
  SUM(CASE WHEN status = 'Open' THEN 1 ELSE 0 END) AS open_cases
FROM operational_cases
GROUP BY analyst
ORDER BY total_cases DESC;
