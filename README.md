# Project Worldcup database (PostgreSQL)

This repository contains my second project for the Relational Databases course provided by freeCodeCamp.org. The project involves creating a PostgreSQL database named `worldcup`, defining the schema, populating it with data from a CSV file using a bash script, and querying the database to extract meaningful insights.

## Project Structure

- **Database Schema**

  - `teams` table: Contains information about teams.
    - `team_id SERIAL PRIMARY KEY`
    - `name VARCHAR(100) NOT NULL`
  - `games` table: Contains information about games.
    - `game_id SERIAL PRIMARY KEY`
    - `year INT NOT NULL`
    - `winner_id INT NOT NULL`
    - `opponent_id INT NOT NULL`
    - `winner_goals INT NOT NULL`
    - `opponent_goals INT NOT NULL`

- **Data Insertion**

  - Bash script `insert_data.sh`: Populates the `teams` and `games` tables with data from `games.csv`.

- **Data Queries**
  - Bash script `queries.sh`: Executes SQL queries to extract and display various statistics from the database.

## Files

### `games.csv`

The CSV file containing data about World Cup games, including the year, round, winning team, opponent team, and the number of goals scored by each team.

### `worldcup.csv`

The worldcup.sql file can be used to recreate the worldcup database again.

### `insert_data.sh`

Bash script to populate the `teams` and `games` tables from `games.csv`. Below is the script:

```
#! /bin/bash

if [[ $1 == "test" ]]
then
  PSQL="psql --username=postgres --dbname=worldcuptest -t --no-align -c"
else
  PSQL="psql --username=freecodecamp --dbname=worldcup -t --no-align -c"
fi

# Do not change code above this line. Use the PSQL variable above to query your database.

# drops all the records from all the tables before running the script
echo $($PSQL "TRUNCATE teams, games;")

cat games.csv | while IFS="," read YEAR ROUND WINNER OPPONENT WINNER_GOALS OPPONENT_GOALS
do
if [[ $WINNER != winner ]]
then
  # echo $WINNER $OPPONENT

  TEAM_ID=$($PSQL "SELECT team_id FROM teams WHERE name='$WINNER';")

  if [[ -z $TEAM_ID ]]
  then
    INSERT_TEAM_RESULT=$($PSQL "INSERT INTO teams(name) VALUES('$WINNER');")
    if [[ $INSERT_MAJOR_RESULT == 'INSERT 0 1' ]]
    then
      echo "Inserted into teams, $WINNER"
    fi
  fi

fi

if [[ $OPPONENT != opponent ]]
then

  TEAM_ID=$($PSQL "SELECT team_id FROM teams WHERE name='$OPPONENT';")

  if [[ -z $TEAM_ID ]]
  then
    INSERT_TEAM_RESULT=$($PSQL "INSERT INTO teams(name) VALUES('$OPPONENT');")
    if [[ $INSERT_MAJOR_RESULT == 'INSERT 0 1' ]]
    then
      echo "Inserted into teams, $OPPONENT"
    fi
  fi

fi

done

cat games.csv | while IFS="," read YEAR ROUND WINNER OPPONENT WINNER_GOALS OPPONENT_GOALS
do
  if [[ $YEAR != year ]]
  then
    # echo $YEAR $ROUND $WINNER $OPPONENT $WINNER_GOALS $OPPONENT_GOALS
    WINNER_ID=$($PSQL "SELECT team_id FROM teams WHERE name='$WINNER';")
    OPPONENT_ID=$($PSQL "SELECT team_id FROM teams WHERE name='$OPPONENT';")
    # echo $WINNER_ID, $OPPONENT_ID
    GAME_ID=$($PSQL "SELECT game_id FROM games WHERE year='$YEAR' AND round='$ROUND' AND winner_id='$WINNER_ID' AND opponent_id='$OPPONENT_ID';")
    if [[ -z $GAME_ID ]]
    then
      INSERT_GAME_RESULT=$($PSQL "INSERT INTO games(year, round, winner_id, opponent_id, winner_goals, opponent_goals) VALUES($YEAR, '$ROUND', $WINNER_ID, $OPPONENT_ID, $WINNER_GOALS, $OPPONENT_GOALS);")
      if [[ $INSERT_GAME_RESULT == 'INSERT 0 1' ]]
      then
        echo "Inserted into games"
      fi
    fi


  fi
done
```

### `queries.sh`

Bash script to run various SQL queries on the worldcup database. Below is the script:

```
#! /bin/bash

PSQL="psql --username=freecodecamp --dbname=worldcup --no-align --tuples-only -c"

# Do not change code above this line. Use the PSQL variable above to query your database.

echo -e "\nTotal number of goals in all games from winning teams:"
echo "$($PSQL "SELECT SUM(winner_goals) FROM games;")"

echo -e "\nTotal number of goals in all games from both teams combined:"
echo "$($PSQL "SELECT SUM(winner_goals + opponent_goals) FROM games;")"

echo -e "\nAverage number of goals in all games from the winning teams:"
echo $($PSQL "SELECT AVG(winner_goals) FROM games;")

echo -e "\nAverage number of goals in all games from the winning teams rounded to two decimal places:"
echo $($PSQL "SELECT ROUND(AVG(winner_goals), 2) FROM games;")

echo -e "\nAverage number of goals in all games from both teams:"
echo $($PSQL "SELECT AVG(winner_goals + opponent_goals) FROM games;")

echo -e "\nMost goals scored in a single game by one team:"
echo $($PSQL "SELECT MAX(winner_goals) FROM games;")

echo -e "\nNumber of games where the winning team scored more than two goals:"
echo $($PSQL "SELECT COUNT(*) FROM games WHERE winner_goals > 2;")

echo -e "\nWinner of the 2018 tournament team name:"
echo $($PSQL "SELECT name FROM games FULL JOIN teams ON games.winner_id = teams.team_id WHERE round='Final' AND year=2018;")

echo -e "\nList of teams who played in the 2014 'Eighth-Final' round:"
echo $($PSQL "SELECT name FROM games INNER JOIN teams ON games.winner_id = teams.team_id OR games.opponent_id = teams.team_id WHERE round='Eighth-Final' AND year=2014 ORDER BY name ASC;")

echo -e "\nList of unique winning team names in the whole data set:"
echo $($PSQL "SELECT DISTINCT(name) FROM games LEFT JOIN teams ON games.winner_id = teams.team_id ORDER BY name ASC;")

echo -e "\nYear and team name of all the champions:"
echo $($PSQL "SELECT year, name FROM games FULL JOIN teams ON games.winner_id = teams.team_id WHERE round='Final' ORDER BY year;")

echo -e "\nList of teams that start with 'Co':"
echo $($PSQL "SELECT name FROM teams WHERE name LIKE 'Co%';")

```

### Learnings and Insights

This project provided hands-on experience with:

- Creating and managing PostgreSQL databases.
- Writing and executing bash scripts.
- Populating database tables from CSV files.
- Querying databases to extract and analyze data.
- Applying SQL queries to real-world datasets.
