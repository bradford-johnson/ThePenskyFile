`baseball_total_pitches_by_team_and_player`
---
```sql
WITH
  away_pitches AS (
  SELECT
    awayTeamName AS team,
    pitcherId,
    COUNT(atBatEventType) AS n_pitches
  FROM
    `bigquery-public-data.baseball.games_wide`
  WHERE
    inningHalf = 'BOT'
    AND atBatEventType = 'PITCH'
  GROUP BY
    team,
    pitcherId ),
  home_pitches AS (
  SELECT
    homeTeamName AS team,
    pitcherId,
    COUNT(atBatEventType) AS n_pitches
  FROM
    `bigquery-public-data.baseball.games_wide`
  WHERE
    inningHalf = 'TOP'
    AND atBatEventType = 'PITCH'
  GROUP BY
    team,
    pitcherId ),
  names AS (
  SELECT
    DISTINCT pitcherId,
    CONCAT(pitcherFirstName, " ", pitcherLastName) AS p_name
  FROM
    `bigquery-public-data.baseball.games_wide` )
SELECT
  p.team,
  p.pitcherId,
  n.p_name,
  SUM(n_pitches) AS total_pitches
FROM (
  SELECT
    *
  FROM
    away_pitches
  UNION ALL
  SELECT
    *
  FROM
    home_pitches ) AS p
INNER JOIN
  names AS n
ON
  p.pitcherId = n.pitcherId
GROUP BY
  p.team,
  p.pitcherId,
  n.p_name
  ```
## Output:
| Row |  team     |  pitcherId                           |  p_name        |  total_pitches |
|-----|-----------|--------------------------------------|----------------|----------------|
|   1 | Blue Jays | cc86d4d3-1618-415e-b7fc-a303f3b8dd6f | Marcus Stroman |           3108 |
|     |           |                                      |                |                |
