Project for DSC80 at UCSD | Kyle Dang

## Introduction

League of Legends is a competitive multiplayer online battle arena (MOBA) game, where two teams of five players aim to destroy the opposing team's base. In professional esports, understanding what drives a team to victory is a key question for analysts, coaches, and fans.

This project analyzes the **2025 League of Legends Esports Match Data** from Oracle's Elixir, which contains detailed statistics for professional matches played in 2025. The dataset contains **120,636 rows** and **165 columns**, where each row represents either a single player's performance or a team's aggregate performance in a game.

The central question of this project is: **Does having a gold advantage at 15 minutes predict whether a team will win the game?**

Gold is one of the most important resources in League of Legends — it allows teams to buy items that makes individual champions stronger. The benchmark of 15 minutes is appropriate since it is at this phase of the game where players begin contesting for map objectives and largely marks the end of the "laning" or "leveling" phase. A gold lead at 15 minutes indicates early game dominance, but does it actually translate to winning?

The relevant columns for our analysis are:

| Column             | Description                                                                                                                                          |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `result`           | Game outcome: 1 = win, 0 = loss                                                                                                                      |
| `golddiffat15`     | Gold difference between the team and their opponent at 15 minutes                                                                                    |
| `xpdiffat15`       | Experience difference at 15 minutes - XP is gained by being near creature deaths, and determines a champion's level which unlocks stronger abilities |
| `csdiffat15`       | Creep score difference at 15 minutes - CS measure the number of creatures killed, which is the primary way players earn gold through farming         |
| `golddiffat10`     | Gold difference at 10 minutes                                                                                                                        |
| `killsat15`        | Number of kills at 15 minutes                                                                                                                        |
| `firstblood`       | Whether the team secured first blood                                                                                                                 |
| `firstdragon`      | Whether the team secured first dragon                                                                                                                |
| `datacompleteness` | Whether the game data is complete or partial                                                                                                         |

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The raw dataset contains 120,636 rows representing both individual player rows and team aggregate rows. We performed the following cleaning steps:

1. **Filtered to team rows only** — since our question is about team-level outcomes, we kept only rows where `position == 'team'`, giving us 20,106 rows where each row represents one team's performance in one game.

2. **Removed partial games** — we filtered out rows where `datacompleteness == 'partial'` since these games were missing all early game statistics like `golddiffat15`. This left us with 18,472 complete team rows representing 9,236 unique games.

3. **Created a gold advantage indicator** — we created a binary column `gold_positive` indicating whether a team had a positive gold difference at 15 minutes, which was used in our hypothesis test.

The head of our cleaned DataFrame is shown below:

| gameid           | result | golddiffat15 | xpdiffat15 | csdiffat15 | playoffs |
| :--------------- | -----: | -----------: | ---------: | ---------: | -------: |
| LOLTMNT03_179647 |      0 |        -3837 |       -469 |        -16 |        0 |
| LOLTMNT03_179647 |      1 |         3837 |        469 |         16 |        0 |
| LOLTMNT06_96134  |      1 |         5069 |       2014 |         64 |        0 |
| LOLTMNT06_96134  |      0 |        -5069 |      -2014 |        -64 |        0 |
| LOLTMNT06_95160  |      0 |          118 |       1990 |        -43 |        0 |

### Univariate Analysis

The distribution of gold difference at 15 minutes across all team rows is approximately normal and centered around 0, which makes sense since every game has one team with a positive gold diff and one with the exact negative value.

<iframe
  src="assets/golddiff_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of game length is right skewed, with most games lasting between 25-33 minutes. Very few games end before 20 minutes or go beyond 50 minutes in professional play.

<iframe
  src="assets/gamelength_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

Winning teams tend to have a higher gold difference at 15 minutes compared to losing teams, though there is overlap between the two distributions suggesting gold diff alone does not guarantee a win.

<iframe
  src="assets/golddiff_by_result.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

The table below shows the average gold difference at 15 minutes for winning teams and the win rate for gold-positive teams, grouped by league:

| league | avg_golddiff15_winners | win_rate_gold_positive | num_games |
| :----- | ---------------------: | ---------------------: | --------: |
| PRMP   |                2997.03 |                   0.42 |       146 |
| LJL    |                2335.81 |                   0.40 |       748 |
| LPLOL  |                2254.51 |                   0.40 |       320 |
| EBL    |                2034.67 |                   0.39 |       382 |
| HM     |                1974.10 |                   0.39 |       380 |
| PCS    |                1898.94 |                   0.39 |       606 |
| KeSPA  |                1794.12 |                   0.37 |       112 |
| LAS    |                1663.04 |                   0.37 |       978 |
| HLL    |                1653.47 |                   0.38 |       442 |
| LRN    |                1642.58 |                   0.38 |       302 |

Leagues like PRMP and LJL tend to have larger gold advantages for winning teams, suggesting more dominant early game performances in those regions compared to more competitive leagues.

## Assessment of Missingness

### MNAR Analysis

We believe the `url` column in the dataset is likely **MNAR (Missing Not at Random)**. The URL field links to match recap pages, and its missingness is likely tied to whether a league or tournament has an official broadcast partner that publishes match pages — information that is not captured in the dataset itself.

### Missingness Dependency

We analyzed the missingness of `golddiffat15` by running permutation tests against other columns.

**Depends on `datacompleteness`** (p-value = 0.0): Games marked as "partial" are missing all gold difference data, while complete games have none missing. This is a clear MAR dependency.

**Does not depend on `side`** (p-value = 1.0): Missingness in `golddiffat15` is perfectly balanced between blue and red side teams, meaning which side a team plays on has no relationship to whether their gold data is missing.

<iframe
  src="assets/missingness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Hypothesis Testing

**Null Hypothesis:** Teams that are ahead in gold at 15 minutes win at the same rate as teams that are behind in gold at 15 minutes; any observed difference is due to random chance.

**Alternative Hypothesis:** Teams that are ahead in gold at 15 minutes win at a higher rate than teams that are behind.

**Test Statistic:** Difference in win rates between gold-positive teams (golddiffat15 > 0) and gold-negative teams (golddiffat15 < 0).

**Significance Level:** alpha = 0.05

**Result:** We observed a difference in win rates of **0.4577** (73% win rate for gold-positive teams vs 27% for gold-negative teams). After running 1000 permutations, the p-value was **0.0** — not a single permutation produced a difference as large as the observed one by chance.

**Conclusion:** We reject the null hypothesis. The data strongly suggests that having a gold advantage at 15 minutes is associated with a significantly higher win rate. However, we cannot conclude causation — both winning and having a gold lead may be driven by an underlying factor such as overall team skill.

<iframe
  src="assets/hypothesis_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Framing a Prediction Problem

**Prediction Problem:** Can we predict whether a team will win or lose a League of Legends match using only early game statistics available at or before 15 minutes?

**Type:** Binary Classification — the response variable `result` takes on two values: 0 (loss) or 1 (win).

**Response Variable:** We chose `result` because it is the most direct measure of game outcome and is ultimately what teams and analysts care about most.

**Features:** We only use statistics available at or before 15 minutes into the game, such as `golddiffat15`, `xpdiffat15`, `csdiffat15`, `golddiffat10`, `killsat15`, `firstblood`, and `firstdragon`. We cannot use end-game statistics like total kills, towers destroyed, or total gold earned, because those would not be known at the time of prediction.

**Evaluation Metric:** We use **accuracy** as our primary metric. Since the dataset is perfectly balanced — every game has exactly one winning team and one losing team — accuracy is a fair and appropriate metric and will not be misleading. We also report **F1-score** as a secondary metric to confirm our results.

## Baseline Model

Our baseline model is a **Logistic Regression** classifier implemented in a single sklearn Pipeline with a StandardScaler preprocessing step.

**Features used:**

- `golddiffat15` (quantitative) — gold difference at 15 minutes
- `xpdiffat15` (quantitative) — experience difference at 15 minutes
- `csdiffat15` (quantitative) — creep score difference at 15 minutes

All three features are quantitative and were standardized using `StandardScaler` to ensure they are on the same scale before being passed to the logistic regression model. No categorical encoding was needed for this baseline.

**Performance:**

| Split | Accuracy | F1-Score |
| ----- | -------- | -------- |
| Train | 0.7375   | 0.7358   |
| Test  | 0.7318   | 0.7342   |

The baseline model achieves **73.18% test accuracy**, which is significantly better than random guessing (50%). The train and test scores are very close, indicating the model generalizes well and is not overfitting. However, 73% accuracy leaves room for improvement — we believe adding more features capturing early objective control and kill differences could improve performance.

## Final Model

For the final model we switched to a **Random Forest Classifier** and engineered four additional features on top of the baseline features.

**New features added:**

- `golddiffat10` (quantitative) — gold difference at 10 minutes captures an earlier indication of game state before teams have started grouping for objectives. A team that is already ahead at 10 minutes is likely snowballing (progressively stacking) their advantage.
- `killsat15` (quantitative) — kills at 15 minutes capture early aggression and fighting. Teams that are winning fights early or have more kills tend to translate that into gold and objective advantages.
- `firstblood` (nominal) — whether the team secured first blood (binary 0/1). First blood gives an additional gold and experience bonus, possibly signifying which team has an early momentum.
- `firstdragon` (nominal) — whether the team secured first dragon (binary 0/1). Dragons give stacking buffs that stack over the course of the game, making early dragon control strategically significant.

**Hyperparameter Tuning:**

We used `GridSearchCV` with 5-fold cross validation to tune `n_estimators` and `max_depth`. The best parameters were:

- `n_estimators`: 50
- `max_depth`: 5

A shallow forest with fewer trees performed best, suggesting the relationship between early game stats and winning is relatively straightforward and doesn't require a highly complex model.

**Performance:**

| Split | Accuracy | F1-Score |
| ----- | -------- | -------- |
| Train | 0.7490   | 0.7458   |
| Test  | 0.7394   | 0.7394   |

The final model improves over the baseline by **+0.76% test accuracy** and **+0.005 F1-score**. The model at this point still generalizes well with no signs of overfitting.

<iframe
  src="assets/confusion_matrix.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Fairness Analysis

**Groups:**

- **Group X:** Blue side teams
- **Group Y:** Red side teams

**Evaluation Metric:** Accuracy

**Null Hypothesis:** Our model is fair. Its accuracy for blue side and red side teams are roughly the same and any differences are due to random chance.

**Alternative Hypothesis:** Our model is unfair. Its accuracy differs between blue side and red side teams.

**Test Statistic:** Difference in accuracy between blue side and red side teams (blue - red).

**Significance Level:** alpha = 0.05

**Results:**

- Blue side accuracy: **0.7335**
- Red side accuracy: **0.7450**
- Observed difference: **-0.0115**
- P-value: **0.426**

**Conclusion:** We fail to reject the null hypothesis. The observed accuracy difference of 1.15% between blue and red side teams is not statistically significant as differences this large occur roughly 42% of the time by chance alone. We conclude that our model performs fairly across blue and red side teams.

<iframe
  src="assets/fairness_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
