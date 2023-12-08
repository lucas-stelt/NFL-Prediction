# NFL Predictions

https://www.kaggle.com/datasets/kendallgillies/nflstatistics 

https://www.kaggle.com/datasets/speckledpingu/nfl-qb-stats 

https://www.kaggle.com/datasets/mitchellweg1/nfl-combine-results-dataset-2000-2022

### 2009 - 2016

Use all player stats and be able to predict whether someone would be a good candidate for the NFL. Want to create a model to predict if a given person would be a good fit for the NFL, predict hall of fame, predict years in NFL, and more.

Look into college stats - join college stats to NFL stats and use Ttest to determine performance difference, if applicable.

Use structural equation modeling to find links between variables.

### Find factors which contribute to overall player success

* Determine overall success

* Find factors which contribute using exploratory factor analysis

* Combine data (training and test data)


### Methods Applied

* Multivariate regression

* Multivariate T test

* Factor analysis

* Model building 

* Structural Equation Modeling


### Steps

1. Merge multiple datasets by player name to create dataset optimal for analysis

2. Clean data

3. Run analysis

## 1. Data Cleaning/Creation

#### Career Stats

We have a lot of different data sources and we want to combine them all into one dataframe to run analysis on.

First, we started with a .csv which included stats of every game for each year. To make this data ready for analysis, I found the sum of each variable for each quarterback to get their overall season stats.

```
qb09 = read.csv("QBStats_2009.csv") %>%
  select(-lg) %>%
  group_by(qb) %>%
  summarize(att = sum(att),  cmp = sum(cmp), yds = sum(yds), ypa = sum(ypa), td = sum(td), int = sum(as.numeric(int)), sack = sum(sack), loss = sum(loss), game_points = sum(game_points)) %>%
  na.omit()
qb09$year = rep(2009, each = nrow(qb09)) #year column
```

I then repeated this process for each of the 7 years and then combined them into one large dataframe, which is the qbStats.csv file in this repository.

```
qbCombined = rbind(qb09, qb10, qb11, qb12, qb13, qb14, qb15, qb16)
```

#### Combine Stats

Next we found a combine dataset which comprised of the combine data. !FILL IN LATER!

Unfortunately, the names of the players were formatted differently in these 2 datasets, so I had to use some RegEx to fix them.

The nfl stats had names formatted as "Firstname Lastname.FirstInitial Lastname" while the combine stats had the format of "Firstname Lastname-FirstinitialLastname"

*Example:*

* NFL Stats: "Aaron Rodgers.A Rodgers"

* Combine Stats: "Aaron Rodgers-ARodgers"

* Desired Output: "Aaron Rodgers"

* I was able to use the following code to achieve this:

```
qbCombined$qb = sub("[A-Z]\\..*", "", qbCombined$qb) #nfl players

combine2000 = read.csv("combine_qb2000.csv")
combine2000$`Player` = sub("-.*", "", combine2000$`Player`) #combine players
```
Now that the names match, it is simple enough to filter the combine data to only the players present in the NFL data.

```
results = combine2000 %>%
  filter(combine2000$Player %in% qbCombined$qb)
```

With this done, I was then able to create a dataframe of the players NFL stats and Combine stats.

```
qbTotal = qbCombined %>%
  select(-year) %>%
  group_by(qb) %>%
  summarize(att = sum(att),  cmp = sum(cmp), yds = sum(yds), ypa = sum(ypa), td = sum(td), int = sum(as.numeric(int)), sack = sum(sack), loss = sum(loss), game_points = sum(game_points)) %>%
  na.omit()

names(results)[names(results) == "Player"] <- "qb"

combineQBCombined = merge(qbTotal, results, by = "qb")
```

Finally, I wanted to combine the combine stats to the player's overall NFL stats. I first created an overall stats dataframe by summing the yearly stats and merge them with the combine stats.

```
qbStatsSum = qbStats %>%
  select(-X, -ypa) %>%
  group_by(qb) %>%
  summarize(att = sum(att),  cmp = sum(cmp), yds = sum(yds), ypa = sum(yds)/sum(att), td = sum(td), int = sum(as.numeric(int)), sack = sum(sack), loss = sum(loss), game_points = sum(game_points), experience = max(experience)) %>%
  na.omit()

career_combineStats = merge(qbStatsSum, totalCombinedQB, by = "qb")
```

This resulted in the **career_combine.csv** file which analysis was run on.
