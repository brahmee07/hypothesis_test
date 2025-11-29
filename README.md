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

## **4. Merging the Data**

To merge the two datasets:

### **Name Cleaning**
- Lowercased names  
- Removed accents using `unidecode`  
- Created clean comparison keys  

### **Three-Stage Matching**
1. **Exact match**  
2. **Substring match**  
3. **Fuzzy match** (using `fuzzywuzzy` token sort ratio)

Players with no reliable match were excluded.

### **Final Working Dataset Columns**
- `name`  
- `preferred_foot`  
- `situation`  
- `shotType` (LeftFoot, RightFoot only)  
- `shotResult`  
- `xGoal`

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

## **6. Methods**

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
- **average efficiency** (our main metric)

---

## **7. Test Statistic (Plain Language)**

We compare:

> **Difference in average finishing efficiency  
(left-footed mean − right-footed mean)**

This is the statistic we shuffle in the permutation test.

---

## **8. Permutation Test**

### **Observed Difference**
Using ~750 matched players:

