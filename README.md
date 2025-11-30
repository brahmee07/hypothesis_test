# **Footedness & Finishing Efficiency in Football**  
---
## **Research Question**
Do left-footed players have better finishing ability than right-footed players?

There has always been debate about whether left-footed players have a natural advantage in finishing compared to right-footed players. As someone who has played football for many years and followed the sport closely, I’ve often felt that left-footed players tend to be more clinical. This project aims to test that belief using real shot-level data, rather than opinions or appearances, to see whether one group truly finishes better.

---

## **Hypothesis**

### **Null Hypothesis (H₀)**
There is **no difference** in finishing efficiency between left-footed and right-footed players.

### **Alternative Hypothesis (H₁)**
Left-footed players have **higher finishing efficiency** than right-footed players.

---

## **Data Description**

### **Datasets Used**

This project uses two separate datasets, combined to form one analytical dataset:

---

### **1. Football Database (2014–2020)**
**Source:** https://www.kaggle.com/datasets/technika148/football-database  

This dataset contains detailed event-level football data from the **Top 5 European leagues** between **2014 and 2020**.  
It includes matches, players, teams, and—most importantly for this project—**shot-level data with xG values**.

- **Unit of analysis:** Each row in the *shots* table represents **one individual shot** taken in a match.
- **Key variables used:**  
  - `shooterID`  
  - `situation`  
  - `shotType`  
  - `shotResult`  
  - `xGoal`  
  - `gameID`  
  - `playerID` (separate table)

---

### **2. Football Players Data (SoFIFA, 2023)**  
**Source:** https://www.kaggle.com/datasets/maso0dahmed/football-players-data  

This dataset contains **17,000+ FIFA player profiles**, including attributes like full name, nationality, club, and—crucially—**preferred foot**.

- **Unit of analysis:** Each row is **one player** with biographical and skill attributes.
- **Key variable used:**  
  - `full_name`  
  - `preferred_foot`

This dataset does **not** cover all players from 2014–2020, so some players from the Football Database could not be matched.

---

## **Merging the Two Datasets**

The Football Database does not include preferred foot, so I performed the following steps:

### **Name Cleaning**
- Lowercased all names  
- Removed accents using `unidecode`  
- Removed punctuation  
- Created “clean” name columns

### **Matching Logic**
Used a three-stage matching pipeline:
1. **Exact match**  
2. **Substring match**  
3. **Fuzzy match** (fuzzywuzzy, token_sort_ratio)

Players not matched in any step (~3,200 players) were dropped because their preferred foot was unavailable.

---

## **Final Shot-Level Dataset**

After merging and cleaning:

### **Unit of analysis:**  
Each row represents **one footed shot** by a player, with preferred foot attached.

### **Variables:**
| Variable | Description |
|---------|-------------|
| `name` | Player name |
| `preferred_foot` | Left or Right |
| `situation` | Shot situation (OpenPlay, SetPiece, DirectFreekick, etc.) |
| `shotType` | LeftFoot / RightFoot (others removed) |
| `shotResult` | Goal, SavedShot, BlockedShot, MissedShots |
| `xGoal` | Expected goal value of the shot |

---

## **Filtering Steps**

To ensure the analysis focuses on meaningful, comparable finishing behavior, I applied several filtering steps:

### **1. Footed Shots Only**
Kept only shots taken with a clear foot:
- `LeftFoot`
- `RightFoot`

Removed:
- `Head`
- `OtherBodyPart`

This isolates **true finishing ability with the preferred or non-preferred foot**.

---

### **2. Open-Play Shots Only**
Kept only:
- `situation == 'OpenPlay'`

Reason:  
Set pieces, penalties, and direct free kicks involve different mechanics and do not reflect natural finishing ability in open play.

---

### **3. Remove Irrelevant Shot Outcomes**
Excluded:
- `OwnGoal`
- `ShotOnPost`

Reason:  
These do not represent normal finishing outcomes for a player.

---

### **4. Keep Only Players With Sufficient Sample Size**
After computing total shots per player, I kept **only players who took at least 10 shots** across the 2014–2020 period.

Reason:
- Players with very few shots have noisy efficiency values.
- A minimum of *n = 10* ensures more stable estimates of finishing behavior.

---

### **5. Efficiency Metric**
For each shot:

\[
\text{efficiency} = \text{Goal (1/0)} - \text{xG}
\]

Then aggregated at the player level to compute:
- total shots  
- total goals  
- total xG  
- total efficiency  
- **average efficiency**


---

## **Player-Level Dataset (Aggregated)**

To analyze finishing ability, I aggregated the shot-level data:

```python
player_efficiency = df.groupby(['name', 'preferred_foot']).agg(
    total_shots=('goal', 'count'),
    total_goals=('goal', 'sum'),
    total_xg=('xGoal', 'sum'),
    total_efficiency=('efficiency', 'sum'),
    avg_efficiency=('efficiency', 'mean')
).reset_index()
```
## **Methods**

### **Test Statistic**
To compare finishing ability between left-footed and right-footed players, I used:

\[
\textbf{Test Statistic} = \bar{E}_{\text{Left}} - \bar{E}_{\text{Right}}
\]

where:

\[
E = \text{average finishing efficiency} = \text{mean}( \text{Goal} - \text{xG} )
\]

This measures whether left-footed players exceed their expected goals by more than right-footed players.

---

### **Permutation Test**

To test the null hypothesis (*no difference between groups*), I simulated datasets under the assumption that footedness has no effect.

**Steps:**

1. Combine all players’ efficiency values into one array.
2. Randomly shuffle (permute) the values.
3. Split the shuffled data into two groups of the same sizes as:
   - number of left-footed players
   - number of right-footed players
4. Compute the difference in means for each shuffle.
5. Repeat **10,000 times** to build the null distribution.
6. The p-value is the proportion of simulated differences that are greater than or equal to the observed difference.

This tests whether the observed difference could arise by chance.

---

### **Bootstrap Confidence Intervals**

To quantify uncertainty, I bootstrapped the **difference in average efficiency**:

1. Resample left-footed players *with replacement*.
2. Resample right-footed players *with replacement*.
3. Compute the mean difference for each resample.
4. Repeat **10,000 times**.
5. Construct the **95% confidence interval** from the 2.5th and 97.5th percentiles of the bootstrap distribution.
---

## **Results**

### **Key Summary Statistics**

After filtering the data to include only:
- open-play shots,
- footed shots (LeftFoot / RightFoot),
- valid outcomes (excluding own goals and shots off the post),
- players with ≥ 10 total shots,

the final dataset included **3,234 players** with aggregated finishing metrics.

Mean finishing efficiency:

| Footedness | Mean Efficiency |
|-----------|-----------------|
| **Left-footed** | -0.0008 |
| **Right-footed** | -0.0041 |

Left-footed players showed slightly higher efficiency on average.

---

### **Observed Test Statistic**

I compared groups using:

\[
\text{Difference} = \bar{E}_{\text{Left}} - \bar{E}_{\text{Right}}
\]

**Observed difference:**  
\[
\textbf{0.0033}
\]

This means that left-footed players overperformed xG by about **0.33 percentage points more** than right-footed players.

---

### **Permutation Test**

To test whether this difference could occur by chance, I performed **10,000 permutations** under the null hypothesis.

**P-value:**  
\[
\textbf{0.0643}
\]

Interpretation:
- A p-value of ~0.06 indicates *weak evidence* against the null hypothesis.
- The difference is suggestive, but not conventionally statistically significant at the 0.05 level.

---

### **Bootstrap Confidence Interval**

Using **10,000 bootstrap resamples**, I estimated uncertainty around the difference in finishing efficiency.

**95% Bootstrap Confidence Interval:**  
\[
\textbf{(-0.0009,\ 0.0076)}
\]

Interpretation:
- The interval is wide and crosses zero.
- This means the true difference **could be slightly negative or moderately positive**.
- The data is compatible with left-footed players having a small advantage—but not definitively.

---

### **Visualizations**
(Two plots were generated using Matplotlib)

1. **Permutation Distribution Plot**
   - Shows where the observed difference lies relative to the null distribution.
   - The observed statistic is near the right tail but not extreme.

2. **Bootstrap Distribution Plot**
   - Shows the sampling variability of the estimated difference.
   - Helps visualize uncertainty around the estimate.

---

## **Uncertainty Estimation**

To quantify uncertainty in my results, I used **resampling-based methods** from class: permutation tests and bootstrapping.

### **Permutation Test**  
- **Number of permutations:** 10,000  
- The permutation distribution was centered around **0**, as expected under the null hypothesis that footedness does not matter.
- The observed difference (0.0033) was located slightly in the right tail of this distribution, but not far enough to be considered rare.
- This reinforces that the observed difference is small and only marginally surprising under the null model.

### **Bootstrap Confidence Interval**
- **Number of bootstrap samples:** 10,000  
- The bootstrap distribution of the mean difference between left-footed and right-footed players was roughly symmetrical but showed noticeable spread.
- The **95% bootstrap confidence interval** was:
  
  \[
  (-0.0009,\ 0.0076)
  \]

### **Interpretation**
- Because the interval includes **0**, we cannot confidently conclude that left-footed players are strictly better finishers.
- However, the interval leans slightly positive, meaning **the data is consistent with a small left-foot advantage**, even if the evidence is not statistically strong.
- Importantly, bootstrapping does not rely on the Central Limit Theorem—especially useful here because:
  - efficiency values are not normally distributed,
  - Some players have very few shots,
  - Efficiency includes extreme outliers (big xG overperformance or underperformance).

Bootstrapping gives a robust sense of uncertainty when theoretical approximations fail.

---
##  **Limitations**

Despite careful processing, this project has several important limitations:

### **Data Limitations**
- The preferred-foot dataset (SoFIFA, 2023) does **not perfectly overlap** with the Football Database (2014–2020).
- ~3,200 players were unmatched and removed from the analysis.
- Some players’ preferred foot may have changed or may be inaccurately labeled.
---

---

## **8. References**

### **Datasets**
- **Football Database (2014–2020)**  
  Kaggle: https://www.kaggle.com/datasets/technika148/football-database  
- **Football Players Data (SoFIFA)**  
  Kaggle: https://www.kaggle.com/datasets/maso0dahmed/football-players-data


