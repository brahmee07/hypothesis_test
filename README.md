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

To ensure fairness and reduce noise:

- **Kept only footed shots:**  
  - `LeftFoot`, `RightFoot`
- **Kept only open-play shots:**  
  - `df = df[df['situation'] == 'OpenPlay']`
- **Removed outcomes not relevant to finishing:**  
  - Dropped `OwnGoal` and `ShotOnPost`
- **Computed shot-level efficiency:**  
  - `efficiency = goal - xGoal`

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


