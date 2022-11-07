# Clustering NBA Players to Identify Team Roles

## The Problem

The National Basketball Association(NBA) consists of 30 teams with ~450 players during the season. Each team is permitted 15 roster spots, and can acquire new players by either trading with other teams, signing a free agent, or through the draft. Understanding the contribution the player will make to a team is critical in roster decision making. Sometimes perceptions distort a players relevance and a player becomes either overvalued or undervalued. A players relevance is dynamic, and it can be difficult to identify what kind of role each player still deserves on a team, or has a potential to attain on the team.

The following anlaysis will identify player roles in relation to their relevance and contributions to their teams. Identifying roles such as "Starting Player" , "Reserve Player", or "Role Player" based on game statistics can highlight patterns within the groups to answer some complicated questions such as:

- How late in the draft could I find a star player?
- How many points per game can I expect on average from a role player?
- How many players are still considered star players over age 35?
- Who are the players at the cusp of increasing their role?

## Clustering

For this analysis, I will be using K-Means Clustering. Clustering is an unsupervised machine learning technique used to assign subsets of data to specific categories(clusters). In this case, I will be assigning players to role categories which we use with vague measurements such as "Star Player" or "Role Player" based on their game statistics. K-Means is one method of clustering that forms it's clusters by measuring distances from a center point. If only 2 categories are at question, two random centers would be found and distances from all datapoints would be found in relation to the two centers. The datapoint is assigned a cluster with the center that is closest. This process is repeated until there is the smallest possible variation amoung the two clusters. While the clusters represent the arbitrary categories we define, it is up to us to identify which cluster is which category after they are formed.

## The Data

The data I use for this analysis is a Kaggle Dataset found ![here](https://www.kaggle.com/datasets/justinas/nba-players-data) made available by Justinas Cirtautas who utilized the NBA Stats API and filled in any missing gaps with ![Basketball Reference](https://www.basketball-reference.com/). It contains rows for each player that has been on an NBA teams' roster from 1996 - 2022. Each row contains stats for a player in a single season. There are numerous statistical averages such as points, rebounds, assits, net-rating, usage percentage, and others. There is also non-statistical information such as draft number, college attended, age, height, etc.

## Data Understanding

I begin by selecting only the 2021-22 NBA season in order to make my unit of analysis individual players. Next, I identify categorical datatypes for the feautures of Player Name, Team, Country, Season, Draft Year, Draft Number, and Draft Round. K-Means Clustering requires data to be numerical types, so this is something to consider cleaning. I found that the 'Draft Number' feature contained the string 'Undrafted' for certain players and the 'College' feature contained the string 'None' for certain players. There were no missing values in the dataset. No missing values is great, because that means nothing has to be dropped or estimated in order to use the model.

## Data Cleaning

In an attempt to use a players draft number, I need to change this into a numerical type. In order to do so, the 'Undrafted' players have to be dealt with. I find the highest drafted number to be '60' and assign all the undrafted players a value of '65'. I didnt want to skew the scale drastically, but I wanted to padd their assigned 'draft number' further from the standard one position away to account for them not being in the draft at all. During this process, I found there was a player with a draft number of '0'. Since it is not possible for a player to be drafted with a #0 pick, I recoded all palyers with a draft number of 0 to also be 65 and represetn 'Undrafted'. This method is repeated for 'Draft Round' and with 2 being the highest draft round, '3' was used for the undrafted players.

## Feature Engineering

I created two new features in this dataset. The first was whether or not a player played college basketabll. If the 'College' column contained 'None', the player received a '0' in this new binary column, otherwise they received a '1'.

The second feature was the time a player has spent in the NBA. This required four steps
- Create the 'NBA Years' column as a copy the draft year column 
- Replace the undrafted players with a value of '0'
- Subtract every players draft year from the current season year '2022' and use those values as their 'NBA Years'
- If a player was undrafted (currently labelled as 0 years) replace their value with the mean value of all 'NBA Years'.

## Modelling

## Cluster Analysis

## Impact

Positive: This model can help teams identify where certain players fall within arbitrary roles. Further exploration into the players that lie on the cusp of a different role can help aid a player in finding ways to improve their performance. That same exploration can potentially identify undervalued players that are over-achieving for what their current role is perceived as. 

Negative: This model can be used to find undistinguishable separations between players, and then compare out of game statistics such as average draft position. While a question explored in this analysis was "What is the furthest position a star player can be drafted?" it is important to understand that the answer to this question is not definate. Taking any information from this model as facts can disregard the sample bias that exists within the data provided.


## Conclusion

*See the jupyter notebook in this repository to access the code used in this analysis.
