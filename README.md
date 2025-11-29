### Footedness & Finishing Efficiency in Football (2014–2020)

Research Question

Do left-footed football players finish better than right-footed players?

As a football fanatic, I’ve always heard people say that “left-footed players have naturally better finishing ability.”
This project uses six seasons of real shot-level data from Europe’s top leagues to test whether that belief is actually true.

Hypothesis
Null Hypothesis (H₀):

There is no difference in finishing efficiency between left-footed and right-footed players.

Alternative Hypothesis (H₁):

Left-footed players have higher finishing efficiency than right-footed players.

Finishing efficiency is defined as:

efficiency
=
Goals Scored
−
Expected Goals (xG)
efficiency=Goals Scored−Expected Goals (xG)

Data Description
Datasets Used
1. Football Database (Kaggle — 2014 to 2020)

A relational football dataset covering the Top 5 European leagues. Includes match events, shot-level xG, shot type, and player IDs.
Unit of analysis: individual shots.

2. Football Players Data (SoFIFA, 2023)

A dataset of 17,000+ players scraped from SoFIFA.com.
Includes preferred foot, weak-foot rating, nationality, and full player names.

Why Two Datasets?

The Football Database has shot-level data, but no preferred foot.

The FIFA dataset has preferred foot, but different names, and not all players overlap.

Merging the Data

To link the two datasets:

Cleaned and normalized names (lowercasing, removing accents with unidecode).

Used exact match → substring match → fuzzy matching (fuzzywuzzy) to match players.

Some players were unmatched (expected, since SoFIFA is from 2023 while shot data is 2014–2020).

Final Working Dataset

After merging, I created a clean dataset containing:

name

preferred_foot

situation

shotType (RightFoot / LeftFoot only)

shotResult

xGoal


Methods
Efficiency Metric

For each shot:

efficiency
=
Goal (1/0)
−
xG
efficiency=Goal (1/0)−xG
Player-Level Aggregation

For each player:

total shots

total goals

total xG

total efficiency

average efficiency

Then restricted the dataset to:

Open play shots only

Footed shots only (LeftFoot, RightFoot)

No own goals or shots hitting the post

Players with at least 10 total shots (ensures reliability)

Methods
Efficiency Metric

For each shot:

efficiency
=
Goal (1/0)
−
xG
efficiency=Goal (1/0)−xG
Player-Level Aggregation

For each player:

total shots

total goals

total xG

total efficiency

average efficiency

Then restricted the dataset to:

Open play shots only

Footed shots only (LeftFoot, RightFoot)

No own goals or shots hitting the post

Players with at least 10 total shots (ensures reliability)
