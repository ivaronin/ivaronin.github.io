---
layout: post
title: How do circadian rhythms affect prime time NFL game outcomes in 2024?
tags: 
 - sports
 - circadian rhythm
 - nfl
 - gambling
published: true
---
# How do circadian rhythms affect prime time NFL game outcomes in 2024?
About ten years ago I read [this Deadspin article](https://deadspin.com/the-circadian-advantage-how-sleep-patterns-benefit-cer-5934440/) which discussed the effects of circadian rhythms - the cycle during which a person naturally feels more awake and more sleepy during at different times during a 24 hour period - on NFL game scores and betting outcomes. Researchers studied games played on Monday Night Football between a team based on the west coast and a team based on the east coast, i.e. based in time zones three hours apart. Over a 25 year period the west coast team won 63% of the time, winning by an average of two touchdowns, and covered the [point spread](https://www.forbes.com/betting/football/nfl/nfl-point-spread/) - the forecasted difference between the final scores of the favored team and the underdog team, used in betting markets to try to eliminate the advantage of betting on the favored team - 70% of the time. Researchers have posited that the reason west coast teams have had such a huge advantage over east coast teams in prime time (night) games - which always start around 5:20pm Pacific Time, regardless of where the game is played - is because the players based on the west coast feel more awake than the players based on the east coast during these games. During these night games, this three hour time zone difference is significant because players based on the east coast are much closer to their body's natural time for sleep.

Since I love to compete against my friends picking the results of NFL games (both straight up and against the point spread), I wanted to do some follow up research on how circadian rhythms affect NFL game outcomes today. Is the edge still so great for the west coast team in prime time games or have coaches and team doctors found some way to adjust? Has Las Vegas adjusted the point spreads to carve away some or all of the advantage enjoyed by betting on west coast teams in such games? The original analysis focused on Monday Night Football but the NFL now regularly has prime time games on Monday, Thursday, and Sunday so it makes sense to include all three in our analysis.

I am also curious to investigate what advantage exists for the western-most team in prime time games between teams that are based in time zones two hours apart (e.g. two Mountain Time vs Eastern Time) and time zones that are one only hour apart. 

In short, the circadian rhythm phenomenon is due for an update. To investigate, I will scrape data from [pro-football-reference.com](https://www.pro-football-reference.com/), which has result and betting line data for every NFL game going back decades. I plan to scrape the most recent 30 years worth of data so we can look at trends and compare the status quo to the past.

I will work in python which has a number of helpful libraries for a project like this, notably `requests` and `BeautifulSoup` to help with scraping data and `pandas` to help with analysis, among others.

Let's get started by importing some of the libraries we know we will need for this project.


```python
# import needed libraries
from bs4 import BeautifulSoup
import pandas as pd
import requests
```

## Scrape data
Pro Football Reference (PFR) stores betting line data and game outcome data for each team on individual team pages for each season. This means in order to scrape all the data we need for each team, for each year, we will have to visit that team's [PFR Vegas Lines page](https://www.pro-football-reference.com/teams/sfo/2023_lines.htm) - which has betting lines data - and that team's [PFR Schedule and Game Results page](https://www.pro-football-reference.com/teams/sfo/2023.htm) - which has score data, kickoff times, and other stats like yards gained and yards allowed, which could be interesting to dig into as well. Before we can do that, we need to get the URL for each team page. Once we know the PFR URL of every team, we can loop through and download the data for each team for each year that we care about.

To get started scraping data, we will first download the standings tables from [https://www.pro-football-reference.com/years/2023/](https://www.pro-football-reference.com/years/2023/) which has the abbreviated names that are used to specify that PFR team page in the URL. An example of an abbreviated team name is 'buf' which is used to access the Buffalo Bills team page: https://www.pro-football-reference.com/teams/buf/2023.htm. Each team has its own abbreviated team name which we will plug into the the URL to loop through and visit each team's Vegas Lines and Schedule and Results page.

### Get team URLS


```python
# url that we care about
standings_url = 'https://www.pro-football-reference.com/years/2023/'
```


```python
# download text data from url
user_agent = {'User-Agent': 'Mozilla/5.0'}
data = requests.get(standings_url, headers=user_agent)
```


```python
# parse html
soup = BeautifulSoup(data.text)
```


```python
# select afc and nfc tables using css class selector
standing_tables = soup.select('table.stats_table')

links = []

for table in standing_tables:
    # find all of the 'a' tags to get links to team pages
    teams = table.find_all('a')
    # append each team to links list
    for team in teams:
        links.append(team)
```


```python
# get href property of each link (each 'a' tag)
links = [l.get('href') for l in links]
```


```python
# get team urls
teams = []

for link in links:
    team = link.split('/')[2]
    teams.append(team)
    
# show team name abbreviations    
teams
```




    ['buf',
     'mia',
     'nyj',
     'nwe',
     'rav',
     'cle',
     'pit',
     'cin',
     'htx',
     'jax',
     'clt',
     'oti',
     'kan',
     'rai',
     'den',
     'sdg',
     'dal',
     'phi',
     'nyg',
     'was',
     'det',
     'gnb',
     'min',
     'chi',
     'tam',
     'nor',
     'atl',
     'car',
     'sfo',
     'ram',
     'sea',
     'crd']



### Get betting lines table for one team
Now that we have the abbreviated PFR name for each team in the NFL, let's start by downloading the betting outcome data for one team for one year as a proof of concept. If we are successful, we can then loop through and download data for every team for multiple years.

We can download the table we need from each teams's [PFR Vegas Lines page](https://www.pro-football-reference.com/teams/sfo/2023_lines.htm) by pulling in the text for the table with the name "Vegas Lines"

![vegas_lines.png](https://github.com/ivaronin/ivaronin.github.io/blob/master/images/vegas_lines.png?raw=true)


```python
# build betting lines urls for each team for 2023
lines_urls = [f'https://www.pro-football-reference.com/teams/{team}/2023_lines.htm' for team in teams]
```


```python
# get url for first team in teams list
team_url = lines_urls[0]

# get lines data for first team
lines_data = requests.get(team_url)
```


```python
# convert lines data into df
lines_table = pd.read_html(lines_data.text, match='Vegas Lines')[0]

# check data
lines_table.head()
```




<div style="overflow-x:auto;">
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
      <th>G#</th>
      <th>Opp</th>
      <th>Spread</th>
      <th>Over/Under</th>
      <th>Result</th>
      <th>vs. Line</th>
      <th>OU Result</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>@NYJ</td>
      <td>-2.5</td>
      <td>45.5</td>
      <td>L, 16-22</td>
      <td>Lost</td>
      <td>Under</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>LVR</td>
      <td>-8.0</td>
      <td>47.0</td>
      <td>W, 38-10</td>
      <td>Won</td>
      <td>Over</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>@WAS</td>
      <td>-4.5</td>
      <td>43.5</td>
      <td>W, 37-3</td>
      <td>Won</td>
      <td>Under</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>MIA</td>
      <td>-2.5</td>
      <td>53.0</td>
      <td>W, 48-20</td>
      <td>Won</td>
      <td>Over</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>JAX</td>
      <td>-5.5</td>
      <td>48.5</td>
      <td>L, 20-25</td>
      <td>Lost</td>
      <td>Under</td>
    </tr>
  </tbody>
</table>
</div>



Success! Now, that we know how to download data for one team for one year, let's write a function that we can use to download both Vegas Lines and Schedule and Game Results data for all teams for multiple years.

### Get betting lines data for all teams in NFL for 30 years

One thing to keep in mind is that PFR will block traffic from any user that visits more than 20 pages on their site in less than a minute. To get around this we can import the library `time` which we can use to set our function to delay restarting for a few seconds after each loop. 

Below we will create our function and then run it to scrape betting lines data for all teams for 30 years. We'll save the data in a dataframe called `lines_df`.


```python
import time
# set range of years to pull data for
years = list(range(2023,1993, -1))
```


```python
# create a function the downloads tables from PFR
def download_tables(url, match, teams=teams, years=years):
    '''
    Accepts: 
        url (str): string url where the table to be downloaded lives with bracketed variables for year and team
        match (str): name of table to download
        teams (list): list of team names from sports-rerfernce url
        year (list): list of years to be downloaded
    Returns: list of tables for each team in teams list for each year in years list.
        '''    
    
    # create list to save tables in
    all_tables = []
    
    for year in years:
        for team in teams:
            team_url = url.format(team=team, year=year)

            # download betting data from team url
            data = requests.get(team_url)

            # get lines data
            try:
                table_df = pd.read_html(data.text, match=match)[0]

                # add metadata
                table_df['team'] = team
                table_df['year'] = year

                # append lines data to list
                all_tables.append(table_df)
            
            # account for errors
            except ValueError:
                pass

            # sleep to avoid being blocked 
            finally:
                time.sleep(4)
                
    return all_tables
```


```python
# set base url and name of table we want to pull
url = 'https://www.pro-football-reference.com/teams/{team}/{year}_lines.htm'
match = 'Vegas Lines'

# pull lines tables for every team in the nfl for the years 1994-2003 
all_lines = download_tables(url, match)
```


```python
# concat all dataframes into single dataframe
lines_df = pd.concat(all_lines)
```


```python
# make all column names lowercase
lines_df.columns = [c.lower() for c in lines_df.columns]
```

Let's be sure to drop any duplicate rows. This means we need to drop rows where teams are playing on the road (as the visiting team) because since we scrapped data for every team, these are duplicate rows. For example, the New York Jets and Buffalo Bills played each other twice in 2023, once in Buffalo and once in New York. However, both games are represented in both the dataset for the Bills and the dataset for the Jets, meaning these games are doublecounted. We will account for this by dropping rows where teams are playing on the road.


```python
# dedupe lines df
lines_df = lines_df.drop_duplicates()

# keep only rows where team is at home
lines_df = lines_df.loc[~lines_df['opp'].str.contains('@')]
```


```python
# get to know the data
lines_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 7973 entries, 0 to 7972
    Data columns (total 14 columns):
     #   Column      Non-Null Count  Dtype  
    ---  ------      --------------  -----  
     0   g#          7973 non-null   float64
     1   opp         7973 non-null   object 
     2   spread      7973 non-null   float64
     3   over/under  7973 non-null   float64
     4   result      7973 non-null   object 
     5   vs. line    7973 non-null   object 
     6   ou result   7973 non-null   object 
     7   team        7973 non-null   object 
     8   year        7973 non-null   int64  
     9   outcome     7973 non-null   object 
     10  score       7973 non-null   object 
     11  team_score  7973 non-null   int64  
     12  opp_score   7973 non-null   int64  
     13  score_diff  7973 non-null   int64  
    dtypes: float64(3), int64(4), object(7)
    memory usage: 934.3+ KB



```python
# inspect data in lines_df
lines_df.head()
```




<div style="overflow-x:auto;">
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
      <th>g#</th>
      <th>opp</th>
      <th>spread</th>
      <th>over/under</th>
      <th>result</th>
      <th>vs. line</th>
      <th>ou result</th>
      <th>team</th>
      <th>year</th>
      <th>outcome</th>
      <th>score</th>
      <th>team_score</th>
      <th>opp_score</th>
      <th>score_diff</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2.0</td>
      <td>LVR</td>
      <td>-8.0</td>
      <td>47.0</td>
      <td>W, 38-10</td>
      <td>Won</td>
      <td>Over</td>
      <td>buf</td>
      <td>2023</td>
      <td>W</td>
      <td>38-10</td>
      <td>38</td>
      <td>10</td>
      <td>28</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4.0</td>
      <td>MIA</td>
      <td>-2.5</td>
      <td>53.0</td>
      <td>W, 48-20</td>
      <td>Won</td>
      <td>Over</td>
      <td>buf</td>
      <td>2023</td>
      <td>W</td>
      <td>48-20</td>
      <td>48</td>
      <td>20</td>
      <td>28</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5.0</td>
      <td>JAX</td>
      <td>-5.5</td>
      <td>48.5</td>
      <td>L, 20-25</td>
      <td>Lost</td>
      <td>Under</td>
      <td>buf</td>
      <td>2023</td>
      <td>L</td>
      <td>20-25</td>
      <td>20</td>
      <td>25</td>
      <td>-5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6.0</td>
      <td>NYG</td>
      <td>-15.5</td>
      <td>43.5</td>
      <td>W, 14-9</td>
      <td>Lost</td>
      <td>Under</td>
      <td>buf</td>
      <td>2023</td>
      <td>W</td>
      <td>14-9</td>
      <td>14</td>
      <td>9</td>
      <td>5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>8.0</td>
      <td>TAM</td>
      <td>-10.0</td>
      <td>43.5</td>
      <td>W, 24-18</td>
      <td>Lost</td>
      <td>Under</td>
      <td>buf</td>
      <td>2023</td>
      <td>W</td>
      <td>24-18</td>
      <td>24</td>
      <td>18</td>
      <td>6</td>
    </tr>
  </tbody>
</table>
</div>



### Save betting lines dataframe to database

That worked and we now have 30 years of NFL betting line data but it took a very long time to run, mostly due to the delays we built in to avoid being blocked by PFR.

Given how long it took to pull data from football reference, we should save it somewhere to avoid having to do it again in case our program crashes or closes. I already have a database called `pick_four` (the name of my friend's nfl and college pickem league) that I use to store other NFL data so I'll create a new table in it called `nfl_spread_data` and save our `lines_df` we've been working on there.


```python
# save seasons df to sqlite database
import sqlite3
with sqlite3.connect('/Users/ianvaronin/code/betting_lines/pick_four') as conn:
    seasons_df.to_sql(name='nfl_spread_data', con=conn, if_exists='replace', index=False)
    conn.commit()
    
    # ensure df was saved correctly
    _ = pd.read_sql_query('''select * from nfl_spread_data''', conn)
    
_.head()
```




<div style="overflow-x:auto;">
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
      <th>g#</th>
      <th>opp</th>
      <th>spread</th>
      <th>over/under</th>
      <th>result</th>
      <th>vs. line</th>
      <th>ou result</th>
      <th>team</th>
      <th>year</th>
      <th>outcome</th>
      <th>score</th>
      <th>team_score</th>
      <th>opp_score</th>
      <th>score_diff</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2.0</td>
      <td>LVR</td>
      <td>-8.0</td>
      <td>47.0</td>
      <td>W, 38-10</td>
      <td>Won</td>
      <td>Over</td>
      <td>buf</td>
      <td>2023</td>
      <td>W</td>
      <td>38-10</td>
      <td>38</td>
      <td>10</td>
      <td>28</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4.0</td>
      <td>MIA</td>
      <td>-2.5</td>
      <td>53.0</td>
      <td>W, 48-20</td>
      <td>Won</td>
      <td>Over</td>
      <td>buf</td>
      <td>2023</td>
      <td>W</td>
      <td>48-20</td>
      <td>48</td>
      <td>20</td>
      <td>28</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5.0</td>
      <td>JAX</td>
      <td>-5.5</td>
      <td>48.5</td>
      <td>L, 20-25</td>
      <td>Lost</td>
      <td>Under</td>
      <td>buf</td>
      <td>2023</td>
      <td>L</td>
      <td>20-25</td>
      <td>20</td>
      <td>25</td>
      <td>-5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6.0</td>
      <td>NYG</td>
      <td>-15.5</td>
      <td>43.5</td>
      <td>W, 14-9</td>
      <td>Lost</td>
      <td>Under</td>
      <td>buf</td>
      <td>2023</td>
      <td>W</td>
      <td>14-9</td>
      <td>14</td>
      <td>9</td>
      <td>5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>8.0</td>
      <td>TAM</td>
      <td>-10.0</td>
      <td>43.5</td>
      <td>W, 24-18</td>
      <td>Lost</td>
      <td>Under</td>
      <td>buf</td>
      <td>2023</td>
      <td>W</td>
      <td>24-18</td>
      <td>24</td>
      <td>18</td>
      <td>6</td>
    </tr>
  </tbody>
</table>
</div>



### Get schedule and game results data for one team
Let's run through the same process to download game outcome data from PFR. As before, we'll start by downloading table data (in this case the NFL Schedule and Game Results table) from one team for one year as a proof of concept, then download data for all teams for multiple years.


```python
# build season urls for each team for 2023
team_urls = [f'https://www.pro-football-reference.com/teams/{team}/2023.htm' for team in teams]
```


```python
# get season data for first team
team_url = team_urls[0]
team_data = requests.get(team_url)
```


```python
# get match data
team_table = pd.read_html(team_data.text)[0]
```


```python
# get match data
team_table = pd.read_html(team_data.text, match='Schedule & Game Results')[0]

# keep only 2nd level column values
team_table.columns = team_table.columns.get_level_values(1)

# drop unneeded column
team_table = team_table.drop(team_table.columns[4], axis=1)

# rename unlabled columns
col_mapper = {
              team_table.columns[3] : 'Kickoff Time',
              team_table.columns[4] : 'Result',
              team_table.columns[7] : 'Location'
             }
team_table = team_table.rename(columns=col_mapper)
```


```python
# get to know the columns
seasons_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 7916 entries, 0 to 7915
    Data columns (total 26 columns):
     #   Column            Non-Null Count  Dtype 
    ---  ------            --------------  ----- 
     0   Week              7916 non-null   object
     1   Day               7916 non-null   object
     2   Date              7916 non-null   object
     3   Kickoff_Time      7916 non-null   object
     4   Result            7916 non-null   object
     5   OT                486 non-null    object
     6   Rec               7916 non-null   object
     7   Location          0 non-null      object
     8   Opponent          7916 non-null   object
     9   Points_Scored     7916 non-null   object
     10  Points_Allowed    7916 non-null   object
     11  1stD_Off          7916 non-null   object
     12  TotYd_Off         7916 non-null   object
     13  PassY_Off         7916 non-null   object
     14  RushY_Off         7916 non-null   object
     15  TO_Off            6063 non-null   object
     16  1stD_Def          7916 non-null   object
     17  TotYd_Def         7916 non-null   object
     18  PassY_Def         7914 non-null   object
     19  RushY_Def         7916 non-null   object
     20  TO_Def            6239 non-null   object
     21  Exp_Points_Off    7916 non-null   object
     22  Exp_Points_Def    7916 non-null   object
     23  Exp_Points_SpTms  7916 non-null   object
     24  team              7916 non-null   object
     25  year              7916 non-null   object
    dtypes: object(26)
    memory usage: 1.6+ MB



```python
# inspect data
team_table.head()
```




<div style="overflow-x:auto;">
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
      <th>Week</th>
      <th>Day</th>
      <th>Date</th>
      <th>Kickoff Time</th>
      <th>Result</th>
      <th>OT</th>
      <th>Rec</th>
      <th>Location</th>
      <th>Opp</th>
      <th>Tm</th>
      <th>Opp</th>
      <th>1stD</th>
      <th>TotYd</th>
      <th>PassY</th>
      <th>RushY</th>
      <th>TO</th>
      <th>1stD</th>
      <th>TotYd</th>
      <th>PassY</th>
      <th>RushY</th>
      <th>TO</th>
      <th>Offense</th>
      <th>Defense</th>
      <th>Sp. Tms</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Mon</td>
      <td>September 11</td>
      <td>8:15PM ET</td>
      <td>L</td>
      <td>OT</td>
      <td>0-1</td>
      <td>@</td>
      <td>New York Jets</td>
      <td>16.0</td>
      <td>22.0</td>
      <td>19.0</td>
      <td>314.0</td>
      <td>217.0</td>
      <td>97.0</td>
      <td>4.0</td>
      <td>13.0</td>
      <td>289.0</td>
      <td>117.0</td>
      <td>172.0</td>
      <td>1.0</td>
      <td>-4.85</td>
      <td>1.19</td>
      <td>-2.27</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Sun</td>
      <td>September 17</td>
      <td>1:00PM ET</td>
      <td>W</td>
      <td>NaN</td>
      <td>1-1</td>
      <td>NaN</td>
      <td>Las Vegas Raiders</td>
      <td>38.0</td>
      <td>10.0</td>
      <td>29.0</td>
      <td>450.0</td>
      <td>267.0</td>
      <td>183.0</td>
      <td>NaN</td>
      <td>13.0</td>
      <td>240.0</td>
      <td>185.0</td>
      <td>55.0</td>
      <td>3.0</td>
      <td>27.19</td>
      <td>4.89</td>
      <td>-3.85</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Sun</td>
      <td>September 24</td>
      <td>1:00PM ET</td>
      <td>W</td>
      <td>NaN</td>
      <td>2-1</td>
      <td>@</td>
      <td>Washington Commanders</td>
      <td>37.0</td>
      <td>3.0</td>
      <td>20.0</td>
      <td>386.0</td>
      <td>218.0</td>
      <td>168.0</td>
      <td>1.0</td>
      <td>15.0</td>
      <td>230.0</td>
      <td>125.0</td>
      <td>105.0</td>
      <td>5.0</td>
      <td>10.16</td>
      <td>23.07</td>
      <td>-1.39</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Sun</td>
      <td>October 1</td>
      <td>1:00PM ET</td>
      <td>W</td>
      <td>NaN</td>
      <td>3-1</td>
      <td>NaN</td>
      <td>Miami Dolphins</td>
      <td>48.0</td>
      <td>20.0</td>
      <td>24.0</td>
      <td>414.0</td>
      <td>310.0</td>
      <td>104.0</td>
      <td>NaN</td>
      <td>20.0</td>
      <td>393.0</td>
      <td>251.0</td>
      <td>142.0</td>
      <td>2.0</td>
      <td>27.56</td>
      <td>2.09</td>
      <td>-1.37</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Sun</td>
      <td>October 8</td>
      <td>9:30AM ET</td>
      <td>L</td>
      <td>NaN</td>
      <td>3-2</td>
      <td>NaN</td>
      <td>Jacksonville Jaguars</td>
      <td>20.0</td>
      <td>25.0</td>
      <td>18.0</td>
      <td>388.0</td>
      <td>359.0</td>
      <td>29.0</td>
      <td>2.0</td>
      <td>29.0</td>
      <td>474.0</td>
      <td>278.0</td>
      <td>196.0</td>
      <td>2.0</td>
      <td>6.27</td>
      <td>-13.93</td>
      <td>-2.21</td>
    </tr>
  </tbody>
</table>
</div>



### Get schedule and game results data for all teams in NFL for 30 years
Since we had success pulling game outcome data for one team for one year, let's now use our function to loop through and grab the Schedule and Game Results tables for all teams for 30 years just as we did with betting data. We'll save it in a dataframe called `seasons_df`.


```python
# pull seasons data from football reference for all teams for past three decades
url = 'https://www.pro-football-reference.com/teams/{team}/{year}.htm'
match = 'Schedule & Game Results'
all_seasons = download_tables(url, match)
```


```python
# concatate seasons data into df
seasons_df = pd.concat(all_seasons)

# drop unneeded column
seasons_df = seasons_df.drop(seasons_df.columns[4], axis=1)
```


```python
#hide
# rename unlabled columns and columns with unclear names
col_mapper_reverse = {
              seasons_df.columns[3] : 'Kickoff_Time',
              seasons_df.columns[4] : 'Result',
              seasons_df.columns[7] : 'Location',
              'Tm' : 'Points_Scored',
              'Offense' : 'Exp_Points_Off',
              'Defense' : 'Exp_Points_Def',
              'Sp. Tms' : 'Exp_Points_Sp_Tms',
             }
seasons_df = seasons_df.rename(columns=col_mapper_reverse)

# rename offense and defense columns
offdef_reverse = {'Opponent' : 'Opp',
          'Points_Allowed': 'Opp',
          '1stD_Off' : '1stD', 
          '1stD_Def' : '1stD',
          'TotYd_Off' : 'TotYd',
          'TotYd_Def' : 'TotYd',
          'PassY_Off' : 'PassY',
          'PassY_Def' : 'PassY' ,
          'RushY_Off' : 'RushY',
          'RushY_Def' : 'RushY',
          'TO_Off' : 'TO',
          'TO_Def' : 'TO'}

# rename duplicate column names per the offdef dictionary
seasons_df = seasons_df.rename(columns=offdef_reverse)
```


```python
# get to know the columns
seasons_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 7916 entries, 0 to 7915
    Data columns (total 26 columns):
     #   Column            Non-Null Count  Dtype 
    ---  ------            --------------  ----- 
     0   Week              7916 non-null   object
     1   Day               7916 non-null   object
     2   Date              7916 non-null   object
     3   Kickoff_Time      7916 non-null   object
     4   Result            7916 non-null   object
     5   OT                486 non-null    object
     6   Rec               7916 non-null   object
     7   Location          0 non-null      object
     8   Opp               7916 non-null   object
     9   Points_Scored     7916 non-null   object
     10  Opp               7916 non-null   object
     11  1stD              7916 non-null   object
     12  TotYd             7916 non-null   object
     13  PassY             7916 non-null   object
     14  RushY             7916 non-null   object
     15  TO                6063 non-null   object
     16  1stD              7916 non-null   object
     17  TotYd             7916 non-null   object
     18  PassY             7914 non-null   object
     19  RushY             7916 non-null   object
     20  TO                6239 non-null   object
     21  Exp_Points_Off    7916 non-null   object
     22  Exp_Points_Def    7916 non-null   object
     23  Exp_Points_SpTms  7916 non-null   object
     24  team              7916 non-null   object
     25  year              7916 non-null   object
    dtypes: object(26)
    memory usage: 1.6+ MB



```python
# inspect data in seasons_df
seasons_df.head()
```




<div style="overflow-x:auto;">
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
      <th>Week</th>
      <th>Day</th>
      <th>Date</th>
      <th>Kickoff_Time</th>
      <th>Result</th>
      <th>OT</th>
      <th>Rec</th>
      <th>Location</th>
      <th>Opp</th>
      <th>Points_Scored</th>
      <th>Opp</th>
      <th>1stD</th>
      <th>TotYd</th>
      <th>PassY</th>
      <th>RushY</th>
      <th>TO</th>
      <th>1stD</th>
      <th>TotYd</th>
      <th>PassY</th>
      <th>RushY</th>
      <th>TO</th>
      <th>Exp_Points_Off</th>
      <th>Exp_Points_Def</th>
      <th>Exp_Points_SpTms</th>
      <th>team</th>
      <th>year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>Sun</td>
      <td>September 17</td>
      <td>1:00PM ET</td>
      <td>W</td>
      <td>None</td>
      <td>1-1</td>
      <td>None</td>
      <td>Las Vegas Raiders</td>
      <td>38.0</td>
      <td>10.0</td>
      <td>29.0</td>
      <td>450.0</td>
      <td>267.0</td>
      <td>183.0</td>
      <td>None</td>
      <td>13.0</td>
      <td>240.0</td>
      <td>185.0</td>
      <td>55.0</td>
      <td>3.0</td>
      <td>27.19</td>
      <td>4.89</td>
      <td>-3.85</td>
      <td>buf</td>
      <td>2023</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4</td>
      <td>Sun</td>
      <td>October 1</td>
      <td>1:00PM ET</td>
      <td>W</td>
      <td>None</td>
      <td>3-1</td>
      <td>None</td>
      <td>Miami Dolphins</td>
      <td>48.0</td>
      <td>20.0</td>
      <td>24.0</td>
      <td>414.0</td>
      <td>310.0</td>
      <td>104.0</td>
      <td>None</td>
      <td>20.0</td>
      <td>393.0</td>
      <td>251.0</td>
      <td>142.0</td>
      <td>2.0</td>
      <td>27.56</td>
      <td>2.09</td>
      <td>-1.37</td>
      <td>buf</td>
      <td>2023</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5</td>
      <td>Sun</td>
      <td>October 8</td>
      <td>9:30AM ET</td>
      <td>L</td>
      <td>None</td>
      <td>3-2</td>
      <td>None</td>
      <td>Jacksonville Jaguars</td>
      <td>20.0</td>
      <td>25.0</td>
      <td>18.0</td>
      <td>388.0</td>
      <td>359.0</td>
      <td>29.0</td>
      <td>2.0</td>
      <td>29.0</td>
      <td>474.0</td>
      <td>278.0</td>
      <td>196.0</td>
      <td>2.0</td>
      <td>6.27</td>
      <td>-13.93</td>
      <td>-2.21</td>
      <td>buf</td>
      <td>2023</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6</td>
      <td>Sun</td>
      <td>October 15</td>
      <td>8:20PM ET</td>
      <td>W</td>
      <td>None</td>
      <td>4-2</td>
      <td>None</td>
      <td>New York Giants</td>
      <td>14.0</td>
      <td>9.0</td>
      <td>22.0</td>
      <td>297.0</td>
      <td>169.0</td>
      <td>128.0</td>
      <td>2.0</td>
      <td>20.0</td>
      <td>317.0</td>
      <td>185.0</td>
      <td>132.0</td>
      <td>None</td>
      <td>2.9</td>
      <td>-3.02</td>
      <td>-6.7</td>
      <td>buf</td>
      <td>2023</td>
    </tr>
    <tr>
      <th>4</th>
      <td>8</td>
      <td>Thu</td>
      <td>October 26</td>
      <td>8:15PM ET</td>
      <td>W</td>
      <td>None</td>
      <td>5-3</td>
      <td>None</td>
      <td>Tampa Bay Buccaneers</td>
      <td>24.0</td>
      <td>18.0</td>
      <td>25.0</td>
      <td>427.0</td>
      <td>312.0</td>
      <td>115.0</td>
      <td>1.0</td>
      <td>17.0</td>
      <td>302.0</td>
      <td>224.0</td>
      <td>78.0</td>
      <td>None</td>
      <td>9.78</td>
      <td>-3.38</td>
      <td>-1.24</td>
      <td>buf</td>
      <td>2023</td>
    </tr>
  </tbody>
</table>
</div>



Success! 

### Rename columns to be more descriptive

Some of the columns in our `seasons_df` are missing labels and others have confusing or duplicate labels. We can infer the meaning of these columns by looking at the source data on pro-football-reference.com. Let's rename these column names to be more descriptive.

![pfr_sfo_szn.png](https://github.com/ivaronin/ivaronin.github.io/blob/master/images/pfr_sfo_szn.png?raw=true)


```python
# rename unlabled columns and columns with unclear names
col_mapper = {
              seasons_df.columns[3] : 'Kickoff_Time',
              seasons_df.columns[4] : 'Result',
              seasons_df.columns[7] : 'Location',
              'Tm' : 'Points_Scored',
              'Offense' : 'Exp_Points_Off',
              'Defense' : 'Exp_Points_Def',
              'Sp. Tms' : 'Exp_Points_Sp_Tms',
             }
# rename confusing column names
seasons_df = seasons_df.rename(columns=col_mapper)
```


```python
# rename offense and defense columns
offdef = {'Opp' : ['Opponent', 'Points_Allowed'],
          '1stD' : ['1stD_Off', '1stD_Def'],
          'TotYd' : ['TotYd_Off', 'TotYd_Def'],
          'PassY' : ['PassY_Off', 'PassY_Def'],
          'RushY' : ['RushY_Off', 'RushY_Def'],
          'TO' : ['TO_Off', 'TO_Def']}

seasons_df = seasons_df.rename(columns=(lambda x: offdef[x].pop(0) if x in offdef.keys() else x))
```


```python
# inspect columns
seasons_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 7916 entries, 0 to 7915
    Data columns (total 26 columns):
     #   Column            Non-Null Count  Dtype 
    ---  ------            --------------  ----- 
     0   Week              7916 non-null   object
     1   Day               7916 non-null   object
     2   Date              7916 non-null   object
     3   Kickoff_Time      7916 non-null   object
     4   Result            7916 non-null   object
     5   OT                486 non-null    object
     6   Rec               7916 non-null   object
     7   Location          0 non-null      object
     8   Opponent          7916 non-null   object
     9   Points_Scored     7916 non-null   object
     10  Points_Allowed    7916 non-null   object
     11  1stD_Off          7916 non-null   object
     12  TotYd_Off         7916 non-null   object
     13  PassY_Off         7916 non-null   object
     14  RushY_Off         7916 non-null   object
     15  TO_Off            6063 non-null   object
     16  1stD_Def          7916 non-null   object
     17  TotYd_Def         7916 non-null   object
     18  PassY_Def         7914 non-null   object
     19  RushY_Def         7916 non-null   object
     20  TO_Def            6239 non-null   object
     21  Exp_Points_Off    7916 non-null   object
     22  Exp_Points_Def    7916 non-null   object
     23  Exp_Points_SpTms  7916 non-null   object
     24  team              7916 non-null   object
     25  year              7916 non-null   object
    dtypes: object(26)
    memory usage: 1.6+ MB



```python
# inspect same of data
seasons_df.head()
```




<div style="overflow-x:auto;">
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
      <th>Week</th>
      <th>Day</th>
      <th>Date</th>
      <th>Kickoff_Time</th>
      <th>Result</th>
      <th>OT</th>
      <th>Rec</th>
      <th>Location</th>
      <th>Opponent</th>
      <th>Points_Scored</th>
      <th>Points_Allowed</th>
      <th>1stD_Off</th>
      <th>TotYd_Off</th>
      <th>PassY_Off</th>
      <th>RushY_Off</th>
      <th>TO_Off</th>
      <th>1stD_Def</th>
      <th>TotYd_Def</th>
      <th>PassY_Def</th>
      <th>RushY_Def</th>
      <th>TO_Def</th>
      <th>Exp_Points_Off</th>
      <th>Exp_Points_Def</th>
      <th>Exp_Points_SpTms</th>
      <th>team</th>
      <th>year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>Sun</td>
      <td>September 17</td>
      <td>1:00PM ET</td>
      <td>W</td>
      <td>None</td>
      <td>1-1</td>
      <td>None</td>
      <td>Las Vegas Raiders</td>
      <td>38.0</td>
      <td>10.0</td>
      <td>29.0</td>
      <td>450.0</td>
      <td>267.0</td>
      <td>183.0</td>
      <td>None</td>
      <td>13.0</td>
      <td>240.0</td>
      <td>185.0</td>
      <td>55.0</td>
      <td>3.0</td>
      <td>27.19</td>
      <td>4.89</td>
      <td>-3.85</td>
      <td>buf</td>
      <td>2023</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4</td>
      <td>Sun</td>
      <td>October 1</td>
      <td>1:00PM ET</td>
      <td>W</td>
      <td>None</td>
      <td>3-1</td>
      <td>None</td>
      <td>Miami Dolphins</td>
      <td>48.0</td>
      <td>20.0</td>
      <td>24.0</td>
      <td>414.0</td>
      <td>310.0</td>
      <td>104.0</td>
      <td>None</td>
      <td>20.0</td>
      <td>393.0</td>
      <td>251.0</td>
      <td>142.0</td>
      <td>2.0</td>
      <td>27.56</td>
      <td>2.09</td>
      <td>-1.37</td>
      <td>buf</td>
      <td>2023</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5</td>
      <td>Sun</td>
      <td>October 8</td>
      <td>9:30AM ET</td>
      <td>L</td>
      <td>None</td>
      <td>3-2</td>
      <td>None</td>
      <td>Jacksonville Jaguars</td>
      <td>20.0</td>
      <td>25.0</td>
      <td>18.0</td>
      <td>388.0</td>
      <td>359.0</td>
      <td>29.0</td>
      <td>2.0</td>
      <td>29.0</td>
      <td>474.0</td>
      <td>278.0</td>
      <td>196.0</td>
      <td>2.0</td>
      <td>6.27</td>
      <td>-13.93</td>
      <td>-2.21</td>
      <td>buf</td>
      <td>2023</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6</td>
      <td>Sun</td>
      <td>October 15</td>
      <td>8:20PM ET</td>
      <td>W</td>
      <td>None</td>
      <td>4-2</td>
      <td>None</td>
      <td>New York Giants</td>
      <td>14.0</td>
      <td>9.0</td>
      <td>22.0</td>
      <td>297.0</td>
      <td>169.0</td>
      <td>128.0</td>
      <td>2.0</td>
      <td>20.0</td>
      <td>317.0</td>
      <td>185.0</td>
      <td>132.0</td>
      <td>None</td>
      <td>2.9</td>
      <td>-3.02</td>
      <td>-6.7</td>
      <td>buf</td>
      <td>2023</td>
    </tr>
    <tr>
      <th>4</th>
      <td>8</td>
      <td>Thu</td>
      <td>October 26</td>
      <td>8:15PM ET</td>
      <td>W</td>
      <td>None</td>
      <td>5-3</td>
      <td>None</td>
      <td>Tampa Bay Buccaneers</td>
      <td>24.0</td>
      <td>18.0</td>
      <td>25.0</td>
      <td>427.0</td>
      <td>312.0</td>
      <td>115.0</td>
      <td>1.0</td>
      <td>17.0</td>
      <td>302.0</td>
      <td>224.0</td>
      <td>78.0</td>
      <td>None</td>
      <td>9.78</td>
      <td>-3.38</td>
      <td>-1.24</td>
      <td>buf</td>
      <td>2023</td>
    </tr>
  </tbody>
</table>
</div>



### Save seasons dataframe to database

Just as we saved our `lines_df` to our database we should do the same with our `seasons_df`.


```python
# save seasons df to sqlite database
with sqlite3.connect('/Users/ianvaronin/code/betting_lines/pick_four') as conn:
    seasons_df.to_sql(name='nfl_seasons_data', con=conn, if_exists='replace', index=False)
    conn.commit()
    
    # ensure df was saved correctly
    _ = pd.read_sql_query('''select * from nfl_seasons_data''', conn)
    
_.head()
```




<div style="overflow-x:auto;">
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
      <th>Week</th>
      <th>Day</th>
      <th>Date</th>
      <th>Kickoff_Time</th>
      <th>Result</th>
      <th>OT</th>
      <th>Rec</th>
      <th>Location</th>
      <th>Opponent</th>
      <th>Points_Scored</th>
      <th>Points_Allowed</th>
      <th>1stD_Off</th>
      <th>TotYd_Off</th>
      <th>PassY_Off</th>
      <th>RushY_Off</th>
      <th>TO_Off</th>
      <th>1stD_Def</th>
      <th>TotYd_Def</th>
      <th>PassY_Def</th>
      <th>RushY_Def</th>
      <th>TO_Def</th>
      <th>Exp_Points_Off</th>
      <th>Exp_Points_Def</th>
      <th>Exp_Points_SpTms</th>
      <th>team</th>
      <th>year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>Sun</td>
      <td>September 17</td>
      <td>1:00PM ET</td>
      <td>W</td>
      <td>None</td>
      <td>1-1</td>
      <td>None</td>
      <td>Las Vegas Raiders</td>
      <td>38.0</td>
      <td>10.0</td>
      <td>29.0</td>
      <td>450.0</td>
      <td>267.0</td>
      <td>183.0</td>
      <td>None</td>
      <td>13.0</td>
      <td>240.0</td>
      <td>185.0</td>
      <td>55.0</td>
      <td>3.0</td>
      <td>27.19</td>
      <td>4.89</td>
      <td>-3.85</td>
      <td>buf</td>
      <td>2023</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4</td>
      <td>Sun</td>
      <td>October 1</td>
      <td>1:00PM ET</td>
      <td>W</td>
      <td>None</td>
      <td>3-1</td>
      <td>None</td>
      <td>Miami Dolphins</td>
      <td>48.0</td>
      <td>20.0</td>
      <td>24.0</td>
      <td>414.0</td>
      <td>310.0</td>
      <td>104.0</td>
      <td>None</td>
      <td>20.0</td>
      <td>393.0</td>
      <td>251.0</td>
      <td>142.0</td>
      <td>2.0</td>
      <td>27.56</td>
      <td>2.09</td>
      <td>-1.37</td>
      <td>buf</td>
      <td>2023</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5</td>
      <td>Sun</td>
      <td>October 8</td>
      <td>9:30AM ET</td>
      <td>L</td>
      <td>None</td>
      <td>3-2</td>
      <td>None</td>
      <td>Jacksonville Jaguars</td>
      <td>20.0</td>
      <td>25.0</td>
      <td>18.0</td>
      <td>388.0</td>
      <td>359.0</td>
      <td>29.0</td>
      <td>2.0</td>
      <td>29.0</td>
      <td>474.0</td>
      <td>278.0</td>
      <td>196.0</td>
      <td>2.0</td>
      <td>6.27</td>
      <td>-13.93</td>
      <td>-2.21</td>
      <td>buf</td>
      <td>2023</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6</td>
      <td>Sun</td>
      <td>October 15</td>
      <td>8:20PM ET</td>
      <td>W</td>
      <td>None</td>
      <td>4-2</td>
      <td>None</td>
      <td>New York Giants</td>
      <td>14.0</td>
      <td>9.0</td>
      <td>22.0</td>
      <td>297.0</td>
      <td>169.0</td>
      <td>128.0</td>
      <td>2.0</td>
      <td>20.0</td>
      <td>317.0</td>
      <td>185.0</td>
      <td>132.0</td>
      <td>None</td>
      <td>2.9</td>
      <td>-3.02</td>
      <td>-6.7</td>
      <td>buf</td>
      <td>2023</td>
    </tr>
    <tr>
      <th>4</th>
      <td>8</td>
      <td>Thu</td>
      <td>October 26</td>
      <td>8:15PM ET</td>
      <td>W</td>
      <td>None</td>
      <td>5-3</td>
      <td>None</td>
      <td>Tampa Bay Buccaneers</td>
      <td>24.0</td>
      <td>18.0</td>
      <td>25.0</td>
      <td>427.0</td>
      <td>312.0</td>
      <td>115.0</td>
      <td>1.0</td>
      <td>17.0</td>
      <td>302.0</td>
      <td>224.0</td>
      <td>78.0</td>
      <td>None</td>
      <td>9.78</td>
      <td>-3.38</td>
      <td>-1.24</td>
      <td>buf</td>
      <td>2023</td>
    </tr>
  </tbody>
</table>
</div>



## Prepare data for analysis
Now that we have our data, we will dig in and figure out what cleaning and transformation needs to be done before we can analyze it.
### Drop unneeded rows
There are a few types of rows that can be dropped:
- Some of the rows that are empty except for the `Date` column which says "Playoffs". This is actually a sub heading and these rows can be dropped
- Some of the rows have are empty except for the `Opp` column which says "Bye Week". This is actually a sub heading and these rows can be dropped

![nfl_sfo_2023.png](https://github.com/ivaronin/ivaronin.github.io/blob/master/images/nfl_sfo_2023.png?raw=true))


```python
# drop Playoff and Bye Week rows
seasons_df = seasons_df.drop(seasons_df.loc[seasons_df['Date'] == 'Playoffs'].index)
seasons_df = seasons_df.drop(seasons_df.loc[seasons_df['Opponent'] == 'Bye Week'].index)
```

As we did with `lines_df` df we need to drop rows where the team is playing on the road.


```python
# drop rows where team is visiting (duplicate rows)
seasons_df = seasons_df.drop(seasons_df.loc[~seasons_df['Location'].isna()].index)
```

After we drop the rows where the team is on the road the `Location` column is empty so we can drop it.


```python
# drop empty Location column
seasons_df = seasons_df.drop(columns=['Location'])
```

### Set Team Time Zones
To do our circadian rhythm analysis we need to assign each home team and visiting team a time zone. Our analysis will focus on teams with mismatched time zones and with kickoffs after 8pm ET (prime time/night games).

A couple of circumstances to keep in mind here:
- Teams that moved to a new timezone during the analysis timeframe (since 1994, the first year of our analysis): 
  - <b>The Rams</b> moved from Los Angeles to St Louis starting with the 1995 season and then moved back to LA following the 2015 season. The Rams will be in Pacific Time for the first season of this anlysis, then move to Central Time, then finally back to Pacific. 
- Teams that are based in states that don't adhere to the US standard daylight saving time schedule
  - <b>The Arizona Cardinals</b> align with Mountain Time during DST (until either October or November, see below), then align with Pacific Time after DST ends each year
  
Note that other teams have moved cities during this timeframe (The Chargers, The Raiders (3X), The Oilers/Titans) but they did not change timezones so we do not have to make any special adjustments for them.

Another thing to keep in mind is that the US changed the day of the year clocks change due to daylight saving time starting in 2007. From 1987 to 2006, DST ended on the last Sunday in October and US clocks were moved back one hour on that day (Since the NFL season only overlaps with the fall clock change, we can ignore the spring clock change). Starting in 2007, DST has ended on the first Sunday in November. 

We'll need to account for these changes in our analysis. We'll do this by assigning time zones based on the current location of each team and based on the current daylight saving time rules, then make the necessary adjustments going backward. 

For home teams:
- I prepared a CSV with team/time zone associations that are accurate at the start of the 2024 NFL season for home teams under the `team` column (using the shortened team initals which we used to label teams when we pulled in the data using our loop).

For visiting teams (opponent): 
- I also prepared a CSV with team/time zone associations for all the full team names (city + nickname) that appear in our dataset under the `Opponents` column. Luckily for us, Football Reference includes the opponent's location. Since The Rams label includes their current city in this dataset we won't have to worry about adjusting their timezone based on the year when The Rams are opponents.


```python
# show example team names
seasons_df['team'].sample(n=5)
```




    3106    kan
    140     buf
    4626    nyg
    2917    kan
    5536    min
    Name: team, dtype: object




```python
# show example opponent names
seasons_df['Opponent'].head()
```




    0       Las Vegas Raiders
    1          Miami Dolphins
    2    Jacksonville Jaguars
    3         New York Giants
    4    Tampa Bay Buccaneers
    Name: Opponent, dtype: object



#### Set Home Team Time Zone
Let's start by joining the home team time zone csv data to our dataframe.


```python
# load home team time zone data
time_zones = pd.read_csv('team_initial_time_zone.csv')

# inspect home team time zone data
time_zones.head()
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
      <th>team_initial</th>
      <th>time_zone</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>crd</td>
      <td>MT</td>
    </tr>
    <tr>
      <th>1</th>
      <td>atl</td>
      <td>ET</td>
    </tr>
    <tr>
      <th>2</th>
      <td>rav</td>
      <td>ET</td>
    </tr>
    <tr>
      <th>3</th>
      <td>buf</td>
      <td>ET</td>
    </tr>
    <tr>
      <th>4</th>
      <td>car</td>
      <td>ET</td>
    </tr>
  </tbody>
</table>
</div>




```python
# rename time_zone column to be clearer
time_zones = time_zones.rename(columns={'time_zone' : 'time_zone_home'})
```


```python
# join home team time zone data to season df
seasons_df = pd.merge(left=seasons_df, right=time_zones, left_on='team', right_on='team_initial')
seasons_df = seasons_df.drop(columns=['team_initial'])
```

#### Set Opponent Time Zone 
Now let's join the opponent time zone csv data to our dataframe.


```python
# load opponent time zone data
opponent_time_zones = pd.read_csv('full_team_name_time_zone.csv')

# inspect opponent time zone data
opponent_time_zones.head()
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
      <th>full_team_name</th>
      <th>time_zone</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Arizona Cardinals</td>
      <td>MT</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Atlanta Falcons</td>
      <td>ET</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Baltimore Ravens</td>
      <td>ET</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Buffalo Bills</td>
      <td>ET</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Carolina Panthers</td>
      <td>ET</td>
    </tr>
  </tbody>
</table>
</div>




```python
# rename time_zone column to be clearer
opponent_time_zones = opponent_time_zones.rename(columns={'time_zone' : 'time_zone_opponent'})
```

Let's take a look at The Rams time zone data in the `opponent_time_zones` dataset.


```python
opponent_time_zones.loc[opponent_time_zones['full_team_name'].str.contains('Rams')]
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
      <th>full_team_name</th>
      <th>time_zone_opponent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>28</th>
      <td>St. Louis Rams</td>
      <td>CT</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Los Angeles Rams</td>
      <td>PT</td>
    </tr>
  </tbody>
</table>
</div>



As expected, we can see that the opponent time zone changes based on the location of The Rams.


```python
# join home team time zone data to season df
seasons_df = pd.merge(left=seasons_df, right=opponent_time_zones, left_on='Opponent', right_on='full_team_name')
seasons_df = seasons_df.drop(columns=['full_team_name'])
```


```python
seasons_df.head()
```




<div style="overflow-x:auto;">
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
      <th>Week</th>
      <th>Day</th>
      <th>Date</th>
      <th>Kickoff_Time</th>
      <th>Result</th>
      <th>OT</th>
      <th>Rec</th>
      <th>Opponent</th>
      <th>Points_Scored</th>
      <th>Points_Allowed</th>
      <th>1stD_Off</th>
      <th>TotYd_Off</th>
      <th>PassY_Off</th>
      <th>RushY_Off</th>
      <th>TO_Off</th>
      <th>1stD_Def</th>
      <th>TotYd_Def</th>
      <th>PassY_Def</th>
      <th>RushY_Def</th>
      <th>TO_Def</th>
      <th>Exp_Points_Off</th>
      <th>Exp_Points_Def</th>
      <th>Exp_Points_SpTms</th>
      <th>team</th>
      <th>year</th>
      <th>time_zone_home</th>
      <th>time_zone_opponent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>Sun</td>
      <td>September 17</td>
      <td>1:00PM ET</td>
      <td>W</td>
      <td>None</td>
      <td>1-1</td>
      <td>Las Vegas Raiders</td>
      <td>38.0</td>
      <td>10.0</td>
      <td>29.0</td>
      <td>450.0</td>
      <td>267.0</td>
      <td>183.0</td>
      <td>None</td>
      <td>13.0</td>
      <td>240.0</td>
      <td>185.0</td>
      <td>55.0</td>
      <td>3.0</td>
      <td>27.19</td>
      <td>4.89</td>
      <td>-3.85</td>
      <td>buf</td>
      <td>2023</td>
      <td>ET</td>
      <td>PT</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11</td>
      <td>Sun</td>
      <td>November 19</td>
      <td>1:00PM ET</td>
      <td>W</td>
      <td>None</td>
      <td>7-3</td>
      <td>Las Vegas Raiders</td>
      <td>20.0</td>
      <td>13.0</td>
      <td>21.0</td>
      <td>422.0</td>
      <td>323.0</td>
      <td>99.0</td>
      <td>3.0</td>
      <td>12.0</td>
      <td>296.0</td>
      <td>260.0</td>
      <td>36.0</td>
      <td>3.0</td>
      <td>1.48</td>
      <td>15.77</td>
      <td>-8.12</td>
      <td>mia</td>
      <td>2023</td>
      <td>ET</td>
      <td>PT</td>
    </tr>
    <tr>
      <th>2</th>
      <td>13</td>
      <td>Sun</td>
      <td>December 6</td>
      <td>1:00PM ET</td>
      <td>L</td>
      <td>None</td>
      <td>0-12</td>
      <td>Las Vegas Raiders</td>
      <td>28.0</td>
      <td>31.0</td>
      <td>24.0</td>
      <td>376.0</td>
      <td>170.0</td>
      <td>206.0</td>
      <td>3.0</td>
      <td>24.0</td>
      <td>440.0</td>
      <td>368.0</td>
      <td>72.0</td>
      <td>2.0</td>
      <td>7.59</td>
      <td>-9.86</td>
      <td>0.37</td>
      <td>nyj</td>
      <td>2020</td>
      <td>ET</td>
      <td>PT</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>Sun</td>
      <td>September 27</td>
      <td>1:00PM ET</td>
      <td>W</td>
      <td>None</td>
      <td>2-1</td>
      <td>Las Vegas Raiders</td>
      <td>36.0</td>
      <td>20.0</td>
      <td>25.0</td>
      <td>406.0</td>
      <td>156.0</td>
      <td>250.0</td>
      <td>1.0</td>
      <td>22.0</td>
      <td>375.0</td>
      <td>249.0</td>
      <td>126.0</td>
      <td>3.0</td>
      <td>11.03</td>
      <td>-0.71</td>
      <td>5.81</td>
      <td>nwe</td>
      <td>2020</td>
      <td>ET</td>
      <td>PT</td>
    </tr>
    <tr>
      <th>4</th>
      <td>15</td>
      <td>Mon</td>
      <td>December 20</td>
      <td>5:00PM ET</td>
      <td>L</td>
      <td>None</td>
      <td>7-7</td>
      <td>Las Vegas Raiders</td>
      <td>14.0</td>
      <td>16.0</td>
      <td>13.0</td>
      <td>236.0</td>
      <td>147.0</td>
      <td>89.0</td>
      <td>None</td>
      <td>20.0</td>
      <td>328.0</td>
      <td>230.0</td>
      <td>98.0</td>
      <td>2.0</td>
      <td>1.13</td>
      <td>-1.6</td>
      <td>-0.2</td>
      <td>cle</td>
      <td>2021</td>
      <td>ET</td>
      <td>PT</td>
    </tr>
  </tbody>
</table>
</div>



We currently have separate `Day`, `Date`, `Kickoff_Time`, and `year` columns in `object` format. Since these are `datetime` objects, let's combine `Date` and `year` create a `full_date` column in `datetime` format which will allow us to work with a single date column. (We don't need to include the `Day` column because the `datetime` format will infer the day of week based on the date.)

Let's also convert `Kickoff_Time` to be a `datetime` object to make it easier to work with. All the rows in `Kickoff_Time` contain 'ET' so we discard the time zone portion of the `Kickoff_Time` column to prepare the column to be converted to a `datetime` object.

We actually do not want to combine the date and kickoff time because combining date and time will make it harder to filter by either date or time exclusively without including the other.


```python
# reformat Kickoff_Time column to match datetime object convention
seasons_df['Kickoff_Time'] = seasons_df['Kickoff_Time'].str.extract(r'([^ ]+)')
```


```python
# convert the 'year' column to string
seasons_df['year'] = seasons_df['year'].astype('str')

# combine 'Date' and 'year' columns into a single 'full_date' column
seasons_df['full_date'] = seasons_df['Date'] + ' ' + seasons_df['year']
```


```python
# convert 'full_date' to datetime object
seasons_df['full_date'] = pd.to_datetime(seasons_df['full_date'], format='%B %d %Y')
```


```python
# convert 'Kickoff_Time' to datetime object
seasons_df['Kickoff_Time'] = pd.to_datetime(seasons_df['Kickoff_Time'], format='%I:%M%p').dt.time
```

#### Adjust Time Zones
Now we are ready to make our time zone adjustments. Specifically, we need to:
- Change the time zone for The Rams as the home team from PT to CT for the 1995-2015 seasons
- Change the time zone for The Cardinals as the home or visiting team from MT to PT for all weeks on or after the last week in October through the 2006 season
- Change the time zone for The Cardinals as the home or visiting team from MT to PT for all weeks on or after the first week in November starting with the 2007 season

##### Adjust Ram's Time Zone


```python
# create start and end datetime objects for the rams
from datetime import datetime 
start_date = datetime.strptime('July 1 1995', '%B %d %Y')
end_date = datetime.strptime('July 1 2016', '%B %d %Y')
```


```python
# convert rams timezone from PT to CT for dates based in STL
seasons_df.loc[(
    seasons_df['full_date'] > start_date) & (
    seasons_df['full_date'] < end_date) & (
    seasons_df['team'] == 'ram'), 'time_zone_home'] = 'CT'
```

##### Adjust Cardinals Time Zone
Since October has 31 days the last Sunday of the month will always fall on the 25th or later so through the 2006 season we can set the timezone for The Cardninals to PT starting with any game played on Sunday Oct 25 or later. If the last game of the month The Cardinals played was on a Thursday then we change the timezone for any game played on or after the 22nd, Saturday the 24th, or Monday the 26th of October.

HOWEVER, it seems that PFR got some of the game dates wrong. For example, they record the 1998 wildcard games between The Cowboys and The Cardinals as having been played on a <i>Saturday</i>, January 2. In reality, January 2 was a <i>Friday</i>, both per the datetime object and per [the internet](https://www.dayoftheweek.org/?m=January&d=2&y=1998&go=Go).


```python
seasons_df.loc[(seasons_df['full_date'] == datetime.strptime('January 2 1998', '%B %d %Y')), 'Day']
```




    6809    Sat
    7653    Sat
    Name: Day, dtype: object




```python
seasons_df.loc[(
    seasons_df['full_date'] == datetime.strptime('January 2 1998', '%B %d %Y')), 'full_date'].dt.day_name()
```




    6809    Friday
    7653    Friday
    Name: full_date, dtype: object



On closer inspection we can see that Football Reference has associated the date of January 2 with Saturday, Sunday, and Monday. 


```python
seasons_df.loc[(seasons_df['Date'] == 'January 2'), 'Day'].value_counts()
```




    Day
    Sun    61
    Sat     2
    Mon     1
    Name: count, dtype: int64



Since we cannot be totally sure of the exact day the games with mismatched day of weeks were played, it complicates making time zone adjustments slightly.

Two options we have are:
1. Drop the rows where the true day of week does not match PFR's stated day of week
2. Drop games played by the Cardinals between dates on or around the daylight saving time change for that year, then change the time zone for The Cardinals after that date range. We could drop Cardinals game for the following date ranges:
   - In 2006 or earlier drop Cardinals games played between October 20 and November 5, then realign The Cardinals from MT to PT starting on November 5
   - In 2007 or later drop Cardinals games played between October 27 and November 12, then realign The Cardinals from MT to PT starting on November 12
  
To help make this decision, let's investigate how many rows there are where the true day of week does not match the stated day of week as a percentage of our total rows.


```python
# create dataframe with day of week mismatched rows only
mismatch_df = seasons_df.loc[seasons_df['full_date'].dt.strftime('%a') != seasons_df['Day']]

# number of rows with mismatched day of week as a percent of total rows
mismatched_rows_per = mismatch_df.shape[0] / seasons_df.shape[0] * 100
print(f'Percent of rows with mismatched date: {mismatched_rows_per :.2f}% ')
```

    Percent of rows with mismatched date: 6.70% 


6.7% would be a fairly large number of rows to drop from our dataframe. 

Let's go with option 2 which has the added benefit of having a small buffer between the date in which Arizona changes timezone alignment from MT to PT and games we include in our analysis. Having a buffer makes sense because we shouldn't expect the effects of circadian rhythms to kick in immediately upon time zone realignment. People naturally take some time to adjust to the time change.

Since we are carrying out the same action with two different sets of dates, let's create a function so we don't repeat ourselves. We'll also have to loop through each year in our dataset. As a precaution let's create another function that counts the number of non-Cardinals rows and Cardinals rows before and after we drop rows. The non-Cardinals rows should be unchanged and the Cardinals rows should be reduced by somewhere between 30-90 rows (1-3 rows per year * 30 years. Although the date ranges span two weeks, we might drop only one row if the Cardinals have a bye week during our range).


```python
# get count of non cardinal and cardinal rows before applying function
def count_rows():
    before_cardinals_rows = seasons_df.loc[((seasons_df['Opponent'] == 'Arizona Cardinals') | (
        seasons_df['team'] == 'crd'))].shape[0]

    before_non_cardinals_rows = seasons_df.loc[~((seasons_df['Opponent'] == 'Arizona Cardinals') | (
        seasons_df['team'] == 'crd'))].shape[0]

    print(f'Cardinals rows: {before_cardinals_rows:,}')
    print(f'Non-Cardinals rows: {before_non_cardinals_rows:,}')
```


```python
count_rows()
```

    Cardinals rows: 494
    Non-Cardinals rows: 7,422



```python
# create a function which drops rows in games where the cardinals were the home team or the opponent 
# for a specific set of rows
def drop_card_rows(df, start_date, end_date, years):
    
    for year in years:
        start_date = start_date.replace(year=year)
        end_date = end_date.replace(year=year)
            
        df = df.drop(df.loc[((df['Opponent'] == 'Arizona Cardinals') | (
        df['team'] == 'crd')) & (
        df['full_date'] >= start_date) & (
        df['full_date'] <= end_date)
                  ].index)
        
    return df
```


```python
# set up dates and years to be dropped for Cardinals in 2006 or earlier
years_2006 = list(range(2006,1993, -1))

start_date_cards_2006 = datetime.strptime('October 20', '%B %d')
end_date_cards_2006 = datetime.strptime('November 5', '%B %d')

# set up dates and years to be dropped for Cardinals in 2007 or later
years_2007 = list(range(2023,2006, -1))

start_date_cards_2007 = datetime.strptime('October 27', '%B %d')
end_date_cards_2007 = datetime.strptime('November 12', '%B %d')
```


```python
# apply function to year 2006 and earlier then check row count
seasons_df = drop_card_rows(seasons_df, start_date_cards_2006, end_date_cards_2006, years_2006)
count_rows()
```

    Cardinals rows: 466
    Non-Cardinals rows: 7,422



```python
# apply function to year 2006 and earlier then check row count
seasons_df = drop_card_rows(seasons_df, start_date_cards_2007, end_date_cards_2007, years_2007)
count_rows()
```

    Cardinals rows: 433
    Non-Cardinals rows: 7,422


We lost 61 Cardinals rows (494 - 433) which is right in our expected range.

Now let's adjust the time zone for The Cardinals as the home or visiting team as described above.


```python
# set time zone for Cardinals to PT starting on Nov 5 when they are the visiting team on or before 2006
seasons_df.loc[(seasons_df['Opponent'] == 'Arizona Cardinals') & (
    seasons_df['full_date'].dt.year <= 2006) & 
    (
    (seasons_df['full_date'].dt.month == 11) & (seasons_df['full_date'].dt.day >= 5) |
    (seasons_df['full_date'].dt.month > 11)
    ),
'time_zone_opponent'] = 'PT'

# set time zone for Cardinals to PT starting on Nov 12 when they are the visiting team on or after 2007
seasons_df.loc[(seasons_df['Opponent'] == 'Arizona Cardinals') & (
    seasons_df['full_date'].dt.year >= 2007) & 
    (
    (seasons_df['full_date'].dt.month == 11) & (seasons_df['full_date'].dt.day >= 12) |
    (seasons_df['full_date'].dt.month > 11)
    ),
'time_zone_opponent'] = 'PT'

# set time zone for Cardinals to PT starting on Nov 5 are the home team on or before 2006
seasons_df.loc[(seasons_df['team'] == 'crd') & (
    seasons_df['full_date'].dt.year <= 2006) & 
    (
    (seasons_df['full_date'].dt.month == 11) & (seasons_df['full_date'].dt.day >= 5) |
    (seasons_df['full_date'].dt.month > 11)
    ),
'time_zone'] = 'PT'

# set time zone for Cardinals to PT starting on Nov 12 when they are the home team on or after 2007
seasons_df.loc[(seasons_df['team'] == 'crd') & (
    seasons_df['full_date'].dt.year >= 2007) & 
    (
    (seasons_df['full_date'].dt.month == 11) & (seasons_df['full_date'].dt.day >= 12) |
    (seasons_df['full_date'].dt.month > 11)
    ),
'time_zone'] = 'PT'
```

### Assign time zone to teams and create night game column

To easily compare teams across time zones, we can assign a numerical time zone offset value. To make it easier to filter our dataset for night games, we'll also make a boolean `night_game` column for games that kickoff after 20:00 (8pm) ET.


```python
# add numerical time zone offset value for home team
seasons_df.loc[seasons_df['time_zone_home'] == 'PT', 'offset_home'] = -3
seasons_df.loc[seasons_df['time_zone_home'] == 'CT', 'offset_home'] = -2
seasons_df.loc[seasons_df['time_zone_home'] == 'MT', 'offset_home'] = -1
seasons_df.loc[seasons_df['time_zone_home'] == 'ET', 'offset_home'] = 0

# add numerical time zone offset value for visiting team
seasons_df.loc[seasons_df['time_zone_opponent'] == 'PT', 'offset_opponent'] = 3
seasons_df.loc[seasons_df['time_zone_opponent'] == 'CT', 'offset_opponent'] = 2
seasons_df.loc[seasons_df['time_zone_opponent'] == 'MT', 'offset_opponent'] = 1
seasons_df.loc[seasons_df['time_zone_opponent'] == 'ET', 'offset_opponent'] = 0

# calculate time zone offset difference between home and visiting team
seasons_df['offset_diff'] = abs(seasons_df['offset_home'] + seasons_df['offset_opponent'])

# create night game boolean column
from datetime import time
eight_pm = time(hour=20, minute=0)

seasons_df['night_game'] = seasons_df['Kickoff_Time'] > eight_pm
```

### Combine lines and seasons dataframe

Now we will combine our lines and seasons dataframes. One way to do this is to join on home team, opponent, and the game week of season.

HOWEVER `lines_df` and `seasons_df` represent the game week differently. The `lines_df` dataframe has a `g#` column which corresponds to the count of games that team has played that season up to that point (which ignores bye weeks). Our other dataframe `seasons_df` has a `Week` column which corresponds to the actual week of the season the game was played, including bye weeks. 

How should we proceed? Since 2021, there are 17 weeks in the regular season - prior to the 2021 season there were 16 regular seasons games - and during the regular season each team will only play each other team a maximum of one time at home and one time on the road. During the playoffs each team can only play each other team a maximum of one time. Since teams can play each other at home or on the road more than once pre season (including playoffs), we can't simply join on team, year, and opponent. One approach is do two joins: one for the regular season and one for the playoffs. 

The `seasons_df` has playoff weeks labeled with strings like 'Wildcard' and 'Division'. In the `lines_df` week 18+ is considered the playoffs in 2021 or later, and week 17+ in years 2020 and earlier. We'll make a new column in each dataframe called `playoffs` and set it to `True` for playoff games and `False` for regular season games.

The opponent columns in each dataframe also currently represent teams differently. The `opp` column in the `lines_df` uses an abbreviation to represent teams and the `Opponent` column in the `seasons_df` uses the full city and team name to represent the team. We will need synchronize the team names in these columns before we can join them.

Lastly, we will also have to make sure that the columns we are joining on are of the same type. We will convert the `year` columns in both dataframes to `int`.

To recap, before we can join our dataframes we will:
- Create a boolean `playoffs` column
- Synchronize the values in the opponent name columns in both datframes 
- Ensure that the column types match

Then we'll be able to successfully join our dataframes on team, year, opponent, and playoffs (true/false).

First, let's make a function that counts the number of rows in our dataframes so we can quickly check and see how many rows are lost after we join the dataframes.


```python
# create function to print number of rows
def prn_rows(df_name, df):
    rows = df.shape[0]
    print(f'{df_name} has {rows} rows')
```


```python
# create playoffs column in lines_df
lines_df['playoffs'] = False
lines_df.loc[(lines_df['g#'] > 17) & (lines_df['year'] >= 2021), 'playoffs'] = True
lines_df.loc[(lines_df['g#'] > 16) & (lines_df['year'] < 2021), 'playoffs'] = True

# create playoffs column in seasons_df
seasons_df['playoffs'] = False
playoff_weeks = ['Conf. Champ.', 'Division', 'Wild Card']
seasons_df.loc[seasons_df['Week'].isin(playoff_weeks), 'playoffs'] = True
```


```python
# convert year column in both dfs to int
seasons_df['year'] = seasons_df['year'].astype('int')
lines_df['year'] = lines_df['year'].astype('int')
```


```python
# the LA Rams are listed as both LAR (2016+) and RAM (1994) in lines_df. 
# consolidate team names in order to avoid getting duplicate rows when we join.
lines_df.loc[lines_df['opp'] == 'RAM', 'opp'] = 'LAR'
```

Let's check the number of rows before and after we join the `seasons_df` with the opponent names dataframe


```python
prn_rows('seasons_df', seasons_df)
```

    seasons_df has 7855 rows



```python
# load team name sync dictionary and merge onto seasons df
opp_team_name_sync = pd.read_csv('opp_team_name_sync.csv')

seasons_df = pd.merge(left=seasons_df, right=opp_team_name_sync, left_on='Opponent', right_on='opp_full')

# drop duplicate column
seasons_df = seasons_df.drop(columns=['opp_full'])

prn_rows('seasons_df', seasons_df)
```

    seasons_df has 7855 rows



```python
# some columns in the lines_df also exist in the seasons_df so we can drop them before joining
lines_kept_cols = ['opp', 'spread', 'over/under', 'vs. line', 'ou result', 'team', 'year', 'playoffs']

lines_df = lines_df[lines_kept_cols]
```

Before we join let's check how many rows there are in `lines_df`


```python
# QA check the number of rows in lines df before we join
prn_rows('lines_df', lines_df)
```

    lines_df has 7973 rows


The join should result in a dataframe with a maximum number of rows equal to or lesser than that of the smaller dataframe, in this case the 7,855 rows of the `seasons_df`. If the join results in a table with more than 7,855 rows it means that there are duplicate rows in one or both of the dataframes. As a QA, let's check both dataframes for duplicate rows against the columns we plan on joining on before joining.


```python
# check for duplicate rows across join columns
join_cols = ['team', 'year', 'opp', 'playoffs']
lines_dupes = lines_df.loc[lines_df.duplicated(subset=join_cols, keep=False)].shape[0]
seasons_dupes = seasons_df.loc[seasons_df.duplicated(subset=join_cols, keep=False)].shape[0]

print(f'lines_df duplicate rows: {lines_dupes}, seasons_duplicate rows: {seasons_dupes}')
```

    lines_df duplicate rows: 0, seasons_duplicate rows: 0


Great! No duplicate rows to worry about. Let's join our dataframes and check the number of rows.


```python
# join dataframes
df = pd.merge(left=seasons_df, right=lines_df, on=join_cols)

# check number of rows in df
prn_rows('df', df)
```

    df has 7854 rows


From basic QA checks it looks like the join worked great, with all but one row from the seasons_df being retained. Let's take a look at the column types our new datafame `df`.


```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 7854 entries, 0 to 7853
    Data columns (total 39 columns):
     #   Column              Non-Null Count  Dtype         
    ---  ------              --------------  -----         
     0   Week                7854 non-null   object        
     1   Day                 7854 non-null   object        
     2   Date                7854 non-null   object        
     3   Kickoff_Time        7854 non-null   object        
     4   Result              7854 non-null   object        
     5   OT                  480 non-null    object        
     6   Rec                 7854 non-null   object        
     7   Opponent            7854 non-null   object        
     8   Points_Scored       7854 non-null   object        
     9   Points_Allowed      7854 non-null   object        
     10  1stD_Off            7854 non-null   object        
     11  TotYd_Off           7854 non-null   object        
     12  PassY_Off           7854 non-null   object        
     13  RushY_Off           7854 non-null   object        
     14  TO_Off              6014 non-null   object        
     15  1stD_Def            7854 non-null   object        
     16  TotYd_Def           7854 non-null   object        
     17  PassY_Def           7852 non-null   object        
     18  RushY_Def           7854 non-null   object        
     19  TO_Def              6194 non-null   object        
     20  Exp_Points_Off      7854 non-null   object        
     21  Exp_Points_Def      7854 non-null   object        
     22  Exp_Points_SpTms    7854 non-null   object        
     23  team                7854 non-null   object        
     24  year                7854 non-null   int64         
     25  time_zone_home      7854 non-null   object        
     26  time_zone_opponent  7854 non-null   object        
     27  full_date           7854 non-null   datetime64[ns]
     28  time_zone           106 non-null    object        
     29  offset_home         7854 non-null   float64       
     30  offset_opponent     7854 non-null   float64       
     31  offset_diff         7854 non-null   float64       
     32  night_game          7854 non-null   bool          
     33  playoffs            7854 non-null   bool          
     34  opp                 7854 non-null   object        
     35  spread              7854 non-null   float64       
     36  over/under          7854 non-null   float64       
     37  vs. line            7854 non-null   object        
     38  ou result           7854 non-null   object        
    dtypes: bool(2), datetime64[ns](1), float64(5), int64(1), object(30)
    memory usage: 2.2+ MB


Some of our numerical columns are `object` data type. In order to be able to make calulations with them we should change them to a numerical data type.


```python
# convert numeric columns to numeric data type
df[['Points_Scored', 'Points_Allowed', '1stD_Off', 'TotYd_Off',
       'PassY_Off', 'RushY_Off', 'TO_Off', '1stD_Def', 'TotYd_Def',
       'PassY_Def', 'RushY_Def', 'TO_Def', 'Exp_Points_Off', 'Exp_Points_Def',
       'Exp_Points_SpTms']] = df[['Points_Scored', 'Points_Allowed', '1stD_Off', 'TotYd_Off',
       'PassY_Off', 'RushY_Off', 'TO_Off', '1stD_Def', 'TotYd_Def',
       'PassY_Def', 'RushY_Def', 'TO_Def', 'Exp_Points_Off', 'Exp_Points_Def',
       'Exp_Points_SpTms']].apply(pd.to_numeric)
```

There are two more columns I forsee being useful for our anlysis:
1. A boolean `Result` column where `True` == Win and `False` == Loss, which will allow us to create more versitile functions which can be used for calculating win rates and performance against the spread
2. A Score difference column `score_diff`, i.e. the difference between the home team score and the visiting team (opponent) score. 

Let's make those columns then we're (finally) ready start our anlysis!


```python
# get final score difference, home-team score minus visiting-team socre
df['score_diff'] = df['Points_Scored'] - df['Points_Allowed']
```


```python
# create boolean result column where True == W and False == L
df['result_bool'] = df['Result'] == 'W'
```

## Analysis! 
Now that we've pulled, cleaned, and joined our data, and created some helpful columns, it's time to analyze our data.

### Win rate of west coast teams vs east coast teams in NFL night games
Let's provide an updated answer to the original question investigated in that deadspin article I read so many years ago: how often do west coast teams beat east coast teams in NFL night games? How has this win rate changed over time? Let's focus regular season games for now as some teams might make special adjustments during the playoffs.

#### Create baselines
Since we are invesigating advantages, let's first take a look at home field advantage, which will provide us with a baseline against which we can compare circadian rhythm advantages.


```python
# calculate win rate for home team in regular season games as baseline
home_team_win_rate_reg = df.loc[(df['playoffs'] == False) & (
    df['Result'] == 'W')].shape[0] / df.loc[(df['playoffs'] == False)].shape[0]
print(f'Home teams have beating road teams in regular season games {home_team_win_rate_reg * 100 :.2f}% of the time')
```

    Home teams have beating road teams in regular season games 56.89% of the time


Let's create a function that will allow us easily calculate the win rate of west coast teams vs east coast teams in regular season night games, in home games and road games.


```python
# create function that caclculates win rates for west coast teams vs east coast teams in non-playoff night games
def west_coast_win(col, result, offset):
    '''Returns:
    Dataframe of where west coast team and has won/covered a non-playoff game
    Dataframe of where west coast team and played in non-playoff game
    Win rate for west coast teams. 
    Accepts:
    col: result column name
    result: boolean
    offset: number of hours to offset, set to -3 for west coast team home game and 0 for east coast team home game'''
    if offset == -3:
        offset_opp = 0
    else:
        offset_opp = 3
    
    # calculate win rate for west coast home team vs east coast visiting team in regular season night games
    regseas_nightgame_df = df.loc[(df['playoffs'] == False) & (df['night_game'] == True)]

    # create dataframe of west coast home team wins in regular season night games
    wc_home_team_night_game_win_df = regseas_nightgame_df.loc[(regseas_nightgame_df['offset_opponent'] == offset_opp) & (
        regseas_nightgame_df[col] == result) & (
        regseas_nightgame_df['offset_home'] == offset)
                ]

    # create dataframe of all west coast home team regular season night games 
    wc_home_team_night_game_total_df = regseas_nightgame_df.loc[(regseas_nightgame_df['offset_opponent'] == offset_opp) & (
        regseas_nightgame_df['offset_home'] == offset)
                ]

    # get count of west coast home team wins in regular season night games
    wc_home_team_night_game_win_cnt = wc_home_team_night_game_win_df.shape[0]

    # get count of all west coast home team regular season night games 
    wc_home_team_night_game_total_cnt = wc_home_team_night_game_total_df.shape[0]
    
    win_rate = wc_home_team_night_game_win_cnt / wc_home_team_night_game_total_cnt
    
    return wc_home_team_night_game_win_df, wc_home_team_night_game_total_df, win_rate
```


```python
# calculate win rate of west coast vs east coast teams in night games at home
night_game_home_win_df, night_game_home_total_df, win_rate = west_coast_win('result_bool', True, offset=-3)

print(f'West coast teams beat east coast teams in regular season night games {(win_rate) * 100 :.1f}% of the time when playing at home between 1994 and 2023.')
```

    West coast teams beat east coast teams in regular season night games 60.7% of the time when playing at home between 1994 and 2023.



```python
# calculate win rate of west coast vs east coast teaws in night games on the road
night_game_away_win_df, night_game_away_total_df, win_rate = west_coast_win('result_bool', False, offset=0)

print(f'West coast teams beat east coast teams in regular season night games {(win_rate) * 100 :.1f}% of the time when playing on the road between 1994 and 2023.')
```

    West coast teams beat east coast teams in regular season night games 61.0% of the time when playing on the road between 1994 and 2023.



```python
# create function that calculates overall win rate (home and road) for west coast team vs east coast teams in night games
def west_coast_win_total(home_df_win, home_df_total, road_df_win, road_df_total):
    '''Returns dataframe with west coast home and road wins, win rate for west coast teams'''
    # create dataframe that includes all west coast team wins in regular season night games (home and road)
    wc_night_game_win_df = pd.concat([home_df_win, road_df_win])

    # create dataframe that includes all regular season night games
    night_game_df = pd.concat([home_df_total, road_df_total])

    total_wc_rs_winrate = wc_night_game_win_df.shape[0] / night_game_df.shape[0]
    
    return wc_night_game_win_df, night_game_df, total_wc_rs_winrate
```


```python
wc_night_game_win_df, night_game_df, total_wc_rs_winrate = west_coast_win_total(night_game_home_win_df, 
                                              night_game_home_total_df, 
                                              night_game_away_win_df, 
                                              night_game_away_total_df)

print(f'West coast teams beat east coast teams in regular season night games {total_wc_rs_winrate * 100 :.1f}% of the time (as home or road team) between 1994 and 2023.')
```

    West coast teams beat east coast teams in regular season night games 60.8% of the time (as home or road team) between 1994 and 2023.


Over the past 30% years, being a west coast team vs an east coast team (as home or visting team) has been more advantageous than being the home team alone in the NFL.


#### Win rate over time
Now let's look at the win rate of west coast teams vs east coast teams by year. To do this, we'll create a couple of functions:
1. The first function will calculate and return a list of years and a list of associated win rates. It will also be able to return win rate over a span of years which will be useful if year-to-year win rate is too volatile to easily interpret (by calculating and returning a list of list of multiple years and a list of associated win rates)
2. The second function will plot the data returned by the first function


```python
# create function that returns win rate of west coast teams vs east coast teams
def year_loop(wc_night_game_win_df, night_game_df, years, year_span=5):
    '''Returns a win rate by year or span of years. Accepts:
    years: list of years
    year_span: int, number of years included in the span'''
    
    # create lists to save data in
    yr_list = []
    wr_list = []

    for year in years:
        # set year spans 
        yrs = list(range(year,year+year_span, 1))
        
        # get total west coast night game wins during five year span
        total_wins_cnt = wc_night_game_win_df.loc[wc_night_game_win_df['year'].isin(yrs)].shape[0]
        
        # get total night games played during five year span
        total_games_cnt = night_game_df.loc[night_game_df['year'].isin(yrs)].shape[0]
        
        # print change in west coast night game win rate by five year period
        if total_games_cnt == 0:
            continue
            
        # append data to lists
        elif len(yrs) == 1:
            yr_list.append(yrs[0])
            wr_list.append(total_wins_cnt/total_games_cnt)
        else:
            yr_list.append(yrs)
            wr_list.append(total_wins_cnt/total_games_cnt)
            
    return yr_list, wr_list
```


```python
# create plotting function
def plot_winrate(yr_list, wr_list, yr_range=False, custom_title=None, custom_yaxis=None):
    '''Returns a plot of win rate by year. Accepts: 
    yr_list: list of years or list of list of years (if plotting range)
    wr_list: list of win rates
    yr_range: boolean, set to True if plotting range of years
    custom_title: str, custom title label
    custom_yaxis: str, custom y axis label
    '''
    
    # copy yrs into year_list so that both individual years and year ranges use the same list (see directly below)
    year_list = yrs.copy()
    
    # rewrite year_list if range of years
    if yr_range is True:
        year_list = []
        for yr in yrs:
            year_list.append(f'{yr[0]} - {yr[-1]}')
            
    # create a plot
    plt.figure(figsize=(10, 6))
    plt.plot(year_list, wrs, marker='o', linestyle='-', color='b', label='Values over Time')

    # add titles and labels and rotate labels on x axis
    plt.title('Win rate of west coast teams vs east coast teams in NFL night games by year since 1994')
    if custom_title is not None:
        plt.title(custom_title)
    plt.xlabel('Year')
    plt.xticks(rotation=75)
    plt.ylabel('West coast team win rate')
    if custom_yaxis is not None:
        plt.ylabel(custom_yaxis)
    
    return plt
```


```python
# calcuate win rate by year
years = range(1994,2024, 1)
yrs, wrs = year_loop(wc_night_game_win_df, night_game_df, years, year_span=1)
```


```python
# enable plotting
%matplotlib inline

# plot win rate by year
import matplotlib.pyplot as plt
_ = plot_winrate(yr_list=yrs, wr_list=wrs)
```


    
![png](https://github.com/ivaronin/ivaronin.github.io/blob/master/images/output_132_0.png?raw=true)
    


The plot is a bit jerky year to year, but it appears that west coast teams had a big advatange most years from 1994 through 2017, when it seems like the advantage diminishes considerably. Let's plot this data using a rolling 5 year X axis to see if it's easier to interpret.


```python
# calculate win rate west coast vs east coast regular season win rate by rolling five year period

# set range from first season until five years before 2023 season
years = range(1994,2020, 1)

yrs, wrs = year_loop(wc_night_game_win_df, night_game_df, years=years, year_span=5)
```


```python
# plot win rate by rolling five year period
_ = plot_winrate(
    yr_list=yrs, 
    wr_list=wrs,
    yr_range=True, 
    custom_title='Rolling 5 year avg win rate of west coast teams vs east coast teams in NFL night games since 1994')
```


    
![png](https://github.com/ivaronin/ivaronin.github.io/blob/master/images/output_135_0.png?raw=true)
    


Wow! It's certainly clear now that the advantage west coast teams had over east coast teams in night games in the NFL has all but disappeared in the last few years. It's less clear if this is a temporary shift or a permanent one: a similar trend manifested in the 2000s then the west coast advantage exploded again in the early to mid 2010s. 

### Point spread cover rate of west coast teams when playing east coast teams in night games in the NFL?

Let's investigate how often west coast teams have covered the point spread vs east coast teams in night games. To be able to reuse our functions, we'll want to create a boolean version of the `vs. line` (vs point spread) column where `True` == `Won` and `False` == `Lost`


```python
# convert vs. line col to boolean
df['line_bool'] = df['vs. line'] == 'Won'
```


```python
# get home and road point spread cover data and rates
night_game_home_cover_df, night_game_home_total_cover_df, home_cover_rate = west_coast_win('line_bool', True, -3)
night_game_away_cover_df, night_game_away_total_cover_df, road_cover_rate = west_coast_win('line_bool', False, 0)
```


```python
# print home and road point spread cover rates
print(f'West coast teams cover the spread vs east coast teams in regular season night games {(home_cover_rate) * 100 :.1f}% of the time when playing at home between 1994 and 2023.')
print('\n')
print(f'West coast teams cover the spread vs east coast teams in regular season night games {(road_cover_rate) * 100 :.1f}% of the time when playing one the road between 1994 and 2023.')
```

    West coast teams cover the spread vs east coast teams in regular season night games 55.7% of the time when playing at home between 1994 and 2023.
    
    
    West coast teams cover the spread vs east coast teams in regular season night games 70.7% of the time when playing one the road between 1994 and 2023.


It's interesting that west coast teams have covered the spread when playing on the road more often than when playing at home in NFL regular season night games. You could have made a lot of money if you bet on west coast teams on the road in night games during the past three decades. 

Let's see how often west coast teams covered the spread in general (as the home or visiting team).


```python
wc_night_game_cover_win_df, night_game_cover_df, total_wc_cover_winrate = west_coast_win_total(night_game_home_cover_df, 
                                                                                               night_game_home_total_cover_df, 
                                                                                               night_game_away_cover_df, 
                                                                                               night_game_away_total_cover_df)

print(f'West coast teams cover the spread vs east coast teams in regular season night games {(total_wc_cover_winrate) * 100 :.1f}% of the time (as home or road team) betweem 1994 and 2023.')
```

    West coast teams cover the spread vs east coast teams in regular season night games 61.8% of the time (as home or road team) betweem 1994 and 2023.


Covering the point spread more than 60% of the time provides a nice edge to bettors. Let's take a look at how this data has trended over the past 30 years.


```python
years = range(1994,2024, 1)
yrs, wrs = year_loop(wc_night_game_cover_win_df, 
                     night_game_cover_df, years, 
                     year_span=1)
_ = plot_winrate(yr_list=yrs, 
                 wr_list=wrs,
                 custom_title='Avg point spread cover rate of west coast teams vs east coast teams by year in NFL night games since 1994',
                 custom_yaxis='West coast team cover rate')
```


    
![png](https://github.com/ivaronin/ivaronin.github.io/blob/master/images/output_143_0.png?raw=true)
    


As with win rate, it's a bit hard to interpret cover rate year to year but it does appear to be converging around 50% in recent years. Let's again create a plot that uses 5 year rolling average on the X axis.


```python
years = range(1994,2020, 1)
yrs, wrs = year_loop(wc_night_game_cover_win_df, night_game_cover_df, years=years, year_span=5)
# plot win rate by rolling five year period
_ = plot_winrate(
    yr_list=yrs, 
    wr_list=wrs,
    yr_range=True, 
    custom_title='Rolling 5 year avg point spread cover rate of west coast teams vs east coast teams in NFL night games since 1994',
    custom_yaxis='West coast team cover rate')
```


    
![png](https://github.com/ivaronin/ivaronin.github.io/blob/master/images/output_145_0.png?raw=true)
    


Very interesting. As with win rate, a once massive advantage for west coast teams has been virtually eliminated in betting markets in recent years.

## Final thoughts and potential next steps

Both in terms of real-game win rate and win-rate against the spread, the overwhelming advantage enjoyed for years by west coast teams has all but disappeared. It is not clear if these are temporary or permanent changes. 

There was more I originally planned to do with this blog post like investigate the advanatage the western-most team has in night games where there is only a two or one hour difference between time zones. However, since this post has gotten quite long, I'm going to leave it here for now. In a future post, I might dig into this and use some of the other data we have available like margin of victory and yards for and yards against. It might also be interesting to investigate this trend in other US American sports leagues like the NBA, the MLB, college football, and the NHL.

It could also be intersting to investigate how these statistics change when the eastern-most (disadvantaged team) is coming off a bye or playing for the first game of the season (when they have had more time to prepare) or how these statistics change when the game is played during the playoffs.

Thanks for reading! 
