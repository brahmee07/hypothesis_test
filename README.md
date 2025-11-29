# **Footedness & Finishing Efficiency in Football (2014–2020)**  
### *Do left-footed players really finish better?*

---

## **1. Research Question**
As a football fanatic, I’ve always heard people say:

> **“Left-footed players have naturally better finishing ability.”**

This project uses six seasons of shot-level data from Europe’s top leagues (2014–2020) to test whether left-footed players actually finish better than right-footed players once we account for shot quality (xG).

---

## **2. Hypothesis**

### **Null Hypothesis (H₀)**  
There is **no difference** in finishing efficiency between left-footed and right-footed players.

### **Alternative Hypothesis (H₁)**  
**Left-footed players have higher finishing efficiency** than right-footed players.

Finishing efficiency is defined as:

\[
\text{efficiency} = \text{Goal (1/0)} - \text{xG}
\]

---

## **3. Data Description**

### **Datasets Used**

#### **1. Football Database (Kaggle, 2014–2020)**
A relational dataset covering the top 5 European leagues.  
Includes:
- Shot-level data  
- Expected goals (xG)  
- Shot type (RightFoot, LeftFoot, Head, etc.)  
- Shot situation (OpenPlay, SetPiece, Corner, etc.)  
- Player IDs  
- ~1800 matches per season  

**Unit of analysis:** individual shots.

---

#### **2. Football Players Data (SoFIFA, 2023)**
17,000+ players scraped from SoFIFA.com.  
Includes:
- Full names  
- Preferred foot  
- Weak-foot rating  
- Club/nationality  
- Positions & ratings  

**Reason for this dataset:**  
The Football Database *does not* include preferred foot, so this dataset fills that gap.

**Limitation:**  
Some older players (2014–2020) do not appear in FIFA 2023 → unmatched players removed.

---

## **🔧 Data Preparation Note**

All data cleaning, merging, and player–footedness matching were completed in a
separate preprocessing notebook, which is **not included in this repository**
to keep the project clean and focused on the statistical analysis.

The preprocessing steps included:
- Loading the Football Database (2014–2020) shot-level data  
- Loading the FIFA Player dataset (preferred foot)
- Cleaning and normalizing player names (lowercasing, removing accents)
- Applying exact match → substring match → fuzzy string matching
- Removing unmatched or ambiguous players
- Selecting only relevant shot information (Open Play, LeftFoot/RightFoot)
- Constructing the final analysis dataset: `player_shot.csv`

The final cleaned dataset is provided directly in this repository so that the
results can be fully reproduced **without requiring the preprocessing code**.

---

## **4. Methods**

### **Shot-Level Efficiency**
\[
\text{efficiency} = \text{Goal} - \text{xG}
\]

- Positive → finished better than expected  
- Negative → underperformed xG  

### **Player-Level Aggregation**
For each player:
- total shots  
- total goals  
- total xG  
- total efficiency  
- average efficiency  

---

## **5. Data Filtering**

To isolate true finishing ability:

### ✔ Open-play shots only  
(set pieces and penalties removed)

### ✔ Footed shots only  
- LeftFoot  
- RightFoot  
(headers and other body parts removed)

### ✔ Valid outcomes only  
Removed:
- OwnGoal  
- ShotOnPost  

### ✔ Minimum sample size  
Only players with **10+ shots** included.

---

## **6. Test Statistic (Plain Language)**

To compare finishing ability between groups, I used:

> **Difference in average finishing efficiency  
(left-footed mean − right-footed mean)**

If positive → left-footers finish better.  
If negative → right-footers finish better.  
If near zero → no real difference.

This is the statistic used in the permutation test.

---

## **7. Permutation Test**

### **Observed Difference**
Using ~750 matched players:


### **Permutation Procedure**
- Combine both groups  
- Shuffle player labels 10,000 times  
- Recompute the difference each time  
- p-value = proportion of shuffled diffs ≥ observed diff  

### **Permutation Result**
