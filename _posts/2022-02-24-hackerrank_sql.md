---
title: HackerRank SQL 문제
data: 2022-02-24 02:00:00 +09:00
categories: [DB]
tag: [SQL, DB]
---
# [SQL Project Planning](https://www.hackerrank.com/challenges/sql-projects)

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


# [Symmetric Pairs](https://www.hackerrank.com/challenges/symmetric-pairs)
(5, 5) 처럼 X와 Y가 같은 값은 두 개 이상 있어야 pair가 되므로 `COUNT(a.X) > 1`가 필요합니다.
```sql
SELECT a.X, a.Y
FROM Functions a JOIN Functions b ON a.Y=b.X AND a.X=b.Y
GROUP BY a.X, a.Y
HAVING COUNT(a.X) > 1 OR a.X < a.Y
ORDER BY a.X, a.Y
```


# [The Report](https://www.hackerrank.com/challenges/the-report)
```sql
SELECT
    CASE WHEN b.Grade >= 8 THEN a.Name ELSE 'NULL' END
    , b.Grade
    , a.Marks
FROM Students a JOIN Grades b ON b.Min_Mark <= a.Marks AND a.Marks <= b.Max_Mark
ORDER BY b.Grade DESC, a.Name, a.Marks DESC
```


# [Challenges](https://www.hackerrank.com/challenges/challenges)
```sql
SELECT
    hacker_id
    , name
    , challenges_created
FROM (
    SELECT
        h.hacker_id AS hacker_id
        , h.name AS name
        , COUNT(*) AS challenges_created
    FROM Challenges c JOIN Hackers h ON c.hacker_id=h.hacker_id
    GROUP BY h.hacker_id, h.name
    ORDER BY challenges_created DESC, h.hacker_id
) ret
WHERE challenges_created = (
    SELECT MAX(challenges_created)
    FROM (
        SELECT
            hacker_id
            , COUNT(*) AS challenges_created
        FROM Challenges
        GROUP BY hacker_id
    ) r1
) OR challenges_created IN (
    SELECT challenges_created
    FROM (
        SELECT
            hacker_id
            , COUNT(*) AS challenges_created
        FROM Challenges
        GROUP BY hacker_id
    ) r2
    GROUP BY challenges_created
    HAVING COUNT(*)=1
)
```


# [The PADS](https://www.hackerrank.com/challenges/the-pads)
```sql
(
    SELECT
        msg
    FROM (
        SELECT
            Name
            , CONCAT(Name, '(', SUBSTR(Occupation, 1, 1), ')') AS msg
        FROM OCCUPATIONS
        ORDER BY Name
    ) r1
)
UNION ALL
(
    SELECT
        msg
    FROM (
        SELECT
            COUNT(*) as num
            , CONCAT(
                'There are a total of '
                , COUNT(*)
                , ' '
                , LOWER(Occupation)
                , 's.'
            ) AS msg
        FROM OCCUPATIONS
        GROUP BY Occupation
        ORDER BY num DESC, msg
    ) r2
)
```
이렇게 했으나, UNION ALL 단계에서 정렬이 풀렸다.
```sql
SELECT msg
FROM
(
    (
        SELECT
            msg
            , 1 AS filter1
            , ROW_NUMBER() OVER(ORDER BY Name) AS filter2
        FROM (
            SELECT
                Name
                , CONCAT(Name, '(', SUBSTR(Occupation, 1, 1), ')') AS msg
            FROM OCCUPATIONS
            ORDER BY Name
        ) r1
    )
    UNION ALL
    (
        SELECT
            msg
            , 2 AS filter1
            , ROW_NUMBER() OVER(ORDER BY num ASC, msg) AS filter2
        FROM (
            SELECT
                COUNT(*) as num
                , CONCAT(
                    'There are a total of '
                    , COUNT(*)
                    , ' '
                    , LOWER(Occupation)
                    , 's.'
                ) AS msg
            FROM OCCUPATIONS
            GROUP BY Occupation
            ORDER BY num ASC, msg
        ) r2
    )
) ret
ORDER BY filter1, filter2
```
filter값을 추가해서 나중에 다시 정렬하는 방법을 사용하였습니다.

하지만... 그냥 이렇게 출력해도 되는거였다;

```sql
SELECT
    CONCAT(Name, '(', SUBSTR(Occupation, 1, 1), ')') AS msg
FROM OCCUPATIONS
ORDER BY Name;

SELECT
    CONCAT(
        'There are a total of '
        , COUNT(*)
        , ' '
        , LOWER(Occupation)
        , 's.'
    ) AS msg
FROM OCCUPATIONS
GROUP BY Occupation
ORDER BY COUNT(*) ASC, msg;
```


# [Weather Observation Station 20](https://www.hackerrank.com/challenges/weather-observation-station-20)
```sql
SELECT
    ROUND(LAT_N, 4)
FROM (
    SELECT
        LAT_N
        , ROW_NUMBER() OVER(ORDER BY LAT_N) AS num
    FROM STATION
) ret
WHERE num = (
    SELECT CEIL(COUNT(*) / 2)
    FROM STATION
)
```


# [Interviews](https://www.hackerrank.com/challenges/interviews)
길긴 하지만 별 내용은 없었다.
```sql
SELECT
    cont.contest_id
    , cont.hacker_id
    , cont.name
    , SUM(submsum.total_submissions) AS total_submissins
    , SUM(submsum.total_accepted_submissions) AS total_accepted_submissions
    , SUM(viewsum.total_views) AS total_views
    , SUM(viewsum.total_unique_views) AS total_unique_views
FROM Contests cont
    LEFT JOIN Colleges coll
        ON cont.contest_id=coll.contest_id
    LEFT JOIN Challenges chal
        ON coll.college_id=chal.college_id
    LEFT JOIN (
        SELECT
            challenge_id
            , SUM(total_views) AS total_views
            , SUM(total_unique_views) AS total_unique_views
        FROM View_Stats
        GROUP BY challenge_id
    ) AS viewsum
    ON chal.challenge_id=viewsum.challenge_id
    LEFT JOIN (
        SELECT
            challenge_id
            , SUM(total_submissions) AS total_submissions
            , SUM(total_accepted_submissions) AS total_accepted_submissions
        FROM Submission_Stats
        GROUP BY challenge_id
    ) AS submsum
    ON chal.challenge_id=submsum.challenge_id
GROUP BY
    cont.contest_id, cont.hacker_id, cont.name
HAVING
    (
        SUM(submsum.total_submissions)
        + SUM(submsum.total_accepted_submissions)
        + SUM(viewsum.total_views)
        + SUM(viewsum.total_unique_views)
    ) > 0
ORDER BY cont.contest_id
```
