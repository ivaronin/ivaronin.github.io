# Predicting College and NFL Games for Pick 4 (my favorite silly sports picking league)
## Background
With (American) football season kicking off ~~soon~~ last night, I wanted to try to improve my process for picking college and pro football teams against my friends in our silly sports picking leagues. My favorite friends league is called Pick 4 and involves picking exactly four games [against the spread](https://ats.io/sports-betting/what-is-against-the-spread/) each week. Any college game involving an FBS (the top College division) team or any NFL game is eligble. 

## My current approach

Although our picks are due on Saturday morning (and most games are played on Saturday and Sunday), in this league lines lock on Tuesdays. This means that the live (real) line can move quite a bit (due to news or large quantities of money being placed on one of the teams) while the Pick 4 line stays fixed in place at the Tuesday line. When the live line and our fixed Pick 4 line move out of sync it can provide a delightful opportunity for arbitrage. 

Another factor I consider is the percentage of bets that are placed on a team and how the line moves relative to that team. If a team receives a small percentage of bets but the line moves to make it less favorable to bet on that team, this is called [reverse line movement](https://www.actionnetwork.com/education/reverse-line-movement) and it is an indicator that sharp money is betting on that team. 

At a high level, my current approach to picking games in our Pick 4 league has been:
- Scrape Office Football Pool for Pick 4 lines as soon as our lines lock on Tuesdays
- Scrape [Action Network](https://www.actionnetwork.com/nfl/public-betting) for live NFL and College Football game lines and percentage of bets placed on a team
- Join the Office Football Pool data with the most recent Action Network data
- Calculate the opportunity of each game based on a complex heuristic I've developed that considers aribtrage opportunity and reverse line movement
- Sort games by best opportunity 
- Make picks

This system has been remarkably successful and I have at least tied for first in our Pick 4 pool three of the last four years.

## Goals for this post
The purpose of this project is to add more rigor to my Pick 4 game picking process. Although my existing process has borne fruit, it leans too heavily on intuition and I'd like to use machine learning to see if I can beat my existing vibes-based approach. 

Before we proceed, I'd like to call out that I had previously factored [circadian rhythms](https://deadspin.com/the-circadian-advantage-how-sleep-patterns-benefit-cer-5934440/) into my picks until [I investigated this phenomenon two years ago](https://ianvaronin.com/2024/10/15/nfl_circadian_rhythm.htlr) and determined that was no longer a viable signal. 

## A note on LLMs
It's very clear that LLMs make excellent computer and data scientists. However, I chose not to ask an AI to do this for me. This is my own work. I believe that it's important to actually know how to write code, analyze data, and create and evaluate predictions. LLMs are powerful force multipliers but they can and do make mistakes or make incorrect assumptions. A good data science practioner should be able to check the work of an LLM and therefore needs to understand how to do this sort of work. I also believe that by playing around in the data I discovered insights that helped to inform my next steps and which might have been overlooked by an LLM. Lastly, in addition to trying to build something useful for me this was meant as exercise to help me learn and improve my skills. I believe I accomplished these goals and I enjoyed working on it. 

With that preamble out of the way, let's dive into the approach I plan to use here.

## The plan
1. Get game result data for the past few season
2. Join my Office Football Pool and Action Network data that I've saved to the game result data
3. Build and tune a machine learning model to predict whether each favored team would cover the Pick 4 spread given the data available in that moment
4. Compare the model's predictive abilities against my existing process for choosing games

Since we must pick the outcomes of exactly four games each week, it is important to rank the confidence of our weekly picks so we can sort them. While decision trees lack such functionality, logistic regression classifiers can return probability estimates making that approach well suited to this type of problem.

To get started I will run through the process that I used to get game result data and join it to my existing Office Football Pool and Action Network data.

As with my last post we will use python and the pandas library, which is well suited for data wrangling. First we will import the pandas and sqlite3 libraries which we will immediately need.

## Import Office Football Pool Data


```python
import pandas as pd
import sqlite3
```

Going back to October 2023, every time I scraped Action Network and joined it to Office Football Pool data I saved the data to an sqlite database in a table called `game_lines`. Let's read all the data from that table into a pandas dataframe.


```python
with sqlite3.connect('pick_four') as conn:
    query = "SELECT * FROM game_lines"
    df = pd.read_sql_query(query, conn)
```

Let's take a look at the shape of the data and first few rows.

### Investigate the Data


```python
df.shape
```




    (21461, 37)




```python
pd.options.display.max_columns = 100
df.sort_values(by='current_datetime').head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>rot_num_away</th>
      <th>rot_num_home</th>
      <th>ofp_game_id</th>
      <th>team_concat</th>
      <th>organization</th>
      <th>game_time</th>
      <th>away_team</th>
      <th>away_open_line</th>
      <th>away_current_best_line</th>
      <th>ofp_away_line</th>
      <th>away_odds</th>
      <th>away_bet_percentage</th>
      <th>home_team</th>
      <th>home_open_line</th>
      <th>home_current_best_line</th>
      <th>ofp_home_line</th>
      <th>home_odds</th>
      <th>home_bet_percentage</th>
      <th>max_free_points</th>
      <th>max_free_points_team</th>
      <th>line_movement_team</th>
      <th>reverse_line_movement</th>
      <th>circadian_rhythm</th>
      <th>away_team_time_zone_offset</th>
      <th>home_team_time_zone_offset</th>
      <th>free_points_strength</th>
      <th>line_movement_strength</th>
      <th>line_movement_%_bets_value_away_team</th>
      <th>line_movement_%_bets_value_home_team</th>
      <th>free_points_value_away_team</th>
      <th>free_points_value_home_team</th>
      <th>circadian_rhythm_points_away</th>
      <th>circadian_rhythm_points_home</th>
      <th>recommendation_team</th>
      <th>current_datetime</th>
      <th>max_recommendation_points</th>
      <th>net_recommendation_points</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>341</td>
      <td>342</td>
      <td>42211</td>
      <td>tulsaflorida atl.</td>
      <td>College</td>
      <td>15:00</td>
      <td>tulsa</td>
      <td>3.5</td>
      <td>2.5</td>
      <td>4.5</td>
      <td>+104</td>
      <td>17.0</td>
      <td>florida atl.</td>
      <td>-3.5</td>
      <td>-3.0</td>
      <td>-4.5</td>
      <td>-110</td>
      <td>83.0</td>
      <td>2.0</td>
      <td>tulsa</td>
      <td>tulsa</td>
      <td>tulsa</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>very strong</td>
      <td>very strong</td>
      <td>30.0</td>
      <td>0.50</td>
      <td>64.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>tulsa</td>
      <td>2023-10-07 08:59:16.483178</td>
      <td>94.0</td>
      <td>None</td>
    </tr>
    <tr>
      <th>35</th>
      <td>395</td>
      <td>396</td>
      <td>42244</td>
      <td>connecticutrice</td>
      <td>College</td>
      <td>14:00</td>
      <td>connecticut</td>
      <td>10.0</td>
      <td>10.5</td>
      <td>9.5</td>
      <td>-110</td>
      <td>22.0</td>
      <td>rice</td>
      <td>-10.0</td>
      <td>-9.5</td>
      <td>-9.5</td>
      <td>-110</td>
      <td>78.0</td>
      <td>0.0</td>
      <td>rice</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>0.0</td>
      <td>-1.0</td>
      <td></td>
      <td>None</td>
      <td>4.0</td>
      <td>0.50</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>connecticut</td>
      <td>2023-10-07 08:59:16.483178</td>
      <td>4.0</td>
      <td>None</td>
    </tr>
    <tr>
      <th>36</th>
      <td>403</td>
      <td>404</td>
      <td>42212</td>
      <td>texas stateul lafayette</td>
      <td>College</td>
      <td>12:30</td>
      <td>texas state</td>
      <td>1.0</td>
      <td>2.5</td>
      <td>1.5</td>
      <td>-118</td>
      <td>28.0</td>
      <td>ul lafayette</td>
      <td>-1.0</td>
      <td>-1.0</td>
      <td>-1.5</td>
      <td>-110</td>
      <td>72.0</td>
      <td>-0.5</td>
      <td>ul lafayette</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>very weak</td>
      <td>None</td>
      <td>4.0</td>
      <td>0.75</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>texas state</td>
      <td>2023-10-07 08:59:16.483178</td>
      <td>4.0</td>
      <td>None</td>
    </tr>
    <tr>
      <th>37</th>
      <td>467</td>
      <td>468</td>
      <td>43184</td>
      <td>philadelphiala rams</td>
      <td>NFL</td>
      <td>13:05</td>
      <td>philadelphia</td>
      <td>-6.5</td>
      <td>-4.0</td>
      <td>-4.5</td>
      <td>-109</td>
      <td>48.0</td>
      <td>la rams</td>
      <td>6.5</td>
      <td>4.0</td>
      <td>4.5</td>
      <td>-108</td>
      <td>52.0</td>
      <td>0.5</td>
      <td>la rams</td>
      <td>la rams</td>
      <td>None</td>
      <td>None</td>
      <td>0.0</td>
      <td>-3.0</td>
      <td>very weak</td>
      <td>medium</td>
      <td>2.0</td>
      <td>4.00</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0</td>
      <td>0</td>
      <td>la rams</td>
      <td>2023-10-07 08:59:16.483178</td>
      <td>5.0</td>
      <td>None</td>
    </tr>
    <tr>
      <th>38</th>
      <td>303</td>
      <td>304</td>
      <td>42190</td>
      <td>fiunew mexico st.</td>
      <td>College</td>
      <td>Final</td>
      <td>fiu</td>
      <td>5.5</td>
      <td>6.5</td>
      <td>6.5</td>
      <td>-105</td>
      <td>43.0</td>
      <td>new mexico st.</td>
      <td>-5.5</td>
      <td>-7.0</td>
      <td>-6.5</td>
      <td>+100</td>
      <td>57.0</td>
      <td>0.5</td>
      <td>new mexico st.</td>
      <td>new mexico st.</td>
      <td>None</td>
      <td>None</td>
      <td>0.0</td>
      <td>-2.0</td>
      <td>very weak</td>
      <td>medium</td>
      <td>2.0</td>
      <td>4.00</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0</td>
      <td>0</td>
      <td>new mexico st.</td>
      <td>2023-10-07 08:59:16.483178</td>
      <td>5.0</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>



Let's take a look at the number of null rows in each column.


```python
df.isna().sum()
```




    rot_num_away                              873
    rot_num_home                              873
    ofp_game_id                             10791
    team_concat                                 0
    organization                                0
    game_time                                   0
    away_team                                   0
    away_open_line                              0
    away_current_best_line                      0
    ofp_away_line                            6320
    away_odds                                 684
    away_bet_percentage                      2354
    home_team                                   0
    home_open_line                              0
    home_current_best_line                      0
    ofp_home_line                            6320
    home_odds                                 684
    home_bet_percentage                      2400
    max_free_points                             0
    max_free_points_team                    10089
    line_movement_team                       3645
    reverse_line_movement                   15423
    circadian_rhythm                        21167
    away_team_time_zone_offset               6508
    home_team_time_zone_offset               6521
    free_points_strength                        0
    line_movement_strength                   3645
    line_movement_%_bets_value_away_team        0
    line_movement_%_bets_value_home_team        0
    free_points_value_away_team                 0
    free_points_value_home_team                 0
    circadian_rhythm_points_away                0
    circadian_rhythm_points_home                0
    recommendation_team                      2408
    current_datetime                            0
    max_recommendation_points                   0
    net_recommendation_points               21461
    dtype: int64



Right away we notice a few interesting things:
- There are a lot of columns, many more than we need for this project
- Some important columns like `ofp_away_line` and `ofp_home_line` have null values. If values for these columns are null, that means the game did not actually exist in Office Football Pool and the row can safely be dropped
- Although there are columns for `gametime` and `current_datetime`, neither of these has the date the game was played

When I pulled this data, I spent a lot of time trying to extract the game date from Action Network and Office Football Pool without success. That technical debt has come back to bite us now. We want to know the date the game was played in order to join this data to game outcome data. Fortunately, we do know some clues for each game that can help us derive the game date: the home team, the away team, and the datetime the data was pulled. The datetime the data was pulled would be immediately before the actual date when the game was played. We can use these clues to help us derive the actual date the game was played.

First let's drop unusable rows where `ofp_away_line` is null and then recheck the count of null values per column.

### Drop Nulls


```python
df = df.dropna(subset='ofp_away_line', ignore_index=True)
```


```python
df[['ofp_away_line', 'ofp_home_line']].isna().sum()
```




    ofp_away_line    0
    ofp_home_line    0
    dtype: int64



Good, now every row has as non-null value for `ofp_away_line` and `ofp_home_line`.

### Remove Duplicates

Let's take a look at duplicate values. Because I saved the data multiple times per week (every time I scraped Action Network and joined it to Office Football Pool there will be an entry), I expect there will be many duplicates.


```python
df['ofp_game_id'].value_counts().head()
```




    ofp_game_id
    43427    17
    43454    17
    43429    17
    43419    17
    44202    15
    Name: count, dtype: int64



I believe the field `ofp_game_id` is unique for each game but we should verify this by creating a dataframe where each `away_team`, `home_team`, and `ofp_game_id` combination is assigned to a row. Then we can check and see how many unique `ofp_game_id` values there are.  


```python
ofp_game_id_dupes = df.groupby(['away_team', 'home_team', 'ofp_game_id']).size().reset_index().rename(columns={0:'count'})
ofp_game_id_dupes.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>away_team</th>
      <th>home_team</th>
      <th>ofp_game_id</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>air force</td>
      <td>army</td>
      <td>44823</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>air force</td>
      <td>baylor</td>
      <td>44106</td>
      <td>6</td>
    </tr>
    <tr>
      <th>2</th>
      <td>air force</td>
      <td>boise st.</td>
      <td>43062</td>
      <td>9</td>
    </tr>
    <tr>
      <th>3</th>
      <td>air force</td>
      <td>colorado st.</td>
      <td>42582</td>
      <td>7</td>
    </tr>
    <tr>
      <th>4</th>
      <td>air force</td>
      <td>hawaii</td>
      <td>42794</td>
      <td>7</td>
    </tr>
    <tr>
      <th>5</th>
      <td>air force</td>
      <td>navy</td>
      <td>42417</td>
      <td>6</td>
    </tr>
    <tr>
      <th>6</th>
      <td>air force</td>
      <td>nevada</td>
      <td>45156</td>
      <td>5</td>
    </tr>
    <tr>
      <th>7</th>
      <td>air force</td>
      <td>new mexico</td>
      <td>44561</td>
      <td>5</td>
    </tr>
    <tr>
      <th>8</th>
      <td>air force</td>
      <td>san diego st.</td>
      <td>45284</td>
      <td>4</td>
    </tr>
    <tr>
      <th>9</th>
      <td>air force</td>
      <td>wyoming</td>
      <td>44283</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>




```python
ofp_game_id_dupes['ofp_game_id'].value_counts(ascending=False)
```




    ofp_game_id
    44823    1
    44264    1
    44534    1
    42732    1
    44710    1
            ..
    43009    1
    43798    1
    43326    1
    44370    1
    45318    1
    Name: count, Length: 1747, dtype: int64



We can confirm that each `ofp_game_id` is unique.

To double verify this, if we look at a specific `ofp_game_id` with multiple entries the `away_team` and `home_team` should be the same. 


```python
df.loc[df['ofp_game_id'] == df['ofp_game_id'].value_counts(
    ).head(1).index[0]].sort_values(by='current_datetime', ascending=False)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>rot_num_away</th>
      <th>rot_num_home</th>
      <th>ofp_game_id</th>
      <th>team_concat</th>
      <th>organization</th>
      <th>game_time</th>
      <th>away_team</th>
      <th>away_open_line</th>
      <th>away_current_best_line</th>
      <th>ofp_away_line</th>
      <th>away_odds</th>
      <th>away_bet_percentage</th>
      <th>home_team</th>
      <th>home_open_line</th>
      <th>home_current_best_line</th>
      <th>ofp_home_line</th>
      <th>home_odds</th>
      <th>home_bet_percentage</th>
      <th>max_free_points</th>
      <th>max_free_points_team</th>
      <th>line_movement_team</th>
      <th>reverse_line_movement</th>
      <th>circadian_rhythm</th>
      <th>away_team_time_zone_offset</th>
      <th>home_team_time_zone_offset</th>
      <th>free_points_strength</th>
      <th>line_movement_strength</th>
      <th>line_movement_%_bets_value_away_team</th>
      <th>line_movement_%_bets_value_home_team</th>
      <th>free_points_value_away_team</th>
      <th>free_points_value_home_team</th>
      <th>circadian_rhythm_points_away</th>
      <th>circadian_rhythm_points_home</th>
      <th>recommendation_team</th>
      <th>current_datetime</th>
      <th>max_recommendation_points</th>
      <th>net_recommendation_points</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4746</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>Final</td>
      <td>missouri</td>
      <td>4.5</td>
      <td>4.5</td>
      <td>1.5</td>
      <td>-114</td>
      <td>61.0</td>
      <td>ohio st.</td>
      <td>-4.5</td>
      <td>13.5</td>
      <td>-1.5</td>
      <td>+200</td>
      <td>39.0</td>
      <td>-3.0</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>very weak</td>
      <td>None</td>
      <td>0.0</td>
      <td>0.375</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>ohio st.</td>
      <td>2023-12-31 09:52:34.536005</td>
      <td>0.375</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4742</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>Final</td>
      <td>missouri</td>
      <td>4.5</td>
      <td>4.5</td>
      <td>1.5</td>
      <td>-114</td>
      <td>61.0</td>
      <td>ohio st.</td>
      <td>-4.5</td>
      <td>13.5</td>
      <td>-1.5</td>
      <td>+200</td>
      <td>39.0</td>
      <td>-3.0</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>very weak</td>
      <td>None</td>
      <td>0.0</td>
      <td>0.375</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>ohio st.</td>
      <td>2023-12-31 08:43:58.836016</td>
      <td>0.375</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4738</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>Final</td>
      <td>missouri</td>
      <td>4.5</td>
      <td>4.5</td>
      <td>1.5</td>
      <td>-114</td>
      <td>61.0</td>
      <td>ohio st.</td>
      <td>-4.5</td>
      <td>13.5</td>
      <td>-1.5</td>
      <td>+200</td>
      <td>39.0</td>
      <td>-3.0</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>very weak</td>
      <td>None</td>
      <td>0.0</td>
      <td>0.375</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>ohio st.</td>
      <td>2023-12-30 16:47:14.797736</td>
      <td>0.375</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4730</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>Final</td>
      <td>missouri</td>
      <td>4.5</td>
      <td>4.5</td>
      <td>1.5</td>
      <td>-114</td>
      <td>61.0</td>
      <td>ohio st.</td>
      <td>-4.5</td>
      <td>13.5</td>
      <td>-1.5</td>
      <td>+200</td>
      <td>39.0</td>
      <td>-3.0</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>very weak</td>
      <td>None</td>
      <td>0.0</td>
      <td>0.375</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>ohio st.</td>
      <td>2023-12-30 09:01:44.549546</td>
      <td>0.375</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4658</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>17:00</td>
      <td>missouri</td>
      <td>6.5</td>
      <td>4.5</td>
      <td>1.5</td>
      <td>-114</td>
      <td>65.0</td>
      <td>ohio st.</td>
      <td>-6.5</td>
      <td>-4.0</td>
      <td>-1.5</td>
      <td>-108</td>
      <td>35.0</td>
      <td>2.5</td>
      <td>ohio st.</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>elite</td>
      <td>strong</td>
      <td>0.0</td>
      <td>0.375</td>
      <td>0.0</td>
      <td>160.0</td>
      <td>0</td>
      <td>0</td>
      <td>ohio st.</td>
      <td>2023-12-29 16:20:24.119919</td>
      <td>160.375</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4616</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>17:00</td>
      <td>missouri</td>
      <td>6.5</td>
      <td>5.5</td>
      <td>1.5</td>
      <td>-110</td>
      <td>63.0</td>
      <td>ohio st.</td>
      <td>-6.5</td>
      <td>-5.0</td>
      <td>-1.5</td>
      <td>-105</td>
      <td>37.0</td>
      <td>3.5</td>
      <td>ohio st.</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>elite</td>
      <td>strong</td>
      <td>0.0</td>
      <td>0.375</td>
      <td>0.0</td>
      <td>224.0</td>
      <td>0</td>
      <td>0</td>
      <td>ohio st.</td>
      <td>2023-12-29 14:41:20.272406</td>
      <td>224.375</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4577</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>17:00</td>
      <td>missouri</td>
      <td>6.5</td>
      <td>5.5</td>
      <td>1.5</td>
      <td>-110</td>
      <td>62.0</td>
      <td>ohio st.</td>
      <td>-6.5</td>
      <td>-4.5</td>
      <td>-1.5</td>
      <td>-113</td>
      <td>38.0</td>
      <td>3.0</td>
      <td>ohio st.</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>elite</td>
      <td>strong</td>
      <td>0.0</td>
      <td>0.375</td>
      <td>0.0</td>
      <td>192.0</td>
      <td>0</td>
      <td>0</td>
      <td>ohio st.</td>
      <td>2023-12-29 10:56:28.935776</td>
      <td>192.375</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4537</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>17:00</td>
      <td>missouri</td>
      <td>6.5</td>
      <td>3.5</td>
      <td>1.5</td>
      <td>-105</td>
      <td>53.0</td>
      <td>ohio st.</td>
      <td>-6.5</td>
      <td>-3.5</td>
      <td>-1.5</td>
      <td>-110</td>
      <td>47.0</td>
      <td>2.0</td>
      <td>ohio st.</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>elite</td>
      <td>strong</td>
      <td>0.0</td>
      <td>0.250</td>
      <td>0.0</td>
      <td>128.0</td>
      <td>0</td>
      <td>0</td>
      <td>ohio st.</td>
      <td>2023-12-28 16:55:42.060024</td>
      <td>128.250</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4497</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>17:00</td>
      <td>missouri</td>
      <td>6.5</td>
      <td>3.5</td>
      <td>1.5</td>
      <td>-105</td>
      <td>53.0</td>
      <td>ohio st.</td>
      <td>-6.5</td>
      <td>-3.5</td>
      <td>-1.5</td>
      <td>-110</td>
      <td>47.0</td>
      <td>2.0</td>
      <td>ohio st.</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>elite</td>
      <td>strong</td>
      <td>0.0</td>
      <td>0.250</td>
      <td>0.0</td>
      <td>128.0</td>
      <td>0</td>
      <td>0</td>
      <td>ohio st.</td>
      <td>2023-12-28 14:35:40.545716</td>
      <td>128.250</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4457</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>17:00</td>
      <td>missouri</td>
      <td>6.5</td>
      <td>3.5</td>
      <td>1.5</td>
      <td>-105</td>
      <td>53.0</td>
      <td>ohio st.</td>
      <td>-6.5</td>
      <td>-3.5</td>
      <td>-1.5</td>
      <td>-110</td>
      <td>47.0</td>
      <td>2.0</td>
      <td>ohio st.</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>elite</td>
      <td>strong</td>
      <td>0.0</td>
      <td>0.250</td>
      <td>0.0</td>
      <td>128.0</td>
      <td>0</td>
      <td>0</td>
      <td>ohio st.</td>
      <td>2023-12-28 12:40:18.326175</td>
      <td>128.250</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4423</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>17:00</td>
      <td>missouri</td>
      <td>6.5</td>
      <td>3.0</td>
      <td>1.5</td>
      <td>-105</td>
      <td>55.0</td>
      <td>ohio st.</td>
      <td>-6.5</td>
      <td>-3.0</td>
      <td>-1.5</td>
      <td>+100</td>
      <td>45.0</td>
      <td>1.5</td>
      <td>ohio st.</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>strong</td>
      <td>strong</td>
      <td>0.0</td>
      <td>0.250</td>
      <td>0.0</td>
      <td>24.0</td>
      <td>0</td>
      <td>0</td>
      <td>ohio st.</td>
      <td>2023-12-27 16:53:40.234974</td>
      <td>24.250</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4384</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>17:00</td>
      <td>missouri</td>
      <td>6.5</td>
      <td>3.0</td>
      <td>1.5</td>
      <td>-105</td>
      <td>55.0</td>
      <td>ohio st.</td>
      <td>-6.5</td>
      <td>-3.0</td>
      <td>-1.5</td>
      <td>+100</td>
      <td>45.0</td>
      <td>1.5</td>
      <td>ohio st.</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>strong</td>
      <td>strong</td>
      <td>0.0</td>
      <td>0.250</td>
      <td>0.0</td>
      <td>24.0</td>
      <td>0</td>
      <td>0</td>
      <td>ohio st.</td>
      <td>2023-12-27 16:30:27.977229</td>
      <td>24.250</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4344</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>17:00</td>
      <td>missouri</td>
      <td>6.5</td>
      <td>3.0</td>
      <td>1.5</td>
      <td>-105</td>
      <td>55.0</td>
      <td>ohio st.</td>
      <td>-6.5</td>
      <td>-3.0</td>
      <td>-1.5</td>
      <td>+100</td>
      <td>45.0</td>
      <td>1.5</td>
      <td>ohio st.</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>strong</td>
      <td>strong</td>
      <td>0.0</td>
      <td>0.250</td>
      <td>0.0</td>
      <td>24.0</td>
      <td>0</td>
      <td>0</td>
      <td>ohio st.</td>
      <td>2023-12-27 16:03:42.525353</td>
      <td>24.250</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4304</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>17:00</td>
      <td>missouri</td>
      <td>6.5</td>
      <td>3.0</td>
      <td>1.5</td>
      <td>-105</td>
      <td>55.0</td>
      <td>ohio st.</td>
      <td>-6.5</td>
      <td>-3.0</td>
      <td>-1.5</td>
      <td>+100</td>
      <td>45.0</td>
      <td>1.5</td>
      <td>ohio st.</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>strong</td>
      <td>strong</td>
      <td>0.0</td>
      <td>0.250</td>
      <td>0.0</td>
      <td>24.0</td>
      <td>0</td>
      <td>0</td>
      <td>ohio st.</td>
      <td>2023-12-27 14:43:28.730390</td>
      <td>24.250</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4264</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>17:00</td>
      <td>missouri</td>
      <td>6.5</td>
      <td>3.0</td>
      <td>1.5</td>
      <td>-105</td>
      <td>55.0</td>
      <td>ohio st.</td>
      <td>-6.5</td>
      <td>-3.0</td>
      <td>-1.5</td>
      <td>+100</td>
      <td>45.0</td>
      <td>1.5</td>
      <td>ohio st.</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>strong</td>
      <td>strong</td>
      <td>0.0</td>
      <td>0.250</td>
      <td>0.0</td>
      <td>24.0</td>
      <td>0</td>
      <td>0</td>
      <td>ohio st.</td>
      <td>2023-12-27 14:26:21.101843</td>
      <td>24.250</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4241</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>17:00</td>
      <td>missouri</td>
      <td>6.5</td>
      <td>1.5</td>
      <td>1.5</td>
      <td>-110</td>
      <td>56.0</td>
      <td>ohio st.</td>
      <td>-6.5</td>
      <td>-1.0</td>
      <td>-1.5</td>
      <td>-110</td>
      <td>44.0</td>
      <td>0.0</td>
      <td>missouri</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td></td>
      <td>elite</td>
      <td>0.0</td>
      <td>0.250</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>missouri</td>
      <td>2023-12-26 19:42:29.314446</td>
      <td>1.000</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4201</th>
      <td>263</td>
      <td>264</td>
      <td>43427</td>
      <td>missouriohio st.</td>
      <td>College</td>
      <td>17:00</td>
      <td>missouri</td>
      <td>6.5</td>
      <td>1.5</td>
      <td>1.5</td>
      <td>-110</td>
      <td>55.0</td>
      <td>ohio st.</td>
      <td>-6.5</td>
      <td>-1.0</td>
      <td>-1.5</td>
      <td>-110</td>
      <td>45.0</td>
      <td>0.0</td>
      <td>missouri</td>
      <td>missouri</td>
      <td>None</td>
      <td>None</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td></td>
      <td>elite</td>
      <td>0.0</td>
      <td>0.250</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>missouri</td>
      <td>2023-12-26 15:28:19.582816</td>
      <td>1.000</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>



17 rows for the Missouri Ohio St game from 2023! I was busy that week! We only need one row per game and it should be the row closest to, but before, kickoff for that game to give us the most current data. The reason we want to use only data from before kickoff is because I've noticed that sometimes the data on Action Network changes in unexpected ways after the game starts.

To deduplicate we should follow this process: 
1. Make sure each game has a unique ID
2. Exclude rows where `game_time` is not a valid time like 10:00 (see the top four rows above for `game_time` which have the value 'Final'), indicating that the game was past kickoff when the data was pulled
3. Sort the dataframe by `current_datetime` descending and take the first row to get the most recent row before kickoff for each game.

Although `game_id` is unique ID when present it is also null around 2/3 of the time rendering it unusable as a unique ID for the whole dataset. Let's create a unique game ID for the whole dataset by concatinating columns with no null values:
- `rot_num_home` is a weekly unique ID from Action Network. These can be recycled week over week, so this should be used in conjunction with a datetime column
- `team_concat` is a concatination of the team names playing that week
- `organization` indicates if the game is being played between college or NFL teams. It's probably not necessary to use but acts as insurance just in case `rot_num_home` is unique only to college or NFL games within a given week
- `iso_week` indicates the week of the year the game was played in (we will create this below)
- `year` indicates the year the game was played in

A concatination of all these columns will create a unique ID for each game. Then we can deduplicate the dataset against this ID as described above to leave us with the one row per game we have that is closest to kickoff for that game.

First let's extract some useful values from the `current_datetime` column and create new columns including `iso_week` which we will use to create `game_id`.


```python
# add useful columns
df['current_datetime'] = pd.to_datetime(df['current_datetime'])
df['year'] = df['current_datetime'].dt.year
df['month'] = df['current_datetime'].dt.month
df['iso_week'] = df['current_datetime'].dt.isocalendar().week.astype(int).astype(str)
```

Create game ID.


```python
# game id: same game across scrapes, regardless of line movement
# rot_num_home recycles weekly but team_concat disambiguates within a week
df['game_id'] = (df['rot_num_home'].astype(str) + '_' + 
                 df['team_concat'] + '_' + 
                 df['organization'] + '_' + 
                 df['iso_week'] + '_' +
                 df['year'].astype(str)
                )
```

Drop rows where game has already started.


```python
# drop rows where game has already started
mask = df['game_time'].str.match(r'^(?:[01]\d|2[0-3]):[0-5]\d$')
df = df[mask]
```


```python
print(f"Duplicates before dedupe: {df['game_id'].duplicated().sum()}")
```

    Duplicates before dedupe: 10772



```python
# remove duplicates, keeping the latest scrape before game starts
df = (
    df.sort_values(
        ['game_id', 'current_datetime'], ascending=[True, False]).drop_duplicates(subset='game_id', keep='first')
)
```


```python
print(f"Remaining duplicates: {df['game_id'].duplicated().sum()}")
print(f"Shape after dedup: {df.shape}")
print(f"College: {(df['organization'] == 'College').sum()}, NFL: {(df['organization'] == 'NFL').sum()}")
```

    Remaining duplicates: 0
    Shape after dedup: (2707, 41)
    College: 1932, NFL: 775


Great! We went from over 10k duplicate rows down to 0. Our predictive dataset is starting to looking good.

The next step is to get the results dataset and join it to our predictive dataset. This is going to be challenging because:
1. The team names are unlikely to perfectly match in all cases across both datasets (e.g. one dataset might have North Carolina State and the other might have North Carolina St. or NCST)
2. We do not have game dates in our predictive dataset (we only have the dates we pulled the data, not the dates the games were actually played)

We also still need to determine the actual date on which each game was played. This will be easier once we have joined the predictive and results datasets because the results dataset will have the correct game dates.

Once we have joined the datasets, we can use a combination of team name, organization (college or NFL), and the rough timeframe in which the game was played to determine the true game date in our Office Football Pool dataset. Only then can we join the OFP dataset with the results dataset.

### Prepare College Football and NFL dataframes

The college football season runs from August until January. Bowl games are played in January of the following year but are considered by of the previous year's season. 

The NFL is similar with games being played from August until the Super Bowl the following February.

Let's create a season column that groups games played in January or February into the season for the previous year and then divide our datasets into College and NFL.


```python
# derive season: football seasons span aug-jan, so jan/feb scrapes belong to prior year's season
df['season'] = df['current_datetime'].dt.year
df.loc[df['current_datetime'].dt.month <= 2, 'season'] -= 1

# split into college and nfl for separate processing
df_college = df[df['organization'] == 'College'].copy()
df_nfl = df[df['organization'] == 'NFL'].copy()

print(f"College games: {len(df_college)}, seasons: {sorted(df_college['season'].unique())}")
print(f"NFL games: {len(df_nfl)}, seasons: {sorted(df_nfl['season'].unique())}")
```

    College games: 1932, seasons: [2023, 2024, 2025]
    NFL games: 775, seasons: [2023, 2024, 2025]


## Get College Football Game Scores

Now were ready to pull game result data for College and NFL. We'll start with College pull by retrieving college football results data from https://collegefootballdata.com/ which offers a free API.

https://api.collegefootballdata.com/libraries/python has detailed documentation for how to pull game data using python. The next cell is copied an pasted directly from their documentation.

I already saved my API key from collegefootballdata as a variable on my hard drive.


```python
import os
import cfbd
from cfbd.rest import ApiException

configuration = cfbd.Configuration(
    access_token=os.environ['CFBD_API_KEY'],
)

try:
    with cfbd.ApiClient(configuration) as api_client:
        games_api = cfbd.GamesApi(api_client)
        games = games_api.get_games(year=2023, team='Michigan')
        if games:
            print(games[0])
except ApiException as error:
    print(f'College Football Data API request failed: {error}')
```

    id=401520162 season=2023 week=1 season_type=<SeasonType.REGULAR: 'regular'> start_date=datetime.datetime(2023, 9, 2, 16, 0, tzinfo=datetime.timezone.utc) start_time_tbd=False completed=True neutral_site=False conference_game=False attendance=109480 venue_id=3558 venue='Michigan Stadium' home_id=130 home_team='Michigan' home_conference='Big Ten' home_classification=<DivisionClassification.FBS: 'fbs'> home_points=30 home_line_scores=[7, 16, 7, 0] home_postgame_win_probability=0.9985490969984152 home_pregame_elo=1916 home_postgame_elo=1941 away_id=151 away_team='East Carolina' away_conference='American Athletic' away_classification=<DivisionClassification.FBS: 'fbs'> away_points=3 away_line_scores=[0, 0, 0, 3] away_postgame_win_probability=0.0014509030015847912 away_pregame_elo=1506 away_postgame_elo=1481 excitement_index=1.1721121152 highlights='' notes=None


Since the API successfully returned game data for the Michigan 2023 season, let's use it to return data for all games for all teams for all seasons in our college football dataframe.

The API returns a space-separated list of key-value pairs. For each game, we'll create a dictionary of the key-value pairs and we'll load each game (dictionary) into a list. 


```python
# get sorted college football years
seasons = sorted(set(df_college['season']))
```


```python
with cfbd.ApiClient(configuration) as api_client:
    games_api = cfbd.GamesApi(api_client)
    
    # fetch games for all seasons in our data
    all_cfbd_games = []
    for season in seasons:
        games = games_api.get_games(year=int(season))
        all_cfbd_games.extend([g.to_dict() for g in games])
    
print(f"Fetched {len(all_cfbd_games)} CFBD games")
```

    Fetched 11366 CFBD games


Let's take a look at the first dictionary in our list.


```python
all_cfbd_games[0]
```




    {'id': 401525434,
     'season': 2023,
     'week': 1,
     'seasonType': <SeasonType.REGULAR: 'regular'>,
     'startDate': datetime.datetime(2023, 8, 26, 18, 30, tzinfo=datetime.timezone.utc),
     'startTimeTBD': False,
     'completed': True,
     'neutralSite': True,
     'conferenceGame': False,
     'attendance': 49000,
     'venueId': 3504,
     'venue': 'Aviva Stadium',
     'homeId': 87,
     'homeTeam': 'Notre Dame',
     'homeConference': 'FBS Independents',
     'homeClassification': <DivisionClassification.FBS: 'fbs'>,
     'homePoints': 42,
     'homeLineScores': [14, 14, 7, 7],
     'homePostgameWinProbability': 0.998958343504533,
     'homePregameElo': 1733,
     'homePostgameElo': 1819,
     'awayId': 2426,
     'awayTeam': 'Navy',
     'awayConference': 'American Athletic',
     'awayClassification': <DivisionClassification.FBS: 'fbs'>,
     'awayPoints': 3,
     'awayLineScores': [0, 0, 0, 3],
     'awayPostgameWinProbability': 0.0010416564954669472,
     'awayPregameElo': 1471,
     'awayPostgameElo': 1385,
     'excitementIndex': 1.3469076611,
     'highlights': '',
     'notes': None}



The API returned a lot of fields, most of which we probably do not need for this project. The fields that seem obviously useful are: id, season, week, seasonType, startDate, homeTeam, awayTeam, homePoints, awayPoints. Let's create a college football dataframe keeping only these fields. 

Let's also convert startDate to datetime format and convert the camelCase to snake_case for consistency since we've been using snake_case.


```python
df_cfbd = pd.DataFrame(all_cfbd_games)

# keep useful fields
keep = ["id", "season", "week", "seasonType", "startDate",
        "homeTeam", "awayTeam", "homePoints", "awayPoints"]
df_cfbd = df_cfbd[[c for c in keep if c in df_cfbd.columns]]

# rename to snake_case for consistency
df_cfbd = df_cfbd.rename(columns={
    'seasonType': 'season_type',
    'startDate': 'start_date', 
    'homeTeam': 'home_team',
    'awayTeam': 'away_team',
    'homePoints': 'home_points',
    'awayPoints': 'away_points'
})

# parse start_date — strip timezone if present
df_cfbd['start_date'] = pd.to_datetime(df_cfbd['start_date'], utc=True).dt.tz_localize(None)
df_cfbd['home_team_lower'] = df_cfbd['home_team'].str.lower()
df_cfbd['away_team_lower'] = df_cfbd['away_team'].str.lower()
```

Let's look at how many non-null values we have for each field in our college football results dataframe.


```python
df_cfbd.isna().sum()
```




    id                 0
    season             0
    week               0
    season_type        0
    start_date         0
    home_team          0
    away_team          0
    home_points        8
    away_points        7
    home_team_lower    0
    away_team_lower    0
    dtype: int64



A few games have null values for home or away scores. These are not useful to us and the rows should be dropped.


```python
# drop games without scores
df_cfbd = df_cfbd.dropna(subset=['home_points', 'away_points'])
```

### Map Office Football Pool team names to CFBD

Now that we have our college football results dataframe we need to join it to our Office Football Pool college football dataframe. 

To create the OFP dataframe, I had created a key to match Office Football Pool team names and Action Network team names. I did this by using a fuzzy matching library to match as many similar team names as possible and then manually matching any teams as needed. I will use a similar approach here. Matching names can be a pain because different platforms can use very different naming systems which fuzzy matching can't always handle so it can require a decent amount of manual review.

Ultimately we want to build a key that joins every team name in our college football predictve dataset to our college football results dataset.


```python
# get all unique OFP college team names
college_teams_all_ofp = set(df_college['away_team']) | set(df_college['home_team'])

print(f"OFP college teams: {len(college_teams_all_ofp)}")
```

    OFP college teams: 136


`team_names.csv` is the key matches that matches Office Football Pool team names and Action Network team names.


```python
# load team_names.csv: action network team -> office football pool team
team_names_csv = pd.read_csv('team_names.csv')
team_names_csv.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>action network team</th>
      <th>office football pool team</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>49ers</td>
      <td>san francisco</td>
    </tr>
    <tr>
      <th>1</th>
      <td>air force</td>
      <td>air force</td>
    </tr>
    <tr>
      <th>2</th>
      <td>akron</td>
      <td>akron</td>
    </tr>
    <tr>
      <th>3</th>
      <td>alabama</td>
      <td>alabama</td>
    </tr>
    <tr>
      <th>4</th>
      <td>app state</td>
      <td>appalachian state</td>
    </tr>
  </tbody>
</table>
</div>



Let's turn `team_names.csv` into a dictionary with OFP team name matched to action network team name.


```python
# create OFP -> action network team names dictionary
ofp_to_an = dict(zip(
    team_names_csv['office football pool team'].str.lower().str.strip(),
    team_names_csv['action network team'].str.lower().str.strip()
))

print(f"team_names.csv mappings: {len(ofp_to_an)}")
```

    team_names.csv mappings: 159


If you're wondering why there are more `team_names.csv` mappings than OFP college teams it is because Action Network uses multiple names for the same team like with the San Diego State below.


```python
pd.options.display.max_rows= 189
team_names_csv.loc[team_names_csv['office football pool team'] == 'san diego st.']
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>action network team</th>
      <th>office football pool team</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>115</th>
      <td>san diego st</td>
      <td>san diego st.</td>
    </tr>
    <tr>
      <th>183</th>
      <td>sdsu</td>
      <td>san diego st.</td>
    </tr>
  </tbody>
</table>
</div>



Now let's create a set of team names from our college football results dataset.


```python
# build CFBD lookup sets
cfbd_schools = set(df_cfbd['home_team_lower']) | set(df_cfbd['away_team_lower'])

print(f"CFBD schools: {len(cfbd_schools)}")
```

    CFBD schools: 734


Ok now to match. We ultimately want to end up with a CFBD name for every OFP name. 

We will do this in stages:
1. For each OFP team name check to see if the name matches the CFBD name perfectly
2. For each AN team name check to see if the name matches the CFBD name perfectly (since we already have a mapping of OFP name -> AN name)
3. Fuzzy match OFP team name to CFBD team name
4. Manually match OFP team names that did not cleanly match (we'll check all teams that were matched)


```python
# import fuzzy match library
from rapidfuzz import process, fuzz

# create mapping dictionary and match list
ofp_to_cfbd_mapping = {}
ofp_to_cfbd_matched = []

for ofp_name in college_teams_all_ofp:
    # check if OFP name is already a CFBD game name
    if ofp_name in cfbd_schools:
        ofp_to_cfbd_mapping[ofp_name] = ofp_name
        continue

    # try action network name as intermediary
    # look up action network name and see if it matches to the cfdb name exactly and if so use that mapping
    an_name = ofp_to_an.get(ofp_name, ofp_name)
    if an_name in cfbd_schools:
        ofp_to_cfbd_mapping[ofp_name] = an_name
        continue

    # fuzzy match against CFBD game names
    match, score, _ = process.extractOne(ofp_name, cfbd_schools, scorer=fuzz.token_sort_ratio)
    ofp_to_cfbd_mapping[ofp_name] = match
    ofp_to_cfbd_matched.append((ofp_name, an_name, match, score))
```

Now that we have done all the programmatic matching, let's take a look at the the team names that were fuzzy matched to see which matches are incorrect and need to be manually overriden.


```python
# use match list to create dataframe and then sort dataframe by match score ascending
cfbd_matches = pd.DataFrame(ofp_to_cfbd_matched, columns=['ofp_name', 'an_name', 'match', 'score'])
cfbd_matches.sort_values(by='score')
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ofp_name</th>
      <th>an_name</th>
      <th>match</th>
      <th>score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>8</th>
      <td>fiu</td>
      <td>fiu</td>
      <td>furman</td>
      <td>44.444444</td>
    </tr>
    <tr>
      <th>10</th>
      <td>miami ohio</td>
      <td>miami oh</td>
      <td>ohio dominican</td>
      <td>66.666667</td>
    </tr>
    <tr>
      <th>4</th>
      <td>miami fla</td>
      <td>miami (fl)</td>
      <td>miami</td>
      <td>71.428571</td>
    </tr>
    <tr>
      <th>18</th>
      <td>san jose st.</td>
      <td>san jose st</td>
      <td>san josé state</td>
      <td>76.923077</td>
    </tr>
    <tr>
      <th>24</th>
      <td>florida atl.</td>
      <td>fl atlantic</td>
      <td>florida atlantic</td>
      <td>78.571429</td>
    </tr>
    <tr>
      <th>2</th>
      <td>boise st.</td>
      <td>boise</td>
      <td>st. ambrose</td>
      <td>80.000000</td>
    </tr>
    <tr>
      <th>13</th>
      <td>middle tenn</td>
      <td>mid tenn</td>
      <td>middle tennessee</td>
      <td>81.481481</td>
    </tr>
    <tr>
      <th>22</th>
      <td>fresno st.</td>
      <td>fresno</td>
      <td>fresno state</td>
      <td>81.818182</td>
    </tr>
    <tr>
      <th>21</th>
      <td>oregon st.</td>
      <td>beavers</td>
      <td>oregon state</td>
      <td>81.818182</td>
    </tr>
    <tr>
      <th>19</th>
      <td>kansas st.</td>
      <td>k state</td>
      <td>kansas state</td>
      <td>81.818182</td>
    </tr>
    <tr>
      <th>11</th>
      <td>s. florida</td>
      <td>s. florida</td>
      <td>florida</td>
      <td>82.352941</td>
    </tr>
    <tr>
      <th>20</th>
      <td>northern ill</td>
      <td>n. illinois</td>
      <td>northern illinois</td>
      <td>82.758621</td>
    </tr>
    <tr>
      <th>5</th>
      <td>arizona st.</td>
      <td>arizona st</td>
      <td>arizona state</td>
      <td>83.333333</td>
    </tr>
    <tr>
      <th>6</th>
      <td>colorado st.</td>
      <td>colorado st</td>
      <td>colorado state</td>
      <td>84.615385</td>
    </tr>
    <tr>
      <th>16</th>
      <td>michigan st.</td>
      <td>michigan st</td>
      <td>michigan state</td>
      <td>84.615385</td>
    </tr>
    <tr>
      <th>12</th>
      <td>kennesaw st.</td>
      <td>kennesaw st</td>
      <td>kennesaw state</td>
      <td>84.615385</td>
    </tr>
    <tr>
      <th>25</th>
      <td>arkansas st.</td>
      <td>arkansas st</td>
      <td>arkansas state</td>
      <td>84.615385</td>
    </tr>
    <tr>
      <th>3</th>
      <td>oklahoma st.</td>
      <td>ok state</td>
      <td>oklahoma state</td>
      <td>84.615385</td>
    </tr>
    <tr>
      <th>0</th>
      <td>western mich</td>
      <td>w. michigan</td>
      <td>western michigan</td>
      <td>85.714286</td>
    </tr>
    <tr>
      <th>14</th>
      <td>san diego st.</td>
      <td>sdsu</td>
      <td>san diego state</td>
      <td>85.714286</td>
    </tr>
    <tr>
      <th>9</th>
      <td>central mich</td>
      <td>c. michigan</td>
      <td>central michigan</td>
      <td>85.714286</td>
    </tr>
    <tr>
      <th>1</th>
      <td>eastern mich</td>
      <td>e mich</td>
      <td>eastern michigan</td>
      <td>85.714286</td>
    </tr>
    <tr>
      <th>15</th>
      <td>new mexico st.</td>
      <td>nm state</td>
      <td>new mexico state</td>
      <td>86.666667</td>
    </tr>
    <tr>
      <th>7</th>
      <td>washington st.</td>
      <td>wash st</td>
      <td>washington state</td>
      <td>86.666667</td>
    </tr>
    <tr>
      <th>26</th>
      <td>florida st.</td>
      <td>florida st</td>
      <td>west florida</td>
      <td>86.956522</td>
    </tr>
    <tr>
      <th>17</th>
      <td>mississippi st.</td>
      <td>mississippi st</td>
      <td>mississippi state</td>
      <td>87.500000</td>
    </tr>
    <tr>
      <th>23</th>
      <td>hawaii</td>
      <td>hawaii</td>
      <td>hawai'i</td>
      <td>92.307692</td>
    </tr>
  </tbody>
</table>
</div>



It looks like there are five OFP college football team names that did not fuzzy match the name in the CFDB team names correctly: 
- fiu
- miami ohio
- boise st.
- s. florida
- florida st.

We will use a regular expression to look up versions of each of these incorrectly matched names in the CFDB.


```python
import re 

ofp_unmatched_reg = ['fiu', 'miami ohio', 'boise st.', 's. florida', 'florida st.']

# find names in CFDB that are similar to the incorrectly matched OFP names
def find_matches(ofp_unmatched_reg):
    r = re.compile(ofp_unmatched_reg)
    newlist = (list(filter(r.match, cfbd_schools)))
    print(newlist)
    
ofp_unmatched_regs = ['.*florida.*', 'miami.*', 'boise.*']

for ofp_unmatched_reg in ofp_unmatched_regs:
    find_matches(ofp_unmatched_reg)
```

    ['florida a&m', 'florida international', 'florida state', 'florida', 'florida memorial university', 'florida atlantic', 'south florida', 'west florida']
    ['miami (oh)', 'miami']
    ['boise state']


Now let's create a dictionary with the correct OFP name to CFDB matches for these and then update the existing dictionary with the incorrect mappings.


```python
# create manual override of ofp to cfdb team name mapping and update mapping dict
ofp_to_cfbd_mapping_manual = {
    'fiu': 'florida international',
    'miami ohio': 'miami (oh)',
    'boise st.': 'boise state',
    's. florida': 'south florida',
    'florida st.': 'florida state',
}

ofp_to_cfbd_mapping.update(ofp_to_cfbd_mapping_manual)
```

Let's quickly check to make sure that all OFP team names have a CFBD mapping


```python
# review all mappings sorted alphabetically
mapping_review = pd.DataFrame(
    [(k, v) for k, v in sorted(ofp_to_cfbd_mapping.items())],
    columns=['ofp_name', 'cfbd_name']
)
print(f"Total mappings: {len(mapping_review)}")
print(f"Unmapped (None): {mapping_review['cfbd_name'].isna().sum()}")
mapping_review.head()
```

    Total mappings: 136
    Unmapped (None): 0





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ofp_name</th>
      <th>cfbd_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>air force</td>
      <td>air force</td>
    </tr>
    <tr>
      <th>1</th>
      <td>akron</td>
      <td>akron</td>
    </tr>
    <tr>
      <th>2</th>
      <td>alabama</td>
      <td>alabama</td>
    </tr>
    <tr>
      <th>3</th>
      <td>appalachian state</td>
      <td>app state</td>
    </tr>
    <tr>
      <th>4</th>
      <td>arizona</td>
      <td>arizona</td>
    </tr>
  </tbody>
</table>
</div>



Looks good. Now we need to join the OFP data and the CFDB data.

### Join College OFP Data to CFBD Scores

To join the College OFP data and the College CFBD Score data we should to join the datasets on as many fields as possible to reduce duplicates: home team, away team, and season. This will result in duplicates only if the teams played each other more than once during the same season and the same team was the home team multiple times (should be rare, but we will certainly investigate possible duplicates). First we should use our mapper to add the CFBD team names to the OFP college dataset, then we can use the CFBD names to join the CFDB data.


```python
# map ofp team names to cfdb names
df_college['away_cfbd'] = df_college['away_team'].map(ofp_to_cfbd_mapping)
df_college['home_cfbd'] = df_college['home_team'].map(ofp_to_cfbd_mapping)

# check for unmapped teams
print(f"Count of rows without a cfdb away team name: {len(df_college[df_college['away_cfbd'].isna()])}")
print(f"Count of rows without a cfdb home team name: {len(df_college[df_college['home_cfbd'].isna()])}")
```

    Count of rows without a cfdb away team name: 0
    Count of rows without a cfdb home team name: 0


Now we join the CFBD data to the OFP data to create a new dataframe.


```python
# join OFP college data to CFBD scores

df_cfbd_to_merge = df_cfbd[['week', 'start_date', 'home_team_lower', 'away_team_lower', 'home_points', 'away_points', 'season']]

df_college_scored = df_college.merge(
    df_cfbd_to_merge,
    left_on=['away_cfbd', 'home_cfbd', 'season'],
    right_on=['away_team_lower', 'home_team_lower', 'season'],
    how='left'
)

print(f"initial length of df_college_scored: {len(df_college_scored)}")
```

    initial length of df_college_scored: 1955


Let's check for duplicated `game_id`s after the join.


```python
print(f"number of duplicated games: {len(df_college_scored.loc[df_college_scored.duplicated(subset=['game_id'])])}")
```

    number of duplicated games: 23


Let's take a closer look the first couple of those 23 duplicated game IDs and figure out if they are differentiated.


```python
# look at duplicated game_id examples
df_college_scored.loc[df_college_scored.duplicated(subset=['game_id'], keep=False)].head(4)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>rot_num_away</th>
      <th>rot_num_home</th>
      <th>ofp_game_id</th>
      <th>team_concat</th>
      <th>organization</th>
      <th>game_time</th>
      <th>away_team</th>
      <th>away_open_line</th>
      <th>away_current_best_line</th>
      <th>ofp_away_line</th>
      <th>away_odds</th>
      <th>away_bet_percentage</th>
      <th>home_team</th>
      <th>home_open_line</th>
      <th>home_current_best_line</th>
      <th>ofp_home_line</th>
      <th>home_odds</th>
      <th>home_bet_percentage</th>
      <th>max_free_points</th>
      <th>max_free_points_team</th>
      <th>line_movement_team</th>
      <th>reverse_line_movement</th>
      <th>circadian_rhythm</th>
      <th>away_team_time_zone_offset</th>
      <th>home_team_time_zone_offset</th>
      <th>free_points_strength</th>
      <th>line_movement_strength</th>
      <th>line_movement_%_bets_value_away_team</th>
      <th>line_movement_%_bets_value_home_team</th>
      <th>free_points_value_away_team</th>
      <th>free_points_value_home_team</th>
      <th>circadian_rhythm_points_away</th>
      <th>circadian_rhythm_points_home</th>
      <th>recommendation_team</th>
      <th>current_datetime</th>
      <th>max_recommendation_points</th>
      <th>net_recommendation_points</th>
      <th>year</th>
      <th>month</th>
      <th>iso_week</th>
      <th>game_id</th>
      <th>season</th>
      <th>away_cfbd</th>
      <th>home_cfbd</th>
      <th>week</th>
      <th>start_date</th>
      <th>home_team_lower</th>
      <th>away_team_lower</th>
      <th>home_points</th>
      <th>away_points</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>103</td>
      <td>104</td>
      <td>None</td>
      <td>kennesaw st.jacksonville state</td>
      <td>College</td>
      <td>16:00</td>
      <td>kennesaw st.</td>
      <td>-1.5</td>
      <td>-2.5</td>
      <td>-2.5</td>
      <td>-110</td>
      <td>54.0</td>
      <td>jacksonville state</td>
      <td>1.5</td>
      <td>2.5</td>
      <td>2.5</td>
      <td>-108</td>
      <td>46.0</td>
      <td>0.0</td>
      <td>None</td>
      <td>kennesaw st.</td>
      <td>None</td>
      <td>None</td>
      <td>NaN</td>
      <td>NaN</td>
      <td></td>
      <td>very weak</td>
      <td>0.0</td>
      <td>0.25</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>jacksonville state</td>
      <td>2025-12-05 12:08:37.218314</td>
      <td>0.25</td>
      <td>None</td>
      <td>2025</td>
      <td>12</td>
      <td>49</td>
      <td>104_kennesaw st.jacksonville state_College_49_...</td>
      <td>2025</td>
      <td>kennesaw state</td>
      <td>jacksonville state</td>
      <td>12.0</td>
      <td>2025-11-16 01:00:00</td>
      <td>jacksonville state</td>
      <td>kennesaw state</td>
      <td>35.0</td>
      <td>26.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>103</td>
      <td>104</td>
      <td>None</td>
      <td>kennesaw st.jacksonville state</td>
      <td>College</td>
      <td>16:00</td>
      <td>kennesaw st.</td>
      <td>-1.5</td>
      <td>-2.5</td>
      <td>-2.5</td>
      <td>-110</td>
      <td>54.0</td>
      <td>jacksonville state</td>
      <td>1.5</td>
      <td>2.5</td>
      <td>2.5</td>
      <td>-108</td>
      <td>46.0</td>
      <td>0.0</td>
      <td>None</td>
      <td>kennesaw st.</td>
      <td>None</td>
      <td>None</td>
      <td>NaN</td>
      <td>NaN</td>
      <td></td>
      <td>very weak</td>
      <td>0.0</td>
      <td>0.25</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>jacksonville state</td>
      <td>2025-12-05 12:08:37.218314</td>
      <td>0.25</td>
      <td>None</td>
      <td>2025</td>
      <td>12</td>
      <td>49</td>
      <td>104_kennesaw st.jacksonville state_College_49_...</td>
      <td>2025</td>
      <td>kennesaw state</td>
      <td>jacksonville state</td>
      <td>15.0</td>
      <td>2025-12-06 00:00:00</td>
      <td>jacksonville state</td>
      <td>kennesaw state</td>
      <td>15.0</td>
      <td>19.0</td>
    </tr>
    <tr>
      <th>38</th>
      <td>109</td>
      <td>110</td>
      <td>None</td>
      <td>unlvboise st.</td>
      <td>College</td>
      <td>17:00</td>
      <td>unlv</td>
      <td>3.5</td>
      <td>5.5</td>
      <td>3.5</td>
      <td>-110</td>
      <td>65.0</td>
      <td>boise st.</td>
      <td>-3.5</td>
      <td>-5.0</td>
      <td>-3.5</td>
      <td>-110</td>
      <td>35.0</td>
      <td>1.5</td>
      <td>boise st.</td>
      <td>boise st.</td>
      <td>boise st.</td>
      <td>None</td>
      <td>-3.0</td>
      <td>-2.0</td>
      <td>very weak</td>
      <td>very weak</td>
      <td>0.0</td>
      <td>0.75</td>
      <td>0.0</td>
      <td>3.0</td>
      <td>0</td>
      <td>0</td>
      <td>boise st.</td>
      <td>2025-12-05 12:08:37.218314</td>
      <td>3.75</td>
      <td>None</td>
      <td>2025</td>
      <td>12</td>
      <td>49</td>
      <td>110_unlvboise st._College_49_2025</td>
      <td>2025</td>
      <td>unlv</td>
      <td>boise state</td>
      <td>8.0</td>
      <td>2025-10-18 19:30:00</td>
      <td>boise state</td>
      <td>unlv</td>
      <td>56.0</td>
      <td>31.0</td>
    </tr>
    <tr>
      <th>39</th>
      <td>109</td>
      <td>110</td>
      <td>None</td>
      <td>unlvboise st.</td>
      <td>College</td>
      <td>17:00</td>
      <td>unlv</td>
      <td>3.5</td>
      <td>5.5</td>
      <td>3.5</td>
      <td>-110</td>
      <td>65.0</td>
      <td>boise st.</td>
      <td>-3.5</td>
      <td>-5.0</td>
      <td>-3.5</td>
      <td>-110</td>
      <td>35.0</td>
      <td>1.5</td>
      <td>boise st.</td>
      <td>boise st.</td>
      <td>boise st.</td>
      <td>None</td>
      <td>-3.0</td>
      <td>-2.0</td>
      <td>very weak</td>
      <td>very weak</td>
      <td>0.0</td>
      <td>0.75</td>
      <td>0.0</td>
      <td>3.0</td>
      <td>0</td>
      <td>0</td>
      <td>boise st.</td>
      <td>2025-12-05 12:08:37.218314</td>
      <td>3.75</td>
      <td>None</td>
      <td>2025</td>
      <td>12</td>
      <td>49</td>
      <td>110_unlvboise st._College_49_2025</td>
      <td>2025</td>
      <td>unlv</td>
      <td>boise state</td>
      <td>15.0</td>
      <td>2025-12-06 01:00:00</td>
      <td>boise state</td>
      <td>unlv</td>
      <td>38.0</td>
      <td>21.0</td>
    </tr>
  </tbody>
</table>
</div>



The `start_date` is different for each duplicated `game_id`. It appears that the teams played each other during the regular season and again in the conference championship or a bowl game.


```python
# look at select columns from duplicated game_id examples
df_college_scored.loc[df_college_scored.duplicated(subset=['game_id'], keep=False), ['game_id', 'start_date', 'current_datetime']].sort_values(by='game_id').head(4)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>game_id</th>
      <th>start_date</th>
      <th>current_datetime</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>104_kennesaw st.jacksonville state_College_49_...</td>
      <td>2025-11-16 01:00:00</td>
      <td>2025-12-05 12:08:37.218314</td>
    </tr>
    <tr>
      <th>4</th>
      <td>104_kennesaw st.jacksonville state_College_49_...</td>
      <td>2025-12-06 00:00:00</td>
      <td>2025-12-05 12:08:37.218314</td>
    </tr>
    <tr>
      <th>38</th>
      <td>110_unlvboise st._College_49_2025</td>
      <td>2025-10-18 19:30:00</td>
      <td>2025-12-05 12:08:37.218314</td>
    </tr>
    <tr>
      <th>39</th>
      <td>110_unlvboise st._College_49_2025</td>
      <td>2025-12-06 01:00:00</td>
      <td>2025-12-05 12:08:37.218314</td>
    </tr>
  </tbody>
</table>
</div>



We should keep the game with the closest start_date to `current_datetime` field from the OFP dataset and drop the other duplicate row. 


```python
# drop rows with duplicated game_id, keeping row where start_date and current_datetime are closest
df_college_scored['days_diff'] = abs(df_college_scored['start_date'] - df_college_scored['current_datetime'])

# sort dataframe by days_diff
df_college_scored = df_college_scored.sort_values(by='days_diff')

# drop rows with duplicated game_id with higher days_diff
df_college_scored = df_college_scored.loc[~df_college_scored.duplicated(subset=['game_id'], keep='first')]

print(f"length of df_college_scored after filtering duplicates: {len(df_college_scored)}")
print(f"number of duplicated games: {len(df_college_scored.loc[df_college_scored.duplicated(subset=['game_id'])])}")
```

    length of df_college_scored after filtering duplicates: 1932
    number of duplicated games: 0


As a QA measure we should only keep games that have an scraped timestamp within about 7 days of kickoff. Let's sort the dataframe by `days_diff` descending to investigate this.


```python
df_college_scored.sort_values(by='days_diff', ascending=False).head(10)['days_diff']
```




    993   3 days 23:43:53.268012
    991   3 days 08:13:53.268012
    988   3 days 04:43:53.268012
    995   2 days 16:43:15.450454
    792   2 days 16:29:41.957635
    772   2 days 15:29:41.957635
    584   2 days 14:59:41.957635
    996   2 days 14:34:46.300045
    675   2 days 14:29:41.957635
    756   2 days 12:59:41.957635
    Name: days_diff, dtype: timedelta64[ns]



Perfect. It looks like no games data was scraped more than 4 days away from kickoff.

Lastly, we should drop any rows with null values for home or away points.


```python
# drop games with with missing points data
df_college_scored = df_college_scored.dropna(subset=['home_points', 'away_points'])

print(f"length of df_college_scored after filtering out games with null points values: {len(df_college_scored)}")
```

    length of df_college_scored after filtering out games with null points values: 1930


Since our college dataframe has columns we do not need for this project, let's only keep columns that seem like they will useful to us.


```python
cols_to_keep = ['organization', 
                'week',
                'away_team',
                'away_open_line', 
                'away_current_best_line', 
                'ofp_away_line', 
                'away_odds', 
                'away_bet_percentage',
                'home_team',
                'home_open_line',
                'home_current_best_line', 
                'ofp_home_line', 
                'home_odds',
                'home_bet_percentage',
                'max_free_points',
                'max_free_points_team',
                'max_recommendation_points',
                'recommendation_team',
                'game_id',
                'season',
                'start_date',
                'home_points',
                'away_points']

df_college_scored_final = df_college_scored[cols_to_keep].copy()
```

Ok great, our college dataframe now has predictive (OFP) and results data and has been pared down to only include the columns we need to get started building our model! We will still want to do some feature engineering before building our model but it makes sense to prepare our NFL dataframe first so we can combine our datasets and then build out our features. Let's prepare our NFL dataframe.

## Get NFL Game Scores

There is an API that can pull in NFL game data called `nfl_data_py`. According to [the nfl_data_py documentation](https://pypi.org/project/nfl-data-py/):

> `nfl.import_schedules(years)`
>
> Returns dataframe with schedule information for years specified
>
> years : required, list of years to pull data for (earliest available is 1999)

It is not clear if `year` refers to calendar year or season. Let's investigate by pulling in data for one year and seeing if the dataset contains games played outside the calendar year. This will also give us the opportunity to see which fields will be returns by the NFL API and how the team names are formated.


```python
import nfl_data_py as nfl

df_nfl_scores_2023 = nfl.import_schedules(years=[2023])

df_nfl_scores_2023.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>game_id</th>
      <th>season</th>
      <th>game_type</th>
      <th>week</th>
      <th>gameday</th>
      <th>weekday</th>
      <th>gametime</th>
      <th>away_team</th>
      <th>away_score</th>
      <th>home_team</th>
      <th>home_score</th>
      <th>location</th>
      <th>result</th>
      <th>total</th>
      <th>overtime</th>
      <th>old_game_id</th>
      <th>gsis</th>
      <th>nfl_detail_id</th>
      <th>pfr</th>
      <th>pff</th>
      <th>espn</th>
      <th>ftn</th>
      <th>away_rest</th>
      <th>home_rest</th>
      <th>away_moneyline</th>
      <th>home_moneyline</th>
      <th>spread_line</th>
      <th>away_spread_odds</th>
      <th>home_spread_odds</th>
      <th>total_line</th>
      <th>under_odds</th>
      <th>over_odds</th>
      <th>div_game</th>
      <th>roof</th>
      <th>surface</th>
      <th>temp</th>
      <th>wind</th>
      <th>away_qb_id</th>
      <th>home_qb_id</th>
      <th>away_qb_name</th>
      <th>home_qb_name</th>
      <th>away_coach</th>
      <th>home_coach</th>
      <th>referee</th>
      <th>stadium_id</th>
      <th>stadium</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>6421</th>
      <td>2023_01_DET_KC</td>
      <td>2023</td>
      <td>REG</td>
      <td>1</td>
      <td>2023-09-07</td>
      <td>Thursday</td>
      <td>20:20</td>
      <td>DET</td>
      <td>21.0</td>
      <td>KC</td>
      <td>20.0</td>
      <td>Home</td>
      <td>-1.0</td>
      <td>41.0</td>
      <td>0.0</td>
      <td>2023090700</td>
      <td>59173.0</td>
      <td>NaN</td>
      <td>202309070kan</td>
      <td>NaN</td>
      <td>401547353</td>
      <td>NaN</td>
      <td>7</td>
      <td>7</td>
      <td>164.0</td>
      <td>-198.0</td>
      <td>4.0</td>
      <td>-110.0</td>
      <td>-110.0</td>
      <td>53.0</td>
      <td>-110.0</td>
      <td>-110.0</td>
      <td>0</td>
      <td>outdoors</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>00-0033106</td>
      <td>00-0033873</td>
      <td>Jared Goff</td>
      <td>Patrick Mahomes</td>
      <td>Dan Campbell</td>
      <td>Andy Reid</td>
      <td>John Hussey</td>
      <td>KAN00</td>
      <td>GEHA Field at Arrowhead Stadium</td>
    </tr>
    <tr>
      <th>6422</th>
      <td>2023_01_CAR_ATL</td>
      <td>2023</td>
      <td>REG</td>
      <td>1</td>
      <td>2023-09-10</td>
      <td>Sunday</td>
      <td>13:00</td>
      <td>CAR</td>
      <td>10.0</td>
      <td>ATL</td>
      <td>24.0</td>
      <td>Home</td>
      <td>14.0</td>
      <td>34.0</td>
      <td>0.0</td>
      <td>2023091000</td>
      <td>59174.0</td>
      <td>NaN</td>
      <td>202309100atl</td>
      <td>NaN</td>
      <td>401547403</td>
      <td>NaN</td>
      <td>7</td>
      <td>7</td>
      <td>160.0</td>
      <td>-192.0</td>
      <td>3.5</td>
      <td>-108.0</td>
      <td>-112.0</td>
      <td>40.5</td>
      <td>-110.0</td>
      <td>-110.0</td>
      <td>1</td>
      <td>closed</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>00-0039150</td>
      <td>00-0038122</td>
      <td>Bryce Young</td>
      <td>Desmond Ridder</td>
      <td>Frank Reich</td>
      <td>Arthur Smith</td>
      <td>Brad Rogers</td>
      <td>ATL97</td>
      <td>Mercedes-Benz Stadium</td>
    </tr>
    <tr>
      <th>6423</th>
      <td>2023_01_HOU_BAL</td>
      <td>2023</td>
      <td>REG</td>
      <td>1</td>
      <td>2023-09-10</td>
      <td>Sunday</td>
      <td>13:00</td>
      <td>HOU</td>
      <td>9.0</td>
      <td>BAL</td>
      <td>25.0</td>
      <td>Home</td>
      <td>16.0</td>
      <td>34.0</td>
      <td>0.0</td>
      <td>2023091001</td>
      <td>59175.0</td>
      <td>NaN</td>
      <td>202309100rav</td>
      <td>NaN</td>
      <td>401547396</td>
      <td>NaN</td>
      <td>7</td>
      <td>7</td>
      <td>380.0</td>
      <td>-500.0</td>
      <td>9.5</td>
      <td>-110.0</td>
      <td>-110.0</td>
      <td>43.5</td>
      <td>-110.0</td>
      <td>-110.0</td>
      <td>0</td>
      <td>outdoors</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>00-0039163</td>
      <td>00-0034796</td>
      <td>C.J. Stroud</td>
      <td>Lamar Jackson</td>
      <td>DeMeco Ryans</td>
      <td>John Harbaugh</td>
      <td>Tra Blake</td>
      <td>BAL00</td>
      <td>M&amp;T Bank Stadium</td>
    </tr>
    <tr>
      <th>6424</th>
      <td>2023_01_CIN_CLE</td>
      <td>2023</td>
      <td>REG</td>
      <td>1</td>
      <td>2023-09-10</td>
      <td>Sunday</td>
      <td>13:00</td>
      <td>CIN</td>
      <td>3.0</td>
      <td>CLE</td>
      <td>24.0</td>
      <td>Home</td>
      <td>21.0</td>
      <td>27.0</td>
      <td>0.0</td>
      <td>2023091002</td>
      <td>59176.0</td>
      <td>NaN</td>
      <td>202309100cle</td>
      <td>NaN</td>
      <td>401547397</td>
      <td>NaN</td>
      <td>7</td>
      <td>7</td>
      <td>-112.0</td>
      <td>-108.0</td>
      <td>-1.0</td>
      <td>-105.0</td>
      <td>-115.0</td>
      <td>46.5</td>
      <td>-110.0</td>
      <td>-110.0</td>
      <td>1</td>
      <td>outdoors</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>00-0036442</td>
      <td>00-0033537</td>
      <td>Joe Burrow</td>
      <td>Deshaun Watson</td>
      <td>Zac Taylor</td>
      <td>Kevin Stefanski</td>
      <td>Clete Blakeman</td>
      <td>CLE00</td>
      <td>FirstEnergy Stadium</td>
    </tr>
    <tr>
      <th>6425</th>
      <td>2023_01_JAX_IND</td>
      <td>2023</td>
      <td>REG</td>
      <td>1</td>
      <td>2023-09-10</td>
      <td>Sunday</td>
      <td>13:00</td>
      <td>JAX</td>
      <td>31.0</td>
      <td>IND</td>
      <td>21.0</td>
      <td>Home</td>
      <td>-10.0</td>
      <td>52.0</td>
      <td>0.0</td>
      <td>2023091003</td>
      <td>59177.0</td>
      <td>NaN</td>
      <td>202309100clt</td>
      <td>NaN</td>
      <td>401547404</td>
      <td>NaN</td>
      <td>7</td>
      <td>7</td>
      <td>-205.0</td>
      <td>170.0</td>
      <td>-4.0</td>
      <td>-108.0</td>
      <td>-112.0</td>
      <td>45.5</td>
      <td>-110.0</td>
      <td>-110.0</td>
      <td>1</td>
      <td>closed</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>00-0036971</td>
      <td>00-0039164</td>
      <td>Trevor Lawrence</td>
      <td>Anthony Richardson</td>
      <td>Doug Pederson</td>
      <td>Shane Steichen</td>
      <td>Clay Martin</td>
      <td>IND00</td>
      <td>Lucas Oil Stadium</td>
    </tr>
  </tbody>
</table>
</div>



Some fields are available through the API that seem useful are `game_id`, `season`, `week`, `gameday`, `away_team`, `away_score`, `home_team`, and `home_score`.

Let's sort by the `gameday` column descending and see what the most recent gameday is.


```python
df_nfl_scores_2023.sort_values(by='gameday', ascending=False).head(5)['gameday']
```




    6705    2024-02-11
    6704    2024-01-28
    6703    2024-01-28
    6702    2024-01-21
    6701    2024-01-21
    Name: gameday, dtype: object



The 2023 season includes postseason games played in 2024 so `year` = `season` (not calendar year) and we can pull data from this API for the NFL seasons for which we have predictive data. Let's do that and then prune the dataframe for the columns we've flagged as being potentially useful and then take a look at the null values in the dataframe and data types for our columns.


```python
# fetch NFL scores using nfl_data_py
nfl_seasons = sorted(df_nfl['season'].unique())
df_nfl_scores = nfl.import_schedules(nfl_seasons)
```


```python
# keep relevant columns
df_nfl_scores = df_nfl_scores[['game_id', 'season', 'game_type', 'week', 'gameday', 
                                'away_team', 'home_team', 'away_score', 'home_score']].copy()

df_nfl_scores.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 855 entries, 6421 to 7275
    Data columns (total 9 columns):
     #   Column      Non-Null Count  Dtype  
    ---  ------      --------------  -----  
     0   game_id     855 non-null    object 
     1   season      855 non-null    int64  
     2   game_type   855 non-null    object 
     3   week        855 non-null    int64  
     4   gameday     855 non-null    object 
     5   away_team   855 non-null    object 
     6   home_team   855 non-null    object 
     7   away_score  855 non-null    float64
     8   home_score  855 non-null    float64
    dtypes: float64(2), int64(2), object(5)
    memory usage: 66.8+ KB


No null values to worry about. We should convert `gameday` to a `datetime` object.


```python
# convert gameday to datetime
df_nfl_scores['gameday'] = pd.to_datetime(df_nfl_scores['gameday'])

df_nfl_scores.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>game_id</th>
      <th>season</th>
      <th>game_type</th>
      <th>week</th>
      <th>gameday</th>
      <th>away_team</th>
      <th>home_team</th>
      <th>away_score</th>
      <th>home_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>6421</th>
      <td>2023_01_DET_KC</td>
      <td>2023</td>
      <td>REG</td>
      <td>1</td>
      <td>2023-09-07</td>
      <td>DET</td>
      <td>KC</td>
      <td>21.0</td>
      <td>20.0</td>
    </tr>
    <tr>
      <th>6422</th>
      <td>2023_01_CAR_ATL</td>
      <td>2023</td>
      <td>REG</td>
      <td>1</td>
      <td>2023-09-10</td>
      <td>CAR</td>
      <td>ATL</td>
      <td>10.0</td>
      <td>24.0</td>
    </tr>
    <tr>
      <th>6423</th>
      <td>2023_01_HOU_BAL</td>
      <td>2023</td>
      <td>REG</td>
      <td>1</td>
      <td>2023-09-10</td>
      <td>HOU</td>
      <td>BAL</td>
      <td>9.0</td>
      <td>25.0</td>
    </tr>
    <tr>
      <th>6424</th>
      <td>2023_01_CIN_CLE</td>
      <td>2023</td>
      <td>REG</td>
      <td>1</td>
      <td>2023-09-10</td>
      <td>CIN</td>
      <td>CLE</td>
      <td>3.0</td>
      <td>24.0</td>
    </tr>
    <tr>
      <th>6425</th>
      <td>2023_01_JAX_IND</td>
      <td>2023</td>
      <td>REG</td>
      <td>1</td>
      <td>2023-09-10</td>
      <td>JAX</td>
      <td>IND</td>
      <td>31.0</td>
      <td>21.0</td>
    </tr>
  </tbody>
</table>
</div>



### Map Office Football Pool team names to NFL Scores
`away_team` and `home_team` are represented as an (annoying) airport-code style abbreviations. Fuzzymatch would really struggle to match these abbreviations to the longer OFP and AN names. Thankfully there are only 32 NFL teams so creating a mapping dictionary manually won't be too difficult.


```python
# OFP NFL team name -> nfl_data_py abbreviation mapping
ofp_nfl_to_abbr = {
    'arizona': 'ARI',
    'atlanta': 'ATL',
    'baltimore': 'BAL',
    'buffalo': 'BUF',
    'carolina': 'CAR',
    'chicago': 'CHI',
    'cincinnati': 'CIN',
    'cleveland': 'CLE',
    'dallas': 'DAL',
    'denver': 'DEN',
    'detroit': 'DET',
    'green bay': 'GB',
    'houston': 'HOU',
    'indianapolis': 'IND',
    'jacksonville': 'JAX',
    'kansas city': 'KC',
    'la chargers': 'LAC',
    'la rams': 'LA',
    'las vegas': 'LV',
    'miami': 'MIA',
    'minnesota': 'MIN',
    'new england': 'NE',
    'new orleans': 'NO',
    'ny giants': 'NYG',
    'ny jets': 'NYJ',
    'philadelphia': 'PHI',
    'pittsburgh': 'PIT',
    'san francisco': 'SF',
    'seattle': 'SEA',
    'tampa bay': 'TB',
    'tennessee': 'TEN',
    'washington': 'WAS',
}
```

Let's map the abbreviated team names to the OFP team names and verify that all the OFP team names have a mapped abbreviation.


```python
# map OFP names to abbreviations
df_nfl['away_abbr'] = df_nfl['away_team'].map(ofp_nfl_to_abbr)
df_nfl['home_abbr'] = df_nfl['home_team'].map(ofp_nfl_to_abbr)

print(f"Count of rows without an away abbreviation: {len(df_nfl[df_nfl['away_abbr'].isna()])}")
print(f"Count of rows without a home abbreviation: {len(df_nfl[df_nfl['home_abbr'].isna()])}")
```

    Count of rows without an away abbreviation: 0
    Count of rows without a home abbreviation: 0


Great, all OFP team names mapped to an abbreviation.

### Join OFP Data to NFL Scores

Now we should follow the same process we did with the college datasets above:
1. Join the predictive and results datasets
2. Filter out games with duplicated game_id
3. Filter out games with missing score data

We should also verify that all games are played within 7 days of the data scrape.


```python
# join NFL OFP data to scores using team abbreviations
df_nfl_scored = df_nfl.merge(
    df_nfl_scores,
    left_on=['away_abbr', 'home_abbr', 'season'],
    right_on=['away_team', 'home_team', 'season'],
    how='left',
    suffixes=('', '_scores')
)

print(f"initial length of df_nfl_scored: {len(df_nfl_scored)}")
```

    initial length of df_nfl_scored: 787



```python
# drop rows with duplicated game_id, keeping row where gameday and current_datetime are closest
df_nfl_scored['days_diff'] = abs(df_nfl_scored['gameday'] - df_nfl_scored['current_datetime'])

# sort dataframe by days_diff
df_nfl_scored = df_nfl_scored.sort_values(by=['days_diff'])

# drop rows with duplicated game_id with higher days_diff
df_nfl_scored = df_nfl_scored.loc[~df_nfl_scored.duplicated(subset=['game_id'], keep='first')]

print(f"length of df_nfl_scored after filtering duplicates: {len(df_nfl_scored)}")
```

    length of df_nfl_scored after filtering duplicates: 775



```python
# drop missing scores
df_nfl_scored = df_nfl_scored.dropna(subset=['home_team_scores', 'away_team_scores'])

print(f"length of df_nfl_scored after dropping missing scores: {len(df_nfl_scored)}")
```

    length of df_nfl_scored after dropping missing scores: 775



```python
# check max difference between current datetime and actual game date
df_nfl_scored.sort_values(by=['days_diff'], ascending=False).head(10)['days_diff']
```




    405   4 days 07:43:53.268012
    425   4 days 07:43:53.268012
    413   4 days 07:43:53.268012
    389   4 days 07:43:53.268012
    369   4 days 07:43:53.268012
    371   4 days 07:43:53.268012
    391   4 days 07:43:53.268012
    381   4 days 07:43:53.268012
    427   4 days 07:43:53.268012
    397   4 days 07:43:53.268012
    Name: days_diff, dtype: timedelta64[ns]



Now we're ready to combine our College and NFL datasets
## Combine College and NFL data

To prepare the NFL dataset to be combined with the college dataset we need to filter the NFL dataset columns to match the college dataset columns.


```python
# rename nfl columns to match college 
df_nfl_scored = df_nfl_scored.rename(columns={
    'away_score': 'away_points',
    'home_score': 'home_points',
    'gameday': 'start_date'
})

# filter nfl dataframe to contain college dataframe columns only
df_nfl_scored_final = df_nfl_scored[cols_to_keep].copy()
```

Now we can combine the NFL and college datasets.


```python
# combine nfl and college datasets
df_all = pd.concat([df_college_scored_final, df_nfl_scored_final], ignore_index=True)

df_all.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>organization</th>
      <th>week</th>
      <th>away_team</th>
      <th>away_open_line</th>
      <th>away_current_best_line</th>
      <th>ofp_away_line</th>
      <th>away_odds</th>
      <th>away_bet_percentage</th>
      <th>home_team</th>
      <th>home_open_line</th>
      <th>home_current_best_line</th>
      <th>ofp_home_line</th>
      <th>home_odds</th>
      <th>home_bet_percentage</th>
      <th>max_free_points</th>
      <th>max_free_points_team</th>
      <th>max_recommendation_points</th>
      <th>recommendation_team</th>
      <th>game_id</th>
      <th>season</th>
      <th>start_date</th>
      <th>home_points</th>
      <th>away_points</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>College</td>
      <td>1.0</td>
      <td>penn st.</td>
      <td>-1.5</td>
      <td>2.5</td>
      <td>3.5</td>
      <td>-108</td>
      <td>46.0</td>
      <td>clemson</td>
      <td>1.5</td>
      <td>-2.0</td>
      <td>-3.5</td>
      <td>-115</td>
      <td>54.0</td>
      <td>1.0</td>
      <td>penn st.</td>
      <td>64.25</td>
      <td>penn st.</td>
      <td>228_penn st.clemson_College_52_2025</td>
      <td>2025</td>
      <td>2025-12-27 17:00:00</td>
      <td>10.0</td>
      <td>22.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>College</td>
      <td>3.0</td>
      <td>boston college</td>
      <td>17.5</td>
      <td>14.5</td>
      <td>16.5</td>
      <td>-110</td>
      <td>48.0</td>
      <td>missouri</td>
      <td>-17.5</td>
      <td>-14.5</td>
      <td>-16.5</td>
      <td>-105</td>
      <td>52.0</td>
      <td>2.0</td>
      <td>boston college</td>
      <td>5.50</td>
      <td>boston college</td>
      <td>130_boston collegemissouri_College_37_2024</td>
      <td>2024</td>
      <td>2024-09-14 16:45:00</td>
      <td>27.0</td>
      <td>21.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>College</td>
      <td>10.0</td>
      <td>memphis</td>
      <td>-10.5</td>
      <td>-7.0</td>
      <td>-7.5</td>
      <td>-110</td>
      <td>76.0</td>
      <td>utsa</td>
      <td>10.5</td>
      <td>7.0</td>
      <td>7.5</td>
      <td>-105</td>
      <td>24.0</td>
      <td>0.5</td>
      <td>utsa</td>
      <td>12.00</td>
      <td>utsa</td>
      <td>358_memphisutsa_College_44_2024</td>
      <td>2024</td>
      <td>2024-11-02 16:00:00</td>
      <td>44.0</td>
      <td>36.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>College</td>
      <td>10.0</td>
      <td>stanford</td>
      <td>12.5</td>
      <td>9.5</td>
      <td>9.5</td>
      <td>-104</td>
      <td>35.0</td>
      <td>no carolina st.</td>
      <td>-12.5</td>
      <td>-10.0</td>
      <td>-9.5</td>
      <td>-105</td>
      <td>65.0</td>
      <td>0.5</td>
      <td>no carolina st.</td>
      <td>2.25</td>
      <td>stanford</td>
      <td>356_stanfordno carolina st._College_44_2024</td>
      <td>2024</td>
      <td>2024-11-02 16:00:00</td>
      <td>59.0</td>
      <td>28.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>College</td>
      <td>10.0</td>
      <td>toledo</td>
      <td>-6.0</td>
      <td>-10.0</td>
      <td>-8.5</td>
      <td>-110</td>
      <td>46.0</td>
      <td>eastern mich</td>
      <td>6.0</td>
      <td>9.5</td>
      <td>8.5</td>
      <td>+100</td>
      <td>54.0</td>
      <td>1.5</td>
      <td>toledo</td>
      <td>15.00</td>
      <td>toledo</td>
      <td>342_toledoeastern mich_College_44_2024</td>
      <td>2024</td>
      <td>2024-11-02 16:00:00</td>
      <td>28.0</td>
      <td>29.0</td>
    </tr>
  </tbody>
</table>
</div>



Quick gut check to make sure the College dataset still has 1930 rows and the NFL dataset has 775 rows.


```python
df_all['organization'].value_counts()
```




    organization
    College    1930
    NFL         775
    Name: count, dtype: int64



Looks good! 
### Calculate spread coverage

Next we should determine which team covered the pointspread for each match.


```python
# home_actual_margin is the margin by which the home team won, if positive, or lost, if negative
df_all['home_actual_margin'] = df_all['home_points'] - df_all['away_points']
# home_spread_cover is the number of points the home team covered by, if positive, or failed to cover by, if negative
df_all['home_spread_cover'] = df_all['home_actual_margin'] + df_all['ofp_home_line']
# home_covers is true if the home team won by an amount greater than the point spread
df_all['home_covers'] = df_all['home_spread_cover'] > 0
```

Since we make picks every week, we need to separate our dataset into weeks. The NFL season typically starts one week after the College Football season does, and this is true for the seasons in our dataset. This means that NFL week 1 = College week 2. 

We should make a `cannonical_week` column based off the NFL week schedule where all matches played during that matchweek will have the same number.

Complicating matters is that the College Football results dataset restarts the week count for bowl games. 

Another consideration is that NFL weeks typically start on a Thursday and end on the following Monday. However, sometimes games can be played on other days of the week. College games can be played on any day of the week. Most importantly, our pick four league sets the week as Tuesday - Monday so we will do the same.

Our pick four league starts on NFL week 1 and ends after NFL week 18. Games played outside that window, like some College bowl games, should be dropped.

For NFL games `cannonical_week` = `week`. Then we will map college football games onto this based on their game date.

A clean way to do set the cannonical week for every College game would be to:
1. For each NFL week find the min `start_date`
2. Set the adjusted min date for each week to the Tuesday immediately before that NFL week (unless week already starts on a Tuesday)
3. Set the max date equal to the next Monday after the min date
4. Drop College games that are not assigned a cannonical week (because they are outside the acceptable date range)

Let's use this approach.


```python
# set cannonical week equal to NFL week Tuesday - Monday
from datetime import timedelta

df_all.loc[df_all['organization'] == 'NFL', 'cannonical_week'] = df_all['week']

weeks = set(df_all.loc[df_all['organization'] == 'NFL', 'week'])

for season in seasons:
    for week in weeks:
        dt_min = df_all.loc[(
            df_all['organization'] == 'NFL') & (
            df_all['season'] == season) & (
            df_all['week'] == week), 'start_date'].min()
        
        if dt_min is None:
            continue
            
        try:
            # week starts on Tuesday and ends on Monday
            weekday = dt_min.weekday()
            week_start = dt_min - timedelta(weekday - 1)
            week_end = week_start + timedelta(6)

            df_all.loc[(
                df_all['organization'] == 'College') & (
                df_all['season'] == season) & (
                df_all['start_date'].between(
                    week_start, week_end)
            ), 'cannonical_week'] = week
        
        except:
            continue
        
# drop null cannonical weeks
df_all = df_all.dropna(subset='cannonical_week')
        
# set cannonical season week
df_all['cannonical_week'] = df_all['cannonical_week'].astype('int')

# add leading zeros to single digit cannonical season weeks to help with sorting
df_all['cannonical_week_str'] = df_all['cannonical_week'].astype('str')
df_all.loc[df_all['cannonical_week'] < 10, 'cannonical_week_str'] = df_all.loc[df_all['cannonical_week'] < 10, 'cannonical_week_str'].str.zfill(2)
df_all['cannonical_season_week'] = df_all['season'].astype('str') + '_' + df_all['cannonical_week_str']

df_all = df_all.drop(columns=['cannonical_week_str'])
```

Every remaining game should have a `cannonical_week` and a `cannonical_season_week`. Let's verify that.


```python
print(f"cannonical week has {len(df_all.loc[df_all['cannonical_week'].isna()])} null values")
print(f"cannonical week has {len(df_all.loc[df_all['cannonical_season_week'].isna()])} null values")
```

    cannonical week has 0 null values
    cannonical week has 0 null values


And let's check how many games we dropped.


```python
df_all['organization'].value_counts()
```




    organization
    College    1919
    NFL         775
    Name: count, dtype: int64



We dropped 11 college games which is about what you would expect given that some bowl games are played during the NFL playoffs. No NFL games were dropped.

Now we're ready to start picking games. For each cannonincal week we need to make exactly four picks against the spread. 

## Predict Game Outcomes
### Determine an accuracy metric
Since the rules of Pick 4 state that we must pick exactly four games each week we only need to evaluate the accuracy of our best four picks per week. So our accuracy metric should be for each week:
1. Take the four predictions we believe have the highest likelihood of being correct
2. Determine if each prediction was correct, asssigning 1 for each correct prediction and 0 for each incorrect prediction
3. Divide the sum predictions by the total number of games (four per week * number of weeks)

This will give us an accuracy rate that we can use to compare prediction methodologies.

### Use a simple picking heuristic

Let's start by using a simple heuristic to make picks and to set a baseline accuracy rate. 

The simple heuristic will pick the top four teams each week with the most free points. Ties will be broken by picking the team chosen less frequently by the public (sports betting rule: fade the public). The second tie breaker will be to simply select the home team.

We'll try to predict whether or not the home team covers the spread and then evaluate our accuracy using our accuracy metric above.

Let's start by creating a copy of our dataframe. 

Later will we compare the performance of this simple heuristic against the more complex heuristic that I've actually been using to pick games.


```python
# create copy of df for the simple heuristic
df_all_simple = df_all.copy()
```

Create simple game picking heuristic.


```python
# create min_bet_percentage column which has the lowest public bet percentage for either team for that game
df_all_simple['min_bet_percentage'] = df_all_simple[['away_bet_percentage', 'home_bet_percentage']].min(axis=1)

# choose team with more free points to cover
df_all_simple.loc[df_all_simple[
    'max_free_points_team'] == df_all_simple['home_team'], 'simple_home_cover_pred'] = True
df_all_simple.loc[df_all_simple[
    'max_free_points_team'] == df_all_simple['away_team'], 'simple_home_cover_pred'] = False

# use away bet percentage ascending to fill na with False (away team covers)
df_all_simple.loc[(
    df_all_simple['simple_home_cover_pred'].isna()) & (
    df_all_simple['away_bet_percentage'] < 50), 'simple_home_cover_pred'] = False

# get rid of warning message
pd.set_option('future.no_silent_downcasting', True)

# fill remaining na with True (home team covers)
df_all_simple['simple_home_cover_pred'] = df_all_simple['simple_home_cover_pred'].fillna(
    True).infer_objects(copy=False)
```

Let's record if the simple heursitic was accurate for each game.


```python
# record if simple recommendation was accurate for each game
df_all_simple['simple_pred_accurate'] = 0
df_all_simple.loc[df_all['home_covers'] == df_all_simple['simple_home_cover_pred'], 'simple_pred_accurate'] = 1
```

Since we have multiple heursitics to compare (this simple heursitic and next the more complex heuristic), let's create some functions to save from having to repeat ourselves.

Best practice to have each function do one thing (which makes it easier to debug) and then combine functions to complete the task.

We'll make four functions which will do the following:
1. Sort dataframe by cannonical week, then by columns we use to make predictions
2. Select the top four teams by cannonical week
3. Calculate the performance of a given heuristic
4. Run the first three functions 


```python
# create a function to sort df by cannonical_season_week asc first, then prediction score col(s) desc
def sort_picks(df, score_cols, ascending=None):
    '''
    Sort df by cannonical_season_week asc first, then prediction score col(s) desc.
    
    Args:
        df (pandas dataframe)
        score_cols (list): list of score columns to be used to sort dataframe
        ascending (boolean list): optional list to sort score columns, defaults to False
        
    Returns a sorted dataframe.
    '''
    if ascending is None:
        ascending = [False for n in score_cols]
    # sort cannonical_season_week ascending
    score_cols.insert(0, 'cannonical_season_week')
    ascending.insert(0, True)
    
    # sort by score_cols
    sorted_df = df.sort_values(by=score_cols, ascending=ascending).copy()
    
    return sorted_df

# create function to get top 4 recommendation picks by week
def get_weekly_top_four(df):
    '''
    Returns pandas dataframe with top four picks by season week.
        
    '''
    return df.groupby('cannonical_season_week').head(4)

# create function to calculate performance of heuristic at picking four games per week
def correct_picks(df, accuracy_col, description="X heuristic"):
    '''
    Calculate performance of heuristic top four picks by week.
    
    Args: 
        df (pandas dataframe): contains cannonical_season_week and accuracy cols
        accuracy_col (column of type int): 1 for correct pick, 0 for incorrect
        description (str): description of heuristic 
        
    Returns series of number of correct picks by week and rate of all correct picks.
    '''
        
    # calculate sum of correct picks each week
    weekly = df.groupby('cannonical_season_week')[accuracy_col].sum()
    
    # calculate correct pick percentage across all weeks
    picks_correct_cnt = df.groupby('cannonical_season_week')[accuracy_col].sum().sum()
    total_opportunities = len(df.groupby('cannonical_season_week')) * 4

    correct_rate = (f"{description} got {round(picks_correct_cnt / total_opportunities * 100,2)}% of picks correct.")
    
    return weekly, correct_rate

def make_predictions(df, score_cols, ascending, accuracy_col, description):
    '''
    Call functions sort_picks, get_weekly_top_four, and correct_picks.
    
    Accepts:
        df (pandas dataframe)
        score_cols (list): list of score columns to be used to sort dataframe
        ascending (boolean list): optional list to sort score columns, defaults to False
        accuracy_col (column of type int): 1 for correct pick, 0 for incorrect
        description (str): description of heuristic 
    
    Returns a dataframe with the top four picks each season week, the number of correct picks by season week, the rate of correct picks
    '''
    
    sort_picks_result = sort_picks(
        df, score_cols=score_cols, ascending=ascending)
    
    get_weekly_top_four_result = get_weekly_top_four(sort_picks_result)
    
    weekly_correct_picks, correct_rate = correct_picks(
        get_weekly_top_four_result, 
        accuracy_col=accuracy_col, 
        description=description)
    
    return get_weekly_top_four_result, weekly_correct_picks, correct_rate
```


```python
# make predictions
df_all_simple_season_week, simple_weekly, simple_correct_rate = make_predictions(
    df=df_all_simple,
    score_cols=['max_free_points', 'min_bet_percentage'], 
    ascending=[False, True], 
    accuracy_col='simple_pred_accurate', 
    description='Simple heuristic')

simple_weekly.head()
```




    cannonical_season_week
    2023_05    2
    2023_06    1
    2023_07    4
    2023_08    3
    2023_09    4
    Name: simple_pred_accurate, dtype: int64




```python
print(simple_correct_rate)
```

    Simple heuristic got 60.0% of picks correct.


Our simple heuristic got 60% of picks correct across all cannonical weeks. That's pretty impressive. 

Later, when we use machine learning to pick games, we'll holdout the 2025 season to test our algorithm. To set a baseline, let's figure out what percentage of picks our simple heuristic got for the 2025 season specifically.


```python
# caclulate correct percent of picks for 2025 season
df_all_simple_season_week_test = df_all_simple_season_week.loc[df_all_simple_season_week['season'] == 2025]

_, simple_correct_rate_test = correct_picks(df_all_simple_season_week_test, 'simple_pred_accurate', 'Simple heuristic 2025 season')

print(simple_correct_rate_test)
```

    Simple heuristic 2025 season got 52.78% of picks correct.


A fairly large drop in performance for the 2025 season. 52.78% is our 2025 season baseline. Let's see if we can beat that metric using the more complex heuristic.

### Use a more complex picking heuristic
I programmed a more complex heuristic which attemped to mimic my thought process for picking games. The complex heuristic assigns points if a team has free points (recall that free points are the difference between the OFP line and the live line, our opportunity for arbitrage) that cross a [key number](https://www.bettingnews.com/nfl/guides/key-numbers/) (e.g. 3, the value of a field goal and the most common margin of victory, or 7, the value of a touchdown and an extra point) or combinations of key numbers (3 + 7, 7 + 7, etc) or if the point spread has [reverse line movement](https://www.actionnetwork.com/education/reverse-line-movement).

Each week, I would choose the four teams with the most complex heuristic points. 

I could write a whole post about how I designed this heuristic but for brevity sake I'll just share a screenshot of part of the function I made so you can get an idea. I spent a lot of team fine tuning this heuristic and following its recommendations have won me the pick four league multiple seasons.

![image.png](image.png)

As before let's start by creating a copy of our dataframe.

Then for each game we'll pick whichever team has more complex heuristic points (the complex heuristic points were saved in the database and are in the dataframe so we won't need to calculate it again now). 

Then we'll calculate the performance of the complex heurstic, just as we did with the simple heuristic. 


```python
# create copy of df for the complex heuristic
df_all_complex = df_all.copy()
```


```python
# determine if home team complex recommendations cover point spread
df_all_complex['complex_rec_home_cover_pred'] = False
df_all_complex.loc[df_all['recommendation_team'] == df_all_complex['home_team'], 'complex_rec_home_cover_pred'] = True

# determine if complex recommendation was accurate
df_all_complex['complex_rec_accurate'] = 0
df_all_complex.loc[df_all['home_covers'] == df_all_complex['complex_rec_home_cover_pred'], 'complex_pred_accurate'] = 1
```


```python
# get top 4 complex recommendation picks by week
df_all_complex_season_week = get_weekly_top_four(
        sort_picks(
            df_all_complex, score_cols=[
                'max_recommendation_points'], ascending=[
                False]))
```


```python
# calculate accuracy of complex heuristic 
complex_weekly, complex_correct_rate = correct_picks(
    df_all_complex_season_week, accuracy_col='complex_pred_accurate', description='Complex heuristic')

complex_weekly.head()
```




    cannonical_season_week
    2023_05    3.0
    2023_06    2.0
    2023_07    3.0
    2023_08    3.0
    2023_09    4.0
    Name: complex_pred_accurate, dtype: float64




```python
print(complex_correct_rate)
```

    Complex heuristic got 64.5% of picks correct.



```python
# caclulate correct percent of picks for 2025 season
df_all_complex_season_week_test = df_all_complex_season_week.loc[df_all_complex_season_week['season'] == 2025]

_, complex_correct_rate_test = correct_picks(df_all_complex_season_week_test, 'complex_pred_accurate', 'Complex heuristic 2025 season')

print(complex_correct_rate_test)
```

    Complex heuristic 2025 season got 66.67% of picks correct.


The complex heuristic was a huge perfomance improvement over the simple heuristic overall and for the 2025 season. Our new baseline for 2025 is 66.67% (2/3 correct picks), a blistering pace! 

Let's see if we can beat this using machine learning.

## Use Machine Learning to Pick Games
Our goal is to train a machine learning algorithm on seasons prior to 2025 and then test it on the 2025 season to determine performance. We can create features (also called parameters or predictors) and adjust hyperparameters to try to improve performance.

As we did with the heuristics, we will choose only four games per cannonical week. So we will need to choose an algorithm classification algorithm that also returns a probability score that we can use to sort. Logistic regression seems like a good choice. 

Let's start by creating a copy of our dataframe.


```python
# create copy of df for machine learning
df_lr = df_all.copy()
```

Any columns that we use for machine learning cannot have null values.


```python
# fill na rows 
df_lr['home_bet_percentage'] = df_lr['home_bet_percentage'].fillna(50)
df_lr['away_bet_percentage'] = df_lr['away_bet_percentage'].fillna(50)
```

Let's generate a max free points feature for the home team and the away team and convert the odds column from an object to an integer.


```python
# create home and away free points columns
df_lr.loc[df_lr['max_free_points_team'] == df_lr[
    'away_team'], 'away_free_points'] = df_lr.loc[df_lr[
    'max_free_points_team'] == df_lr['away_team'], 'max_free_points']

df_lr.loc[df_lr['max_free_points_team'] == df_lr[
    'home_team'], 'home_free_points'] = df_lr.loc[df_lr[
    'max_free_points_team'] == df_lr['home_team'], 'max_free_points']

df_lr['away_free_points'] = df_lr['away_free_points'].fillna(0)
df_lr['home_free_points'] = df_lr['home_free_points'].fillna(0)
```


```python
# convert odds columns from objects to integers
df_lr['away_odds'] = df_lr['away_odds'].astype('int')
df_lr['home_odds'] = df_lr['home_odds'].astype('int')
```

Now let's create a train dataframe using data from seasons before 2025 and a test dataframe using data from the 2025 season. Then we'll select our predictors and train our model on the train dataframe in order to make predictions on the test dataframe.


```python
# split data into training and test data where training data comes before test data
train = df_lr.loc[df_lr['season'] < 2025].copy()
test = df_lr.loc[df_lr['season'] == 2025].copy()
```


```python
# set predictor columns
predictors = ['away_open_line', 
              'away_current_best_line', 
              'ofp_away_line', 
              'away_odds', 
              'away_bet_percentage', 
              'home_odds', 
              'away_free_points',
              'home_free_points'
             ]

# create X and y train
X_train = train[predictors]
y_train = train['home_covers']
```


```python
# train lr model 
from sklearn.linear_model import LogisticRegression
clf = LogisticRegression().fit(X_train, y_train)
```


```python
# create X and y test
X_test = test[predictors]
y_test = test['home_covers']

# use trained lr model to create predictions on test data
pred = clf.predict(X_test)
pred_probs = clf.predict_proba(X_test)
```

We'll create a column that takes the maximum probability score returned by the logisitic regression algorithm which we'll use to sort the dataframe and make our selections for each cannonical week.


```python
import numpy as np

# create array of max probability for each game
pred_probs_max = np.amax(pred_probs, axis=1)

# add predictions and prediction probabilities columns to test df
test['lr_preds'] = pred
test['lr_pred_probs'] = pred_probs_max

# Determine accuracy of lr model for each game
test['lr_preds_accurate'] = 0
test.loc[test['home_covers'] == test['lr_preds'], 'lr_preds_accurate'] = 1

# get top 4 lr recommendation picks by week
df_lr_season_week_test = get_weekly_top_four(
        sort_picks(
            test, score_cols=[
                'lr_pred_probs']))
```


```python
# calculate accuracy of lr

lr_weekly, lr_correct_rate_test = correct_picks(
    df_lr_season_week_test, accuracy_col='lr_preds_accurate', description='lr 2025 season')

lr_weekly.head()
```




    cannonical_season_week
    2025_01    2
    2025_02    4
    2025_03    3
    2025_04    3
    2025_05    1
    Name: lr_preds_accurate, dtype: int64




```python
print(lr_correct_rate_test)
```

    lr 2025 season got 61.11% of picks correct.


Not bad! Our first attempt at using machine learning to generate predictions got 61.11% of picks correct for the 2025 season. Not as impressive as our complex algorithm so let's see if we can tune our algorithm to improve performance. 

### Make Running Logistic Regression Repeatable
To avoid repeating code let's make some functions that will allow us to repeatedly run logistic regression with different parameters and hyperparameters.


```python
# Make functions to help us repeatedly run logistic regression
def create_train_test_df(df=df_lr, year=2025, predictors=predictors, target_col='home_covers', feature_scale=None):
    '''
    Create X and y train and X and y test datasets, splitting train and test on year.
    
    Accepts: 
        df (dataframe) : dataframe containing target_col and predictors
        year (int) : train set season < year, test set season >= year
        predictors (list ): predictors used to create X
        target_col (str) : target column model will attempt to predict
        feature_scale (str) : features will be scaled using chosen sklearn preprocessing.StandardScaler() method
        
    Returns test, X_train, y_train, X_test, y_test dataframes.
    
    '''
    
    df = df.copy()
    
    # create train df
    train = df.loc[df['season'] < year].copy()
    test = df.loc[df['season'] >= year].copy()

    # create X and y train
    X_train = train[predictors]
    y_train = train[target_col]
    
    # create X and y test
    X_test = test[predictors]
    y_test = test[target_col]
    
    # scale features
    if feature_scale is not None:
        
        from sklearn import preprocessing
        
        if feature_scale == 'standard':
        
            scaler = preprocessing.StandardScaler().fit(X_train)
            X_train = scaler.transform(X_train)
            X_test = scaler.transform(X_test)
                
        if feature_scale == 'min_max':

            min_max_scaler = preprocessing.MinMaxScaler()
            X_train = min_max_scaler.fit_transform(X_train)
            X_test = min_max_scaler.transform(X_test)
        
    return test, X_train, y_train, X_test, y_test

def run_logistic_regression(X_train, y_train, X_test, y_test, max_iter=1000, C=1.0, solver='lbfgs'):
    
    '''
    Train a logistic regression model.
    
    Accepts:
        X_train (dataframe)
        y_train (dataframe)
        X_test (dataframe)
        y_test (dataframe)
        max_iter (int) : maximum number of iterations the logisitic regression test can run
        
    Returns arrays of predictions and prediction probabilities.
    '''
    
    # fit model
    clf = LogisticRegression(max_iter=max_iter, C=C, solver=solver).fit(X_train, y_train)

    # use trained lr model create predictions on test data
    pred = clf.predict(X_test)
    pred_probs = clf.predict_proba(X_test)

    return pred, pred_probs

def append_lr_results(df, pred, pred_probs, target_col):
    '''
    Append predictions and prediction probabilities of a logisitic regression model onto a dataframe and 
    determine accuracy of prediction.
    
    Accepts:
        df (dataframe) : dataframe that results will be appended onto, should contain target_col
        pred (array) : An array of predictions
        pred_probs (array) : An array of prediction probabilities produced by sklearn LogisticRegression
        target_col (str) : target column model attempted to predict
        
    Returns dataframe appended with lr_preds, lr_pred_probs, lr_preds_accurate.
    '''
    
    df = df.copy()
    
    # create array of max probability for each game
    import numpy as np
    pred_probs_max = np.amax(pred_probs, axis=1)
    
    # add predictions and prediction probabilities columns to test df
    df['lr_preds'] = pred
    df['lr_pred_probs'] = pred_probs_max
    
    # Determine accuracy of lr model for each game
    df['lr_preds_accurate'] = 0
    df.loc[df[target_col] == df['lr_preds'], 'lr_preds_accurate'] = 1
    
    return df
```

### Use Feature Scaling
If features use different scales (like bet percentage vs free points) it can affect the performance of the machine learning algorithm. Let's scale our features so they are all using the same scale which can improve accuracy of the algorithm. 

There are two approaches that we will test here: 
1. Standardization - Scale values to set the mean to 0 and each standard deviation in either direction to +1 or -1
2. Normalization (min max) - Shift values so that they are between 0 and 1 

We'll test both starting with standardization.


```python
# scale features using standardization
from sklearn import preprocessing

scaler = preprocessing.StandardScaler().fit(X_train)

X_scaled = scaler.transform(X_train)
```

Create test and train datasets


```python
# create test and train datasets
df_test, X_train, y_train, X_test, y_test = create_train_test_df(feature_scale='standard')
```

Train logistic regression using standardized features on train dataset, make predictions on holdout/test dataset (2025 season) and calculate performance.


```python
# run logistic regression
pred_, pred_probs_ = run_logistic_regression(X_train=X_train, y_train=y_train, X_test=X_test, y_test=y_test)
```


```python
# create results dataframe
df_scaled = append_lr_results(df=df_test, pred=pred_, pred_probs=pred_probs_, target_col='home_covers')
```


```python
scaled_lr_weekly_top_four_result_df, scaled_lr_weekly_correct_picks, scaled_lr_correct_rate = make_predictions(
    df=df_scaled, 
    score_cols=['lr_pred_probs'], 
    ascending=[False], 
    accuracy_col='lr_preds_accurate', 
    description='Standardized features'
)

scaled_lr_correct_rate
```




    'Standardized features got 59.72% of picks correct.'



Standardizing features made performance a bit worse. 

Let's create another function that will call the all the logisitic regression helper functions we created above to streamline running logisitic regression.

Then let's test normalizing features.


```python
def full_logistic_regression(df=df_lr, year=2025, predictors=predictors, target_col='home_covers', max_iter=1000, feature_scale=False, C=1.0, solver='lbfgs'):
    '''
    Call functions create_train_test_df, run_logistic_regression, append_lr_results.
    
    Accepts: 
        df (dataframe) : dataframe containing target_col and predictors
        year (int) : train set season < year, test set season >= year
        predictors (list ): predictors used to create X
        target_col (str) : target column model will attempt to predict
        max_iter (int) : maximum number of iterations the logisitic regression test can run
        feature_scale (bool) : if True features will be scaled using sklearn preprocessing.StandardScaler()
        C (float) : inverse of regularization strength
        solver (str) : algorithm to use in the optimization problem
        
    Returns dataframe appended with lr_preds, lr_pred_probs, lr_preds_accurate.
        
    '''
    
    test_df, X_train, y_train, X_test, y_test = create_train_test_df(df=df, year=year, predictors=predictors, target_col=target_col, feature_scale=feature_scale)
    
    pred, pred_probs = run_logistic_regression(X_train=X_train, y_train=y_train, X_test=X_test, y_test=y_test, max_iter=max_iter, C=C, solver=solver)
    
    df = append_lr_results(df=test_df, pred=pred, pred_probs=pred_probs, target_col=target_col)
    
    return pred, pred_probs, df
```


```python
# normalize features using mix max
pred, pred_probs, df_scaled_min_max = full_logistic_regression(feature_scale='min_max')
```


```python
scaled_lr_weekly_top_four_result_df, scaled_lr_weekly_correct_picks, scaled_lr_correct_rate = make_predictions(
    df=df_scaled_min_max, 
    score_cols=['lr_pred_probs'], 
    ascending=[False], 
    accuracy_col='lr_preds_accurate', 
    description='Normalized features'
)

scaled_lr_correct_rate
```




    'Normalized features got 62.5% of picks correct.'



Normalizing features improved performance over baseline slightly but we're still lagging our complex algorithm. Now let's try generating some new features.

### Generate New Features
#### Key Numbers
I mentioned that our complex heuristic leans heavily on free points crossing a key number or combinations of key numbers. Let's create a function that will help us create boolean features based on whether or not a free points crossed a key number for a given team in a given game.


```python
# create functions to help us with feature generation 
def crosses_key_number(df, key_number, description):
    '''
    Create features based on free points crossing a key number.
    
    Args:
        df (pandas dataframe): dataframe containing key number cols
        key_number (int): key number that is crossed
        description (str): description of key number
        
    Returns a dataframe with game_id and the new feature columns.
    '''

    df_tmp = df.copy()
    
    away_underdog_col_name = '_'.join(('away_free_points_crosses',description,'underdog'))
    away_favorite_col_name = '_'.join(('away_free_points_crosses',description,'favorite'))
    home_underdog_col_name = '_'.join(('home_free_points_crosses',description,'underdog'))
    home_favorite_col_name = '_'.join(('home_free_points_crosses',description,'favorite'))
    
    feature_cols = [away_underdog_col_name, away_favorite_col_name, home_underdog_col_name, home_favorite_col_name]
    feature_cols_game_id = feature_cols.copy()
    feature_cols_game_id.insert(0, 'game_id')
    
    df_tmp[away_underdog_col_name] = False
    df_tmp[away_favorite_col_name] = False
    df_tmp[home_underdog_col_name] = False
    df_tmp[home_favorite_col_name] = False
    
    df_tmp.loc[(df_tmp['ofp_away_line'] > key_number) & (df[
        'away_current_best_line'] < key_number)  & (
        df_tmp['home_current_best_line'] > (-1*key_number)), away_underdog_col_name] = True

    df_tmp.loc[(df_tmp['ofp_away_line'] > (-1*key_number)) & (df[
        'away_current_best_line'] < (-1*key_number))  & (
        df_tmp['home_current_best_line'] > key_number), away_favorite_col_name] = True

    df_tmp.loc[(df['ofp_home_line'] > key_number) & (df[
        'home_current_best_line'] < key_number)  & (
        df_tmp['away_current_best_line'] > (-1*key_number)), home_underdog_col_name] = True

    df_tmp.loc[(df_tmp['ofp_home_line'] > (-1*key_number)) & (df[
        'home_current_best_line'] < (-1*key_number))  & (
        df_tmp['away_current_best_line'] > key_number), home_favorite_col_name] = True
    
    return df_tmp[feature_cols_game_id], feature_cols

key_numbers = {
    3 : 'fg',
    6 : 'two_fg',
    7 : 'td',
    10 : 'td_fg',
    14 : 'two_td',
    21 : 'three_td'
}
```

Let's create a new dataframe that includes our new key number features.


```python
# create a new dataframe for our features
df_features = df_lr.copy()
predictors_key_numbers = predictors.copy()

# create feature columns and append feature name to predictors list

for k, v in key_numbers.items():
    df_features_tmp, feature_cols_tmp = crosses_key_number(df_lr, k, v)
    df_features = pd.merge(left=df_features, right=df_features_tmp, on='game_id')
    for feature in feature_cols_tmp:
        predictors_key_numbers.append(feature)
```

Let's run logisitic regression using our new features using feature normalization.


```python
# run logistic regression with key number features
_pred, _pred_probs, df_features_preds = full_logistic_regression(
    df=df_features, 
    predictors=predictors_key_numbers, 
    feature_scale='min_max'
)
```


```python
# calculate pick accuracy of logistic regression with key number features
df_features_season_week, features_weekly, features_correct_rate = make_predictions(
    df=df_features_preds,
    score_cols=['lr_pred_probs'], 
    ascending=[False], 
    accuracy_col='lr_preds_accurate', 
    description='Key Numbers Features Logisitic Regression'
)
```


```python
features_correct_rate
```




    'Key Numbers Features Logisitic Regression got 62.5% of picks correct.'



Adding the new features did not result in a performance improvement. Let's run logistic regression again using the new features but without feature normalization.


```python
# run logistic regression with key number features
_pred, _pred_probs, df_features_preds = full_logistic_regression(
    df=df_features, 
    predictors=predictors_key_numbers, 
#     feature_scale='min_max'
)
```


```python
# calculate pick accuracy of logistic regression with key number features
df_features_season_week, features_weekly, features_correct_rate = make_predictions(
    df=df_features_preds,
    score_cols=['lr_pred_probs'], 
    ascending=[False], 
    accuracy_col='lr_preds_accurate', 
    description='Key Numbers Features Logisitic Regression'
)
```


```python
features_correct_rate
```




    'Key Numbers Features Logisitic Regression got 63.89% of picks correct.'



Removing feature normalization gave us a small boost in performance when using our new features. Let's create some more features that are also included in the complex heuristic.

#### Add Reverse Line Movement Feature
Earlier I mentioned reverse line movement was a factor in the complex heuristic. When a team has a smaller portion of the public bets but the line moves in such a way to make it less advantageous to bet on that team, it is called reverse line movement. It can be an indicator that sharp bettors are betting on that team. The lower the percentage of bets on a team, the more meaningful it is that the line moved against that team. 

Let's create a reverse line movement feature for each decile of public bet percentage below 50%. Ultimately these features will exist as booleans for each decile with terms of art based on the decile, e.g. away_reverse_line_elite if the away team has less than 10% of the public bets and the line moves against the away team.


```python
# create reverse line movement features
predictors_reverse_line = predictors.copy()
predictors_key_numbers_reverse_line = predictors_key_numbers.copy()

reverse_line_dict = {'elite' : 10,
                    'very_strong' : 20,
                    'strong' : 30,
                    'medium' : 40,
                    'low' : 50}

reverse_line_dict_sorted = dict(sorted(reverse_line_dict.items(), key=lambda item: item[1], reverse=True))

for description, bet_per in reverse_line_dict_sorted.items():
    df_features.loc[(df_features['away_current_best_line'] < df_features['away_open_line']) & (
        df_features['away_bet_percentage'] < bet_per), 'away_reverse_line_movement'] = description 
    
    predictors_reverse_line.append('_'.join(('away_reverse_line', description)))
    predictors_key_numbers_reverse_line.append('_'.join(('away_reverse_line', description)))
    
    df_features.loc[(df_features['home_current_best_line'] < df_features['home_open_line']) & (
        df_features['home_bet_percentage'] < bet_per), 'home_reverse_line_movement'] = description 
                                   
    predictors_reverse_line.append('_'.join(('home_reverse_line', description)))
    predictors_key_numbers_reverse_line.append('_'.join(('home_reverse_line', description)))

# create dummy cols
away_reverse_dummies = pd.get_dummies(df_features['away_reverse_line_movement'], prefix='away_reverse_line')
home_reverse_dummies = pd.get_dummies(df_features['home_reverse_line_movement'], prefix='home_reverse_line')

# Let's merge our features back 
df_features = pd.merge(df_features, away_reverse_dummies, left_index=True, right_index=True)
df_features = pd.merge(df_features, home_reverse_dummies, left_index=True, right_index=True)
```

Let's run logistic regression with our reverse line movement features (but no key number features) and calculate performance.


```python
# run logistic regression with reverse live movement features
_pred, _pred_probs, df_reverse_line_preds = full_logistic_regression(
    df=df_features, 
    predictors=predictors_reverse_line, 
#     feature_scale='min_max'
)
```


```python
# calculate pick accuracy of logistic regression with key number features
df_reverse_line_season_week, reverse_line_weekly, reverse_line_correct_rate = make_predictions(
    df=df_reverse_line_preds,
    score_cols=['lr_pred_probs'], 
    ascending=[False], 
    accuracy_col='lr_preds_accurate', 
    description='Reverse Line Movement Features Logisitic Regression'
)
```


```python
reverse_line_correct_rate
```




    'Reverse Line Movement Features Logisitic Regression got 59.72% of picks correct.'



Performance got a bit worse when using our reverse line movement features but not key number features. Let's try adding back the key number features and running logistic regression again (both reverse line movement and key number features). (Note that I got a max iteration warning so I increased `max_iter` to 10000 below.)


```python
# run logistic regression with key number and reverse live movement features
_pred, _pred_probs, df_key_numbers_reverse_line_preds = full_logistic_regression(
    df=df_features, 
    max_iter = 10000,
    predictors=predictors_key_numbers_reverse_line, 
#     feature_scale='min_max'
)
```


```python
# calculate pick accuracy of logistic regression with key number features
df_key_number_reverse_line_season_week, key_number_reverse_line_weekly, key_number_reverse_line_correct_rate = make_predictions(
    df=df_key_numbers_reverse_line_preds,
    score_cols=['lr_pred_probs'], 
    ascending=[False], 
    accuracy_col='lr_preds_accurate', 
    description='Key Numbers + Reverse Line Movement Features Logisitic Regression'
)
```


```python
key_number_reverse_line_correct_rate
```




    'Key Numbers + Reverse Line Movement Features Logisitic Regression got 62.5% of picks correct.'



Performance is slightly worse when combining key numbers and reverse line movement features vs using just key number features. Let's scrap our reverse line movement features.

Let's move on to hyperparameter tuning.

## Hyperparameter Tuning
Hyperparameter tuning allows adjust the settings of the model to try to improve performance (among other objectives like reducing overfitting and underfitting).

The two logisitic regression hyperparamters I would like to focus on are:
- C: inverse regularization strength, help pervent overfitting by adjusting the loss function 
- Solvers: choose the algorithm:
  - `liblinear` is well suited for smalled datasets like ours 
  - `lbfgs` (which is the default) is well suited to a many different classes of problems.

Let's run logistic regression using our key numbers features and test combinations of these hyperparameters below.


```python
Cs = [1, 10, 100, 1000]
solvers = ['liblinear', 'lbfgs']
```


```python
for solver in solvers:
    for C in Cs:
        _, __, df_features_preds = full_logistic_regression(
            df=df_features,
            feature_scale=False, 
            predictors=predictors_key_numbers, 
            C=C,
            max_iter=10000,
            solver=solver)
        _, __, lr_correct_rate = make_predictions(
            df=df_features_preds,
            score_cols=['lr_pred_probs'], 
            ascending=[False], 
            accuracy_col='lr_preds_accurate', 
            description=' '.join(('Logisitic Regression',str(C),solver))
        )
        print(lr_correct_rate)
```

    Logisitic Regression 1 liblinear got 63.89% of picks correct.
    Logisitic Regression 10 liblinear got 66.67% of picks correct.
    Logisitic Regression 100 liblinear got 66.67% of picks correct.
    Logisitic Regression 1000 liblinear got 66.67% of picks correct.
    Logisitic Regression 1 lbfgs got 63.89% of picks correct.
    Logisitic Regression 10 lbfgs got 66.67% of picks correct.
    Logisitic Regression 100 lbfgs got 66.67% of picks correct.
    Logisitic Regression 1000 lbfgs got 65.28% of picks correct.


A few different combinations of hyperparameters were able to deliver performance 66.67% (2/3 correct picks), matching our complex heuristic. I'm actually pretty pleased with this because I spent a lot of time fine tuning my complex heuristic. I expect that our machine learning performance will improve as we generate more data to help train out model.

I'm going to pause here for now because I want to deliver this in time for Saturday September 12, 2026, the start of the Pick Four season. 

Using feature generation and hyperparameter optimization (and testing feature scaling) we were able to use a dataset of less than 2,000 records train a model that got 2/3 picks correct. 

Next steps for me will be to retrain the model using our best performing combination of features and hyperparameters and then deploy the model to help me make my picks this season.

This is quite a bit more I plan to do with this, fine tuning hyperparameters and generating and testing new features. Additionally, the more data we collect, the better I expect the algorithm to perform. Next year, it will be interesting to see how having another season of data affects the accuracy of our modeling. 

If you made it this far, thank you so much for reading! This project was super specific to my own Pick Four league but hopefully it gave you some ideas you can apply to your own projects. Cheers! 
