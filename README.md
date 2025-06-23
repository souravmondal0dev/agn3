-- Task 1: Consecutive Project Days
WITH project_groups AS (
    SELECT *,
           DATEADD(DAY, -ROW_NUMBER() OVER (ORDER BY Start_Date), Start_Date) AS grp
    FROM Projects
)
SELECT MIN(Start_Date) AS Start_Date, MAX(End_Date) AS End_Date
FROM project_groups
GROUP BY grp
ORDER BY End_Date DESC;

-- Task 2: Students whose best friend earns more
SELECT s.Name
FROM Students s
JOIN Friends f ON s.ID = f.ID
JOIN Packages sp ON s.ID = sp.ID
JOIN Packages fp ON f.Friend_ID = fp.ID
WHERE sp.Salary < fp.Salary
ORDER BY fp.Salary;

-- Task 3: Symmetric Pairs
SELECT DISTINCT f1.X, f1.Y
FROM Functions f1
JOIN Functions f2 ON f1.X = f2.Y AND f1.Y = f2.X
WHERE f1.X <= f1.Y
ORDER BY f1.X;

-- Task 4: Contest Summary
SELECT c.contest_id, c.hacker_id, c.name,
       SUM(ss.total_submissions),
       SUM(ss.total_accepted_submissions),
       SUM(vs.total_views),
       SUM(vs.total_unique_views)
FROM Contests c
JOIN Colleges col ON c.contest_id = col.contest_id
JOIN Challenges ch ON col.college_id = ch.college_id
LEFT JOIN View_Stats vs ON ch.challenge_id = vs.challenge_id
LEFT JOIN Submission_Stats ss ON ch.challenge_id = ss.challenge_id
GROUP BY c.contest_id, c.hacker_id, c.name
HAVING SUM(ss.total_submissions) + SUM(ss.total_accepted_submissions) + SUM(vs.total_views) + SUM(vs.total_unique_views) > 0
ORDER BY c.contest_id;

-- Task 5: Daily Submissions Stats
WITH days AS (
    SELECT DATE '2016-03-01' + INTERVAL n DAY AS submission_date
    FROM generate_series(0, 14) n
),
daily_subs AS (
    SELECT submission_date, hacker_id, COUNT(*) AS sub_count
    FROM Submissions
    GROUP BY submission_date, hacker_id
),
top_hacker AS (
    SELECT submission_date, hacker_id, RANK() OVER (PARTITION BY submission_date ORDER BY COUNT(*) DESC, hacker_id ASC) AS rnk
    FROM Submissions
    GROUP BY submission_date, hacker_id
)
SELECT d.submission_date, 
       COUNT(DISTINCT s.hacker_id) AS total_hackers,
       h.hacker_id, hkr.name
FROM days d
JOIN Submissions s ON d.submission_date = s.submission_date
JOIN top_hacker h ON d.submission_date = h.submission_date AND h.rnk = 1
JOIN Hackers hkr ON h.hacker_id = hkr.hacker_id
GROUP BY d.submission_date, h.hacker_id, hkr.name
ORDER BY d.submission_date;

-- Task 6: Manhattan Distance
SELECT ROUND(ABS(MAX(LAT_N)-MIN(LAT_N)) + ABS(MAX(LONG_W)-MIN(LONG_W)), 4) AS ManhattanDistance
FROM STATION;

-- Task 7: Prime Numbers <= 1000
WITH RECURSIVE numbers AS (
    SELECT 2 AS num
    UNION ALL
    SELECT num + 1 FROM numbers WHERE num < 1000
),
primes AS (
    SELECT num FROM numbers n
    WHERE NOT EXISTS (
        SELECT 1 FROM numbers d
        WHERE d.num < n.num AND n.num % d.num = 0 AND d.num > 1
    )
)
SELECT STRING_AGG(CAST(num AS VARCHAR), '&')
FROM primes;

-- Task 8: Pivot Occupations
SELECT
    MAX(CASE WHEN Occupation = 'Doctor' THEN Name END) AS Doctor,
    MAX(CASE WHEN Occupation = 'Professor' THEN Name END) AS Professor,
    MAX(CASE WHEN Occupation = 'Singer' THEN Name END) AS Singer,
    MAX(CASE WHEN Occupation = 'Actor' THEN Name END) AS Actor
FROM (
    SELECT Name, Occupation, ROW_NUMBER() OVER (PARTITION BY Occupation ORDER BY Name) AS rn
    FROM OCCUPATIONS
) AS pivoted
GROUP BY rn;

-- Task 9: Node Types in BST
SELECT N,
       CASE
           WHEN P IS NULL THEN 'Root'
           WHEN N NOT IN (SELECT DISTINCT P FROM BST WHERE P IS NOT NULL) THEN 'Leaf'
           ELSE 'Inner'
       END AS NodeType
FROM BST
ORDER BY N;

-- Task 10: Company Hierarchy
SELECT c.company_code, c.founder,
       COUNT(DISTINCT lm.lead_manager_code),
       COUNT(DISTINCT sm.senior_manager_code),
       COUNT(DISTINCT m.manager_code),
       COUNT(DISTINCT e.employee_code)
FROM Company c
LEFT JOIN Lead_Manager lm ON c.company_code = lm.company_code
LEFT JOIN Senior_Manager sm ON c.company_code = sm.company_code
LEFT JOIN Manager m ON c.company_code = m.company_code
LEFT JOIN Employee e ON c.company_code = e.company_code
GROUP BY c.company_code, c.founder
ORDER BY c.company_code;

-- Task 11 (Repeated Task 2): Higher Paid Best Friends
-- (Same query as Task 2)
SELECT s.Name
FROM Students s
JOIN Friends f ON s.ID = f.ID
JOIN Packages sp ON s.ID = sp.ID
JOIN Packages fp ON f.Friend_ID = fp.ID
WHERE sp.Salary < fp.Salary
ORDER BY fp.Salary;

-- Task 15: Top 5 Salaries Without ORDER BY
SELECT DISTINCT Salary
FROM Employees e1
WHERE 5 > (
    SELECT COUNT(DISTINCT Salary)
    FROM Employees e2
    WHERE e2.Salary > e1.Salary
);

-- Task 16: Swap column values without temp variable
UPDATE table_name
SET column1 = column1 + column2,
    column2 = column1 - column2,
    column1 = column1 - column2;

-- Task 17: Create user and grant DB_owner
CREATE LOGIN TestUser WITH PASSWORD = 'StrongPassword123';
CREATE USER TestUser FOR LOGIN TestUser;
ALTER ROLE db_owner ADD MEMBER TestUser;

-- Task 18: Weighted Average Cost Month on Month
SELECT MONTH(Date) AS Month,
       SUM(Salary * Weight) / SUM(Weight) AS Weighted_Avg_Cost
FROM Employees
GROUP BY MONTH(Date);

-- Task 19: Salary Miscalculation Difference
SELECT CEILING(AVG(Salary) - AVG(CAST(REPLACE(Salary, '0', '') AS INT))) AS Difference
FROM Employees;

-- Task 20: Copy New Data (Assume NOT EXISTS strategy)
INSERT INTO TargetTable (col1, col2, ...)
SELECT col1, col2, ...
FROM SourceTable s
WHERE NOT EXISTS (
    SELECT 1 FROM TargetTable t WHERE t.primary_key = s.primary_key
);
