<p align="center">
  <img src="screenshots/fifa25.jpg" alt="FIFA 25 Power BI Dashboard" width="800">
</p>

# FIFA 25 Player Analytics – Power BI Dashboard

This project is an end-to-end Power BI dashboard built on top of a FIFA 25 style player dataset.  
The goal is to explore **player quality, league strength, and positional profiles** using a clean data model, DAX measures, and interactive visuals.

---

## 🧩 Project Overview

**Main objectives:**

- Clean and model a raw player dataset in Power BI (using Power Query)
- Build KPIs for:
  - Total number of players
  - Average overall rating (OVR)
  - Number and rate of *elite* players (OVR ≥ 80)
- Compare:
  - Leagues by average OVR and elite-player density
  - Positions by average OVR (outfield players)
  - Nations by player counts
- Design a simple, readable dashboard with slicers and interactive analysis

---

## 🗂 Repository Structure

```
fifa25-powerbi-player-analytics/
│
├─ PowerBI/
│   └─ fifa25_player_dashboard.pbix      # Main Power BI report
│
├─ data/
│   └─ male_players.csv                  # Player dataset
│
├─ screenshots/
│   ├─ fifa25.jpg                        # Hero image used in README
│   ├─ dashboard_overview.png            # Full dashboard view
│   └─ league_insights.png               # League comparison visuals
│
└─ README.md
```

---

## 🧱 Data & Modeling

### Dataset

The dataset contains roughly:

* 16K players

* Dozens of attributes per player:

    * Overall rating: OVR

    * Pace, shooting, passing, dribbling, defending, physical: PAC, SHO, PAS, DRI, DEF, PHY

    * Detailed technical attributes (e.g. Finishing, Short Passing, Interceptions, etc.)

    * Goalkeeper attributes: GK Diving, GK Handling, GK Kicking, GK Positioning, GK Reflexes

    * Player profile: Age, Height, Weight, Nation, League, Team, Position, Preferred foot, Weak foot, Skill moves

Each row represents one player.

### Power Query – Data Cleaning

Key cleaning steps in Power Query:

* Removed index / technical columns:

    * Unnamed: 0.1, Unnamed: 0

* Converted height & weight from text to numeric (cm / kg):

    * Extracted numeric part before delimiter:

        * Height → Height_cm (Whole Number)

        * Weight → Weight_kg (Whole Number)

* Dropped non-analytical columns (for the first version of the model):

    * alternative positions

    * play style

    * url

* Created a PlayerType column:

    * PlayerType = "Goalkeeper" when Position = "GK"

    * PlayerType = "Outfield" otherwise

The final table can be viewed as a single fact table:

* Fact: player-level metrics (OVR, attributes, physical, age, GK stats)

* Dimensions embedded in the same table: Nation, League, Team, Position, Preferred foot, Weak foot, Skill moves

For this project, a single-table model is sufficient and keeps the dashboard simple.

---

## 🧮 DAX Measures

The following measures are used in the report.

1. Total number of player

  Player Count = COUNT('male_players'[Name])

* Counts non-blank player names

* Respects all filters (League, Nation, Position, etc.)

* Example:

    * All data: ~16K players

    * Premier League selected: 597 players

    * Serie A selected: 537 players

2. Average overall rating

  Avg OVR = AVERAGE('male_players'[OVR])

* Replaces the implicit “Average of OVR” with an explicit measure

* Example:

    * All leagues: 66.17

    * Premier League: 74.27

    * Serie A: 73.27

3. Elite players (OVR ≥ 80)

  High OVR Players = 
  CALCULATE(
      COUNTROWS('male_players'),
      'male_players'[OVR] >= 80
  )

* Counts players with OVR >= 80 under the current filter context

* Example:

    * All leagues: 462 elite players

    * Premier League only: 125

    * Serie A only: 70

4. Elite player rate

  High OVR Rate = 
  DIVIDE(
      [High OVR Players],
      [Player Count]
  )

* Percentage of players with OVR ≥ 80 within the current filter context

* Formatted as Percentage with 1 decimal place

* Examples:

    * All leagues: 2.88% of players are 80+

    * Premier League: 20.94%

    * Serie A: 13.04%

This distinction between count and rate turns raw numbers into more meaningful KPIs.

---

## 📊 Main Visuals

The report page is structured in three horizontal bands:

1. KPI Strip (Top)

  Three main cards:

* Total Players → Player Count

* Average Rating → Avg OVR

* 80+ Players Rate → High OVR Rate

  Optional extra KPI:

* 80+ Players → High OVR Players

  These give an instant overview of dataset size and quality.

2. League-level Insights (Middle)

  Two side-by-side column charts:

- Average OVR by League

    * Axis: League

    * Values: Avg OVR

    * Sorted by Avg OVR (descending)

    * Filtered to leagues with a minimum number of players (e.g. Player Count ≥ 200)

    * This removes small leagues with only 1–2 top teams, which would otherwise distort averages.

- 80+ Players Rate by League (≥200 Players)

    * Axis: League

    * Values: High OVR Rate

    * Same Player Count ≥ 200 filter applied

    * Data labels show the share of elite players per league (%)

  Observed pattern (example):

* Top 5 leagues by average OVR and by elite-player rate:

    * Premier League

    * La Liga

    * Serie A

    * Bundesliga

    * Saudi League (high elite density due to a few star-heavy squads)

  Premier League clearly stands out:

* Higher average rating

* Significantly higher proportion of 80+ players (~21%)

3. Position & Nation Insights (Bottom)

- Average OVR by Position

    * Axis: Position

    * Values: Avg OVR

    * Example top positions (values are approximate and close to each other):

        * CDM ~ 66.98

        * RM ~ 66.74

        * LM ~ 66.63

        * CB ~ 66.60

        * LW ~ 66.57

Interpretation:

* Defensive midfielders (CDM) and wide midfielders (RM/LM) slightly lead in average OVR.

* Differences are small, reflecting a relatively balanced dataset across positions.

- Players by Nation

    * Axis: Nation

    * Values: Player Count (or Count of Nation)

    * Sorted by player count (descending)

    Example top nations by player count:

    * England (~1491 players)

    * Germany (~1105 players)

    * Spain (~893 players)

    The dataset is clearly England-heavy, which also influences league and positional distributions.

---

## 🎛 Interactivity – League Slicer

A League slicer is placed on the left side of the report.

* Selecting a league instantly updates:

    * KPI cards (Total Players, Avg OVR, 80+ Players, 80+ Rate)

    * League charts (filtered to the selected league or subset)

    * Position chart (showing position profiles within that league)

Examples:

* Premier League selected

    * Player Count: 597

    * Avg OVR: 74.27

    * High OVR Players: 125

    * High OVR Rate: 20.94%

    * Top positions include ST and CDM around 75 average OVR.

Serie A selected

    * Player Count: 537

    * Avg OVR: 73.27

    * High OVR Players: 70

    * High OVR Rate: 13.04%

This makes the dashboard a simple but effective scouting-style comparison tool for leagues and positions.

---

## 🧠 Key Insights

From this version of the dashboard, some sample insights:

1. Global dataset quality

    * ~16K players with an overall average rating of 66.17

    * Only 2.88% of all players are 80+, which means elite players are rare.

2. League comparison

    * The Premier League leads both in:

        * Average OVR

        * Elite player density (over 20% of players are 80+)

    * La Liga, Serie A, and Bundesliga follow as expected for top European competitions.

    * Saudi League appears among the top leagues by elite-player rate, reflecting a small league with a high concentration of star players.

3. Positional profile

    * Average OVR by position (outfield) is relatively balanced.

    * Defensive midfielders (CDM) and wide midfielders/wings (RM/LM/LW) appear slightly stronger on average.

    * This can support decisions like:

        * Identifying strong positions in a specific league

        * Comparing positional strength between leagues.

4. Nation distribution

    * The dataset is heavily skewed toward England, followed by Germany and Spain.

    * This affects how many Premier League players and English players appear in league and position analyses.

---

## 🛠 Tools & Tech

* Power BI Desktop

    * Data modeling

    * DAX measures

    * Visual design

* Power Query

    * Data type conversion

    * Column cleanup (height/weight parsing, removing unnecessary columns)

    * Derived fields (PlayerType)

* DAX

    * Row counts and averages

    * Conditional measures (OVR ≥ 80)

    * Percentage KPIs

---

## 🤝 Contact

Created by Tacdin Özmen

If you want to discuss this project, data analytics, or Power BI:

    * GitHub: https://github.com/tcdn7

    * LinkedIn: https://www.linkedin.com/in/tacdinözmen/
