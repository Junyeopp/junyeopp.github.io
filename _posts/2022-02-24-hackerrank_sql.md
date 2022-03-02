---
title: HackerRank SQL 문제
data: 2022-02-24 02:00:00 +09:00
categories: [DB]
tag: [SQL, DB]
---
# [SQL Project Planning](https://www.hackerrank.com/challenges/sql-projects/problem?isFullScreen=true)

```sql
WITH
start AS (
    SELECT Start_Date, ROW_NUMBER() OVER(ORDER BY Start_Date) AS rn
    FROM Projects
    WHERE Start_date NOT in (SELECT End_Date FROM Projects)
)
, end AS (
    SELECT End_Date, ROW_NUMBER() OVER(ORDER BY End_Date) AS rn
    FROM Projects
    WHERE End_Date NOT in (SELECT Start_Date FROM Projects)
)
SELECT start.Start_Date, end.End_Date
FROM start JOIN end ON start.rn=end.rn
ORDER BY DATEDIFF(end.End_Date, start.Start_Date), start.Start_Date
```


# [Symmetric Pairs](https://www.hackerrank.com/challenges/symmetric-pairs/problem?isFullScreen=true)
(5, 5) 처럼 X와 Y가 같은 값은 두 개 이상 있어야 pair가 되므로 `COUNT(a.X) > 1`가 필요합니다.
```sql
SELECT a.X, a.Y
FROM Functions a JOIN Functions b ON a.Y=b.X AND a.X=b.Y
GROUP BY a.X, a.Y
HAVING COUNT(a.X) > 1 OR a.X < a.Y
ORDER BY a.X, a.Y
```


# [The Report](https://www.hackerrank.com/challenges/the-report/problem?isFullScreen=true)
```sql
SELECT
    CASE WHEN b.Grade >= 8 THEN a.Name ELSE 'NULL' END
    , b.Grade
    , a.Marks
FROM Students a JOIN Grades b ON b.Min_Mark <= a.Marks AND a.Marks <= b.Max_Mark
ORDER BY b.Grade DESC, a.Name, a.Marks DESC
```
