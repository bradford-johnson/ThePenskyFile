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

`baseball_total_pitches_by_team_and_player`
---
```sql
WITH
  away_pitches AS (
  SELECT
    awayTeamName AS team,
    pitcherId,
    pitcherThrowHand,
    pitchTypeDescription,
    MIN(pitchSpeed) AS slowest_pitch,
    MAX(pitchSpeed) AS fastest_pitch,
    COUNT(atBatEventType) AS n_pitches,
    COUNT(DISTINCT gameId) AS n_games
  FROM
    `bigquery-public-data.baseball.games_wide`
  WHERE
    inningHalf = 'BOT'
    AND atBatEventType = 'PITCH'
    AND pitchSpeed > 0
  GROUP BY
    team,
    pitcherId,
    pitcherThrowHand,
    pitchTypeDescription ),
  home_pitches AS (
  SELECT
    homeTeamName AS team,
    pitcherId,
    pitcherThrowHand,
    pitchTypeDescription,
    MIN(pitchSpeed) AS slowest_pitch,
    MAX(pitchSpeed) AS fastest_pitch,
    COUNT(atBatEventType) AS n_pitches,
    COUNT(DISTINCT gameId) AS n_games
  FROM
    `bigquery-public-data.baseball.games_wide`
  WHERE
    inningHalf = 'TOP'
    AND atBatEventType = 'PITCH'
    AND pitchSpeed > 0
  GROUP BY
    team,
    pitcherId,
    pitcherThrowHand,
    pitchTypeDescription ),
  names AS (
  SELECT
    DISTINCT pitcherId,
    CONCAT(pitcherFirstName, " ", pitcherLastName) AS p_name
  FROM
    `bigquery-public-data.baseball.games_wide` )
SELECT
  p.team AS team_name,
  p.pitcherId,
  n.p_name,
  p.pitchTypeDescription,
  min(p.slowest_pitch) AS slowest_speed,
  max(p.fastest_pitch) AS fastest_speed,
  SUM(n_pitches) AS total_pitches,
  sum(p.n_games) AS num_games
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
  team_name,
  p.pitcherId,
  n.p_name,
  p.pitchTypeDescription
```
## Output:
| Row |  team_name |  pitcherId                           |  p_name        |  pitchTypeDescription |  slowest_speed |  fastest_speed |  total_pitches |  num_games |
|-----|------------|--------------------------------------|----------------|-----------------------|----------------|----------------|----------------|------------|
|   1 | Blue Jays  | cc86d4d3-1618-415e-b7fc-a303f3b8dd6f | Marcus Stroman | Slider                |             80 |             89 |            339 |         32 |
|     |            |                                      |                |                       |                |                |                |            |

`baseball_at_bats_by_batter_and_char`
---
```sql
WITH
  away_batters AS(
  SELECT
    awayTeamName AS team,
    hitterId,
    hitterBatHand,
    pitchTypeDescription,
    outcomeDescription,
    COUNT(DISTINCT gameId) AS n_games,
    COUNT(pitchType) AS n_occur,
    AVG(pitchSpeed) AS avg_pitch_speed
  FROM
    `bigquery-public-data.baseball.games_wide`
  WHERE
    atBatEventType = 'PITCH'
    AND pitchSpeed > 0
    AND inningHalf = 'TOP'
  GROUP BY
    team,
    hitterId,
    hitterBatHand,
    pitchTypeDescription,
    outcomeDescription ),
  home_batters AS(
  SELECT
    homeTeamName AS team,
    hitterId,
    hitterBatHand,
    pitchTypeDescription,
    outcomeDescription,
    COUNT(DISTINCT gameId) AS n_games,
    COUNT(pitchType) AS n_occur,
    AVG(pitchSpeed) AS avg_pitch_speed
  FROM
    `bigquery-public-data.baseball.games_wide`
  WHERE
    atBatEventType = 'PITCH'
    AND pitchSpeed > 0
    AND inningHalf = 'BOT'
  GROUP BY
    team,
    hitterId,
    hitterBatHand,
    pitchTypeDescription,
    outcomeDescription ),
  names AS(
  SELECT
    DISTINCT hitterId,
    CONCAT(hitterFirstName, " ", hitterLastName) AS h_name
  FROM
    `bigquery-public-data.baseball.games_wide` )
SELECT
b.team AS team_name,
b.hitterId,
n.h_name,
b.hitterBatHand,
b.pitchTypeDescription,
b.outcomeDescription,
SUM(b.n_games) AS num_games,
sum(b.n_occur) AS n_occur,
AVG(b.avg_pitch_speed) AS avg_pitch_speed
FROM (
  SELECT
    *
  FROM
    away_batters
  UNION ALL
  SELECT
    *
  FROM
    home_batters ) AS b
INNER JOIN
  names AS n
ON
  b.hitterId = n.hitterId
  GROUP BY
  team_name,
  b.hitterId,
  n.h_name,
  b.hitterBatHand,
  b.pitchTypeDescription,
  b.outcomeDescription
```
## Output:
| Row |  team_name |  hitterId                            |  h_name      |  hitterBatHand |  pitchTypeDescription |  outcomeDescription |  num_games |  n_occur |  avg_pitch_speed |
|-----|------------|--------------------------------------|--------------|----------------|-----------------------|---------------------|------------|----------|------------------|
|   1 | Marlins    | 933a0bc9-26c0-478e-982c-01fd8ffe0614 | Martin Prado | R              | Fastball              | Strike Looking      |        143 |      393 |    91.5761345852 |
|     |            |                                      |              |                |                       |                     |            |          |                  |
