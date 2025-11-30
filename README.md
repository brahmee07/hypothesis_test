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

### **Permutation Test (Randomization Test)**

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

### **Why the CLT Does Not Apply**
The Central Limit Theorem does **not** apply cleanly here for two reasons:

1. **Efficiency values are highly skewed.**  
   Most shots have \( \text{goal} = 0 \), and xG values are small and non-normal.

2. **Player averages vary wildly depending on shot count.**  
   Some players take exactly 10 shots; others take over 500.  
   This leads to **unequal variances** and **non-Gaussian sampling distributions**.

Because of this:

- The sampling distribution of average efficiency **is not normal**.
- Using a bootstrap CI is more appropriate and valid.

---

If you'd like, I can now integrate this into your full README or polish the Results / Uncertainty sections as well.



