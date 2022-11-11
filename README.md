# FIFA2023
 
1. List all games today
```
select * from games where game_date like “8/20/2023"
```

2. List a teams matches and results
```
SELECT games.id as game_id, game_date, minutes_played, penalty_shootout,
(select count(*) from events where games.id = events.game_id and events.event_id = 5) as shots,
(select count(*) from events where games.id = events.game_id and events.team_id = 8 and events.event_id = 1) as score,
(select count(*) from events where games.id = events.game_id and events.team_id != 8 and events.event_id = 1) as opponent_score,
(select count(*) from events where games.id = events.game_id and events.event_id = 2) as assists,
(select count(*) from events where games.id = events.game_id and events.event_id = 3) as yellow_card,
(select count(*) from events where games.id = events.game_id and events.event_id = 4) as red_card
from games

left join events
on events.game_id = games.id

left join event_type_id
on event_type_id.id = events.event_id

inner JOIN teams
on teams.id = games.home_team_id

WHERE home_team_id = 8 or away_team_id = 8

GROUP by game_id
```

3. List a group table
```
select *, (scored_goals - conceeded_goals) as goal_differece from teams where group_id = 1
```

4. List top 10 players sorted by goals
```
Select id, goals, first_name, last_name
from players
Order by goals desc
Limit 10
```

List top 10 players sorted by assist
```
Select id, assists, first_name, last_name
from players
Order by assists desc
Limit 10
```

5. List unavailable players
```
select players.id as player_id, players.first_name, players.last_name,
(select count(id) from events where events.event_id = 4 and events.player_id = players.id) as red_cards,
(select count(*) from events where events.event_id = 3 and events.player_id = players.id) as yellow_cards
from players
inner join events
on events.player_id = players.id
where red_cards >= 1 or yellow_cards >= 2
GROUP by player_id
```

6. List a teams roster
```
select players.team_id, coach_id, players.id as player_id, first_name, last_name,
(select count(*) from events where players.id = events.player_id and events.event_id = 5) as shots,
(select count(*) from events where players.id = events.player_id and events.event_id = 1) as score,
(select count(*) from events where players.id = events.player_id and events.event_id = 2) as assists,
(select count(*) from events where players.id = events.player_id and events.event_id = 3) as yellow_card,
(select count(*) from events where players.id = events.player_id and events.event_id = 4) as red_card,
saves, shots_faced, clean_sheets,
(saves * 100 / shots_faced) as save_percentage
from players
inner join teams
ON teams.id = players.team_id
left join events
ON events.player_id = players.id
left join event_type_id
on event_type_id.id = events.event_id
left join goal_keepers
on goal_keepers.player_id = players.id
where players.team_id = 32
GROUP by players.id
```

7. Detailed info for a finished game
```
select games.id as game_id, game_date, games.minutes_played as game_length, penalty_shootout, referee_id, venue_id, games_players.player_id, players.first_name, players.last_name, started, games_players.minutes_played, players.team_id, teams.team_name,
(event_type_id.id = 1) as score,
(event_type_id.id = 2) as assist,
(event_type_id.id = 3) as yellow,
(event_type_id.id = 4) as red,
(event_type_id.id = 5) as shot,
(event_type_id.id = 6) as save,
(event_type_id.id = 7) as sub_in,
(event_type_id.id = 8) as sub_out,
events."time" as minute, events.event_info
from games
inner join games_players
on games_players.game_id = games.id
left join events
on games_players.player_id = events.player_id
left join event_type_id
on event_type_id.id = events.event_id
inner join players
on players.id = games_players.player_id
inner join teams
on teams.id = players.team_id
where games.id = 3
```

8. Short info about specific game
```
select
home_team.team_name as home_team,
home_team_flag.team_abbreviation as home_team_abbreviation,
home_team_flag.flag_url as home_team_flag,
home_team.scored_goals as home_team_score,
away_team.team_name as away_team,
away_team_flag.team_abbreviation as away_team_abbreviation,
away_team_flag.flag_url as away_team_flag,
away_team.scored_goals as away_team_score,

(ABS(home_team.scored_goals - away_team.scored_goals)) as goal_difference
from games
inner JOIN teams as home_team
on games.home_team_id = home_team.id
inner join teams as away_team
on games.away_team_id = away_team.id
inner join flags as home_team_flag
on home_team.flag_id = home_team_flag.id
inner join flags as away_team_flag
on away_team.flag_id = away_team_flag.id
where games.id = 3
```

9. List playoff tree
```
select
games.id AS "game", 
games.game_date, 
games.game_time_UTC,
games.minutes_played,
home_team.team_abbreviation AS "home_team",
home_team_flags.flag_url AS "home_team_flag",
away_team.team_abbreviation AS "away_team",
away_team_flags.flag_url AS "away_team_flag"
FROM games
left JOIN flags AS "home_team"
ON home_team_id = home_team.id
left JOIN flags AS "away_team"
ON away_team_id = away_team.id

left JOIN flags AS "home_team_flags"
ON home_team_id = home_team_flags.id
left JOIN flags AS "away_team_flags"
ON away_team_id = away_team_flags.id

WHERE games.game_type != "group"
```

