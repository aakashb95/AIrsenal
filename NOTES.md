Note that running any of the commands mentioned below assume you have AIrsenal installed and its environment activated (if it was installed in a virtual environment).

## Database

The database is filled with data from the previous three FPL seasons (stored in the repo at `airsenal/data`), as well as data from the current season (obtained from the FPL API). A lot of the historic FPL data has been compiled with the help of the [vaastav/Fantasy-Premier-League](https://github.com/vaastav/Fantasy-Premier-League) repo. The database also contains information about the user's current FPL squad and AIrsenal's point predictions for each player and transfer recommendations. It has the following tables:

**_Tables with Football Club Data:_**
- **Team:** Name (short and full) and FPL ID for each team in each of the last three (and current) premier league seasons.

- **Fixture:** All past and future scheduled fixtures (home team, away team, date, gameweek and season).

- **Result:** Match result for each fixture (if available) - home goals and away goals.

- **FifaTeamRating:** FIFA (the game) team ratings for each team in each season. These are taken from https://www.fifaindex.com/teams/. Their main use is to give an idea of team strength for promoted teams (but we are considering their use in general - see [Issue #136](https://github.com/alan-turing-institute/AIrsenal/issues/136))

  
**_Tables with Player Data:_**
- **Player:** Name and ID for each player (for all football players that have been in the FPL game at some point in the last 3 seasons). The ID is the same as the FPL player ID for players active in the current season.

- **PlayerAttributes:** Attributes for each player in every gameweek of each season. The main attributes are FPL price, FPL position, and the team the player plays for. 

- **PlayerScore:** Stats for each player in each match. These include points scored, goals scored and condeded, assists, bonus points, minutes played and a few others. Only contains the stats available from the FPL API, we don't currently have other stats (like xG).

**_Tables with FPL Squad Data:_**

- **Transaction:** Contains the history of each player you have bought and sold in your FPL team, and for what price (and which gameweek). This is used to determine your starting team and budget before running transfer optimisations.

**_Tables with AIrsenal Data:_**

- **PlayerPrediction:** Stores predicted points for each player in each fixture from an AIrsenal prediction run.

- **TransferSuggestion:** Stores recommended transfers from AIrsenal optimisation runs.


The database schema is defined using `sqlalchemy` in `airsenal.framework.schema.py`.

### Initial Database Setup
To setup the AIrsenal database:
1. Remove any pre-existing database:
  `rm /tmp/data.db`
2. Run  `airsenal_setup_initial_db`
(previously `setup_airsenal_database`)
3. A sqlite3 database will be created at `/tmp/data.db` containing the AIrsenal data.
4. Update the database to get the latest status of your FPL team (transfers made etc.) - see below.

This runs the function `main()` in `airsenal.scripts.fill_db_init`, which calls functions in other scripts to fill the individual tables from the data files and API.

### Updating the Database

To update the AIrsenal database run `airsenal_update_db` (previously `update_airsenal_database`). This calls the function `main()` in `airsenal.scripts.update_results_transactions_db`.

It does the following:
1. Update player attributes with their latest values.
2. Adds any new match results.
3. Adds any new player scoores.
4. Adds any new FPL transfers that have been made.

You should always run `update_airsenal_database --noattr` after an initial database setup to get the latest status of your FPL team (which is not done as part of `airsenal_setup_initial_db`). Giving the `--noattr` flag means player attributes will not be updated, which is not needed if you have just setup the database (as FPL player attributes can only change once per day).

Note we don't currently have a way to update the list of currently active _players_ (only their attributes). This unfortunately means that if a new player is added to the game the whole database needs to be recreated. It's therefore best to follow the initial database setup steps above at the start of every gameweek. 

### Data Sanity Checks


## Player Points Predictions

Player points predictions are generated from three main components:
1. A team-level model to predict final score probabilities for each match.
2. A player-level model to predict the probability that a player scores or assists each goal his team scores.
3. The number of minutes the player has played in recent matches (by default the last 3)

### Team Model

### Player Model

### How Predicted Points are Calculated

First:
- Fit the team and player models.
- Predict the probability of each number of goals scored and conceded for each team in each fixture in the gameweek to consider, using the team model.
- Get the number of minutes each player played in the last three fixtures.

Then calculate the different points contributions below for each of the number of minutes played in the last 3 fixtures (e.g. if a player played 0 mins, 70 mins and 90 mins in the last 3 games, we make 3 predictions for the next match assuming he will play 0 mins, 70 mins oor 90 mins). For both attacking and defending points, the probability of to scoring, assisting, or conceding a goal is weighted by the number of minutes (fraction of the match) the player is estimated to play.

**Appearance Points:**
As per the FPL rules, 0pts if player doesn't play, 1pt for less 60 minutes, and 2pts for 60 minutes or more.

See `get_appearance_points` in `airsenal.framework.FPL_scoring_rules`.

**Attacking Points (Goals Scored):**
The following is done in `get_attacking_points()` in `airsenal.framework.prediction_utils`:

- probability team scores that number of goals
- possible permutations of number of goals and assists for the player given the team scores tbat many goals
- probability of each permutation of goals and assists using trinomial player model (probability player scores, assists or not involved in a goal)
- FPL points of each permutation given number of points for a goal and assist for player's position.
- multiply probabilities and pts totals

We always assume zero attacking points for goalkeepers (and don't perform the calculation above).

**Defending Points (Goals Conceded):**
The following is done in `get_defending_points()` in `airsenal.framework.prediction_utils`:
- Calculate clean sheet points for each player (assuming player expected to play 60 mins):
    - For goalkeepers & defenders: 4pts x probability their team concedes zero goals.
    - For midfielders: 1pt x probability their team concedes zero goals.
- Calculate points lost due to goals conceded for each player:
    - For goalkeepers & defenders: -P(2 or 3 goals conceded) - P(4 or 5 goals conceded) etc.

Convert to FPL points

Double and blank gameeweeks

We currently don't have predictions for the points contribution from bonus points, save points, yellow and red cards, own goals, or penalty misses and saves.

### Running Points Predictions


## Creating a Team for the Start of the Season


## Transfer & Squad Optimisation

Starting XI, captain & subs

Chips

---
# Old notes

## Data wrangling

Different aspects of the data come from different sources:

 * Player data for the upcoming 2018/19 season, and from the 2017/18 season, are from the FPL API.
 * Player data from previous seasons are from scraping the FPL archives page.
 * Match results from previous seasons are copy/pasted from Sky Sports website, and parsed.
 * Fixtures for 2018/19 are copy/pasted from the FPL website and parsed.

Many of these different sources can have slightly different names for players and/or teams (e.g. FPL
insists on using "Spurs" for some reason).  There are a couple of very naive scripts:
```match_team_names.py``` and ```match_player_names.py``` which aim to reconcile some of these
differences.

For both team names and player names, we will use the FPL values ("first_name second_name" for players).
The scripts mentioned above will generate dictionaries with these names as the keys, and with a list
of possible alternative names as the values.  These dictionaries (saved in json files) will be used
when writing into the SQLite database.

The "results_xxyy.csv" files contain the date of matches, but not their
corresponding "gameweek" in the FPL season (maybe we don't need this anyway,
but it could be useful to match up players with historic results.  There is
a script ```find_gameweek_for_match.py``` that attempts to match these up
using the player_detail_xxyy.json data, but it is not perfect, and the results
require manual tweaking. (To be investigated).

## Database

The proposed schema of the database is as follows:

```
Player Table
============
player_id, player_name, current_team, current_price, purchased_price
```
(The purchased_price is necessary to calculate how much the player can be
sold for)

```
Match Table
===========
match_id, date, season, gameweek, home_team, away_team, home_score, away_score
```

```
Fixture Table
=============
fixture_id, date, gameweek, home_team, away_team
```

````
PlayerScore Table
=================
player_id, match_id, played_for, opponent, score, ... , ...
```
(Note that this has columns for played_for and opponent since players can
change teams)

```
PlayerPrediction Table
======================
player_id, gameweek, predicted_score, method
```
(the method column allows us to have more than one predicted score per
player per gameweek, and add more later, without having to add more columns.
We now have a gameweek column rather than a fixture_id column - in double gameweeks
the player can have more than one fixture in a gameweek - the score we show is the 
sum of both.)

```
Transaction Table
=================
player_id, gameweek, bought_or_sold
```
(the bought_or_sold column can have values +1 for buying a player or -1 for selling)
