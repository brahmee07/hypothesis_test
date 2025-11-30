# **Footedness & Finishing Efficiency in Football**  
---
## **Research Question**

Do left-footed players have better finishing ability than right-footed players?

There has always been debate about whether left-footed players have a natural advantage in finishing compared to right-footed players. As someone who has played football for many years and followed the sport closely, I’ve often felt that left-footed players tend to be more clinical. This project aims to test that belief using real shot-level data rather than opinions or appearances to determine whether one group truly finishes better. The answer will help teams make better decisions when signing new players and planning their game tactics.

---
## **Hypothesis**

### **Null Hypothesis (H₀)**
There is **no difference** in finishing efficiency between left-footed and right-footed players.

### **Alternative Hypothesis (H₁)**
Left-footed players have **higher finishing efficiency** than right-footed players.

---

## Data Description

### Data Sources

This project uses two external datasets that were merged to create the final analysis-ready dataset.

#### 1. Football Database (2014–2020)
**Source:** https://www.kaggle.com/datasets/technika148/football-database  
Used for:
- Shot-level data (`shots.csv`)
- Variables such as `shotType`, `shotResult`, `xGoal`, `situation`
- Player IDs and match information

#### 2. Football Players Data (SoFIFA, 2023)
**Source:** https://www.kaggle.com/datasets/maso0dahmed/football-players-data  
Used for:
- `full_name`
- `preferred_foot`

This dataset provides preferred-foot information that the Football Database does not include.

---

## Unit of Analysis and Observations

- **Unit of Analysis:** Individual player (after aggregating shots)
- **Initial dataset size after merging:** 3,234 players (shape: 3234 × 7)
- **Final dataset size after filtering:** 2,073 players (shape: 2073 × 7)

The final 2,073 players are those who:
- Had a valid preferred foot  
- Took at least 10 qualifying shots  
- Had only open-play, footed (left or right) shots in the dataset  

---

## Data Filtering and Transformation

### 1. Merging and Cleaning
- Cleaned names (lowercased and removed accents using `unidecode`)
- Matched players via:
  - Exact match  
  - Substring match  
  - Fuzzy match (`fuzzywuzzy`)

**Result:**  
After merging and removing unmatched entries, the working dataset contained 3,234 players with the following columns:  
`name`, `preferred_foot`, `situation`, `shotType`, `shotResult`, `xGoal`, `playerID`.

---

### 2. Shot-Level Filtering
To ensure a fair comparison of finishing ability, the following filters were applied:

**Kept:**
- Footed shots (`LeftFoot`, `RightFoot`)
- Open-play shots (`situation == 'OpenPlay'`)

**Removed:**
- Headers and other body-part shots  
- Penalties, direct free kicks, and corners  
- Own goals  
- Shots that hit the post  

---

### 3. Minimum Sample Size Requirement
- Only players with **10 or more** qualifying shots were retained.
- This reduced the dataset from 3,234 players to 2,073 players.

---

### 4. Efficiency Metric and Aggregation

Shot-level efficiency was defined as:

efficiency = Goal(1 or 0) - xG


This was aggregated to the player level:

- total_shots  
- total_goals  
- total_xg  
- total_efficiency  
- avg_efficiency  

`avg_efficiency` is the final statistic used to compare left-footed vs right-footed players.


## **Methods**

### **Test Statistic**
To compare finishing ability between left-footed and right-footed players, I used:

$$\textbf{Test Statistic} = \text{Mean Efficiency}_{\text{Left}} - \text{Mean Efficiency}_{\text{Right}}$$


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

$$\textbf{Test Statistic} = \text{Mean Efficiency}_{\text{Left}} - \text{Mean Efficiency}_{\text{Right}}$$

**Observed difference:**  

0.0033

This means that left-footed players overperformed xG by about **0.33 percentage points more** than right-footed players.

---

### **Permutation Test**

To test whether this difference could occur by chance, I performed **10,000 permutations** under the null hypothesis.

**P-value:**  

0.0643

Interpretation:
- A p-value of ~0.06 indicates *weak evidence* against the null hypothesis.
- The difference is suggestive, but not conventionally statistically significant at the 0.05 level.

---

### **Bootstrap Confidence Interval**

Using **10,000 bootstrap resamples**, I estimated uncertainty around the difference in finishing efficiency.

**95% Bootstrap Confidence Interval:**  

$$\textbf{(-0.0009, 0.0076)}$$

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
  
 $$\textbf{(-0.0009, 0.0076)}$$
 
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

## **8. References**

### **Datasets**
- **Football Database (2014–2020)**  
  Kaggle: https://www.kaggle.com/datasets/technika148/football-database  
- **Football Players Data (SoFIFA)**  
  Kaggle: https://www.kaggle.com/datasets/maso0dahmed/football-players-data


