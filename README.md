# Clustering NBA Players to Identify Team Roles

## The Problem

The National Basketball Association(NBA) consists of 30 teams with ~450 players during the season. Each team is permitted 15 roster spots, and can acquire new players by either trading with other teams, signing a free agent, or through the draft. Understanding the contribution the player will make to a team is critical in roster decision making. Sometimes perceptions distort a players relevance and a player becomes either overvalued or undervalued. A players relevance is dynamic, and it can be difficult to identify what kind of role each player still deserves on a team, or has a potential to attain on the team.

The following anlaysis will identify player roles in relation to their relevance and contributions to their teams. Identifying roles such as "Starting Player" , "Reserve Player", or "Role Player" based on game statistics can highlight patterns within the groups to answer some complicated questions such as:

- How late in the draft could I find a star player?
- How many points per game can I expect on average from a role player?
- How many players are still considered star players over age 35?
- Who are the players at the cusp of increasing their role?

## Clustering

For this analysis, I will be using K-Means Clustering. While there are other forms of clustering such as aglomerative, K-Means is a popular and widely used method which is both suitable and easily understood for this analysis. Clustering is an unsupervised machine learning technique used to assign subsets of data to specific categories(clusters). In this case, I will be assigning players to role categories which we use with vague measurements such as "Star Player" or "Role Player" based on their game statistics. K-Means is one method of clustering that forms it's clusters by measuring distances from a center point. If only 2 categories are at question, two random centers would be found and distances from all datapoints would be found in relation to the two centers. The datapoint is assigned a cluster with the center that is closest. This process is repeated until there is the smallest possible variation amoung the two clusters. While the clusters represent the arbitrary categories we define, it is up to us to identify which cluster is which category after they are formed.

## The Data

The data I use for this analysis is a Kaggle Dataset found [here](https://www.kaggle.com/datasets/justinas/nba-players-data) made available by Justinas Cirtautas who utilized the NBA Stats API and filled in any missing gaps with [Basketball Reference](https://www.basketball-reference.com/). It contains rows for each player that has been on an NBA teams' roster from 1996 - 2022. Each row contains stats for a player in a single season. There are numerous statistical averages such as points, rebounds, assits, net-rating, usage percentage, and others. There is also non-statistical information such as draft number, college attended, age, height, etc.

## Data Understanding

I begin by selecting only the 2021-22 NBA season in order to make my unit of analysis individual players. Next, I identify categorical datatypes for the feautures of Player Name, Team, Country, Season, Draft Year, Draft Number, and Draft Round. K-Means Clustering requires data to be numerical types, so this is something to consider cleaning. I found that the 'Draft Number' feature contained the string 'Undrafted' for certain players and the 'College' feature contained the string 'None' for certain players. There were no missing values in the dataset. No missing values is great, because that means nothing has to be dropped or estimated in order to use the model.

## Data Cleaning

In an attempt to use a players draft number, I need to change this into a numerical type. In order to do so, the 'Undrafted' players have to be dealt with. I find the highest drafted number to be '60' and assign all the undrafted players a value of '65'. I didnt want to skew the scale drastically, but I wanted to padd their assigned 'draft number' further from the standard one position away to account for them not being in the draft at all. During this process, I found there was a player with a draft number of '0'. Since it is not possible for a player to be drafted with a #0 pick, I recoded all palyers with a draft number of 0 to also be 65 and represent 'Undrafted'. This method is repeated for 'Draft Round' and with 2 being the highest draft round, '3' was used for the undrafted players.

## Feature Engineering

I created two new features in this dataset. The first was whether or not a player played college basketabll. If the 'College' column contained 'None', the player received a '0' in this new binary column, otherwise they received a '1'.

The second feature was the time a player has spent in the NBA. This required four steps
- Create the 'NBA Years' column as a copy the draft year column 
- Replace the undrafted players with a value of '0'
- Subtract every players draft year from the current season year '2022' and use those values as their 'NBA Years'
- If a player was undrafted (currently labelled as 0 years) replace their value with the mean value of all 'NBA Years'.

## Modelling

To use K-Means Clustering, the features must be represented in a two dimensional space. In order to account for multiple features, I conduct a principle component analysis(PCA) for dimensionality reduction. There were three different experiments performed. The first two used the same PCA, with different amount of clusters. The PCA of the first two experiments included the following features:

- Age
- Player Height
- Player Weight
- Draft Round
- Draft Number
- Games Played
- Points per Game
- Rebounds per Game
- Assists per Game
- Offensive Rebounds per game
- Defensive Rebounds per game
- Usage Percentage
- Shooting Efficiency (ts_pct)
- Assist Percentage
- Played College
- NBA Years

The feature 'Net Rating' was identified to contain outliers within the initial PCA and was removed in order to maintain integrity of the clusters. If those outliers were to remain, the cluster centers would be forcefully moved by the outliers and miscategorize some players. 

The result yielded the following scatterplot of all data points:

![download](https://user-images.githubusercontent.com/93283651/200421652-2096b816-ac88-4c8d-92cd-5297fcd67adb.png)


From there, the K-Means algorithm was fit with various cluster sizes to identify the most subsets that are homogeneous. The code and resulting plot are as follows:

``python
 inertia = []
for k in range(1,8):
    kmeans = KMeans(n_clusters=k, random_state=1).fit(X)
    inertia.append(np.sqrt(kmeans.inertia_))
``

![download](https://user-images.githubusercontent.com/93283651/200421697-86292087-301a-4d4b-b4df-88a329755874.png)

The plot shows that the most amount of homogenous clusters within the dataset is approximately 2 or 3. 

## Cluster Analysis

Using the current PCA, 2 and 3 clusters were both tested. The results of those clusters are shown below:

![download](https://user-images.githubusercontent.com/93283651/200422152-2cf1107b-b4bd-4dc4-be2d-0075fc53c8f9.png)

The Classification for two clusters would be interpreted as "Starter" and "Non-Starter"

![download](https://user-images.githubusercontent.com/93283651/200422346-ff1235d1-7230-4a2a-944d-c69daedda6a8.png)

The Classification for three clusters would be "Starter", "Role Player", "Reserve Player"

**Even though there may be less distinction in three clusters than two, it is easier to take away actionable insight from three clusters. Since each team has 15 players on a roster, we can not easily divide that roster in half for classification. Additionally, each team has 5 players that are designated as "Starters" which are typically the highest performing players on each team. Forming 3 clusters fits this conceptual mold and makes more sense.

To evaluate these clusters, I create some basic visualizations to compare Averages and Minimums of the three. I am looking to distinguish which cluster is which classification. The bar chart for the minimum averages was the most informative and is below:

![download](https://user-images.githubusercontent.com/93283651/200422470-6cef72aa-32d2-49eb-8ba0-1154f005df6e.png)

*Recall that our classifications are "Starter", "Role Player", and "Reserve Player". We can expect to see some variation in these statistical categories for each cluster. These categories represent season averages for a player. The above chart is showing the Lowest Season Average of a player in each of these clusters. This would mean that the if cluster 1 is the "Starter" cluster, there is a Starter who averaged a minimum of 2 points per game. This indicates a failed clustering attempt, because logically no team would start any player who averaged 2 points per game.

This interpretation lead to the third experiment where the features that were engineered previously were disregarded. I wanted to disregard any non game performance metrics in my model which resulted in only the following being used in the next PCA:

- Points per game
- Rebounds per game
- Assists per game
- Offensive rebounds per game
- Defensive rebounds per game
- Usage Percentage
- Shooting Efficiency
- Assist Percentage

![download](https://user-images.githubusercontent.com/93283651/200422610-73ffb86c-2da9-48a1-b858-6f0144aad4fc.png)

The resulting clusters are below followed by Comparisons of Statistical Averages and Minimums of each cluster:

![download](https://user-images.githubusercontent.com/93283651/200422696-e1e30226-3682-4b7f-b164-ca24340c26d2.png)

![download](https://user-images.githubusercontent.com/93283651/200422859-fbcc5475-ee9a-4d58-82ca-9fb9cb81edad.png)

![download](https://user-images.githubusercontent.com/93283651/200423170-3f437c27-e9d8-4733-bde4-2f74f9f16376.png)

*Using only performance metrics in the model resulted in more accurate clustering of players for these classifications. We can see that the minimum average points for the "Starters" has improved from ~2 points per game to 15 points per game. This intuitively makes a lot more sense. We can also see that the averages decrease from Starters to Reserve Players which also intuitively makes sense and fits our classification mold. I also create an interactive plot which I manually explore to verify the accuracy of these clusters based on my personal knowledge of the game. The code for that plot can be found in the notebook in this repository.

After exploring the clusters and satisfied with the results, we can start asking questions about the clusters.
One of the questions I had was "What is the furthest in the draft that we can find a Starter?" Looking through the cluster of starters, there were two players that were amoung the group who were drafted at #46. This can be valuable information, because as teams evaluate their own draft picks they can identify a potential cutoff for finding a Starting player. However, this type of information is not definate and does not imply that Starting players can not be obtained beyond #46.

## Impact

Positive: The successful clustering of this model can help teams identify where certain players fall within arbitrary roles. Further exploration into the players that lie on the cusp of a different role can help aid a player in finding ways to improve their performance. That same exploration can potentially identify undervalued players that are over-achieving for what their current role is perceived to be. There is no clear measure of what a "Starting Player" must have. Each team has different needs, and each player has different skillsets. The ability to find the seperation of these three categories allows for enahnced development and decision making.

Negative: This model can be used to find undistinguishable separations between players, and then compare out of game statistics such as average draft position. This analysis explored the potential cutoff of finding a starting player in the draft, but taking this kind of information as facts can lead to wrong decisions. The characteristics of these clusters are unique to the sample analyzed and are not predicitive of future player samples. Additionally, the sample does not consider previous players before the 2021-22 season which could be holding additional insights to improve this model.


## Conclusion

It is possible to cluster NBA players into team roles with no distinct lines of measure. When doing so, the most successful attempts will be utilizing only in-game statistcal performances. The insights derived from these clusters may aid in making decisions based on the current sample, but are not predicitive of future samples. Further anaylsis could be completed to reduce the clustering samples by position.

*See the jupyter notebook in this repository to access the code used in this analysis.
