# India Youth Football Match Appearances (2025–26)

## Question

Why hasn't India qualified for the FIFA World Cup despite football's widespread popularity? More fundamentally, why does the country struggle to produce enough elite players?

There is no single answer. Explanations range from governance and infrastructure to coaching quality, talent identification and youth development. I decided to examine one specific claim: that young footballers in India do not play enough competitive matches to develop effectively.

While exploring this question, I came across the AIFF's _Vision 2047_ document, which sets a target of 35 league matches per youth player per season. That shifted the project from a broad exploration of youth development to an accountability story: **how close is Indian football to meeting its own benchmark?**

---

## Data Collection

- **Season analysed:** 2025–26
- **Objective:** Measure official match appearances per player across AIFF-recorded competitions during the season for club league youth players.
- **Data source:** Fixture data was collected from an undocumented AIFF Match Centre API endpoint:
  `https://www.the-aiff.com/api/fixtures/cms/{fixture_id}`

### Determining the fixture ID range

- The API requires a numeric fixture ID, so the first step was identifying the IDs corresponding to the 2025–26 season. Fixture IDs were found to increase roughly over time, although many returned no public records.
- More than **60 fixture IDs** were manually sampled to identify a range that captured all fixtures between **12 July 2025** and **31 May 2026**. To minimise the risk of missing matches, a broader search window of **1 June 2025 – 30 June 2026** was used.
- **Final fixture ID range scraped:** 49,000–72,500.

Consecutive blocks of 20 or more failed fixture IDs (covering approximately **55% of all HTTP 500 responses**) were manually reviewed and found to be consistently empty or outside the study period.

| Metric                |  Value |
| --------------------- | -----: |
| Fixture IDs attempted | 23,501 |
| Successful JSONs      | 21,027 |
| Failed (500)          |  2,474 |
| Success rate          |  89.5% |

---

## Data Preparation

### Player appearance definition

A player was counted as having made an appearance if they either started the match or entered as a substitute. Players who were named in the squad but remained unused substitutes were not counted. Starting players were identified using the `status` field, while substitute appearances were identified using substitution events.

### Competition classification

Each unique competition was classified into five categories:

- Competition level (National, State or District)
- State/Union Territory
- Age group (Youth or Senior)
- Competition format (Club League or Other)
- Gender

Classification was performed using a large language model based primarily on the competition name and a predefined set of decision rules. The model was instructed to classify conservatively, assigning **"Unclear"** whenever the competition name did not provide sufficient evidence. The resulting classifications were manually validated on a sample of competitions, achieving over **90% accuracy** for competition level, state, age group and competition format, and **70% accuracy** for gender.

The full prompt is available in `02_classifying_competitions.ipynb`.

Because _Vision 2047_ sets a target for **male youth club league football**, the analysis includes only male youth club leagues. Senior competitions, women's competitions, representative-team tournaments, futsal, beach soccer and competitions classified as **Other** (primarily knockout competitions) were excluded.

---

## Data Cleaning

- Of the **21,027** successfully scraped fixtures, **18,205** remained in the player-match dataset. A total of **2,822** fixtures were excluded because both teams' player lists were missing. An additional **699** fixtures had one team's player list missing and were retained.
- Player appearances were tracked using the **AIFF ID**, as the Player ID was not unique. Formatting inconsistencies in player names (for example, extra whitespace and bracketed text) were standardised using regex-based cleaning so that each AIFF ID mapped to a single player name without dropping any records.
- A final data quality check identified only **two players** who appeared for multiple teams within the same competition. These cases were manually reviewed and resolved; **one player** was excluded due to irreconcilable data quality issues.
- Six player-match records contained an internal inconsistency, with the same player recorded as both a starter and an incoming substitute. These records were retained because both indicators independently confirmed an appearance, and the appearance logic counted each player only once per match.

---

## Final Analysis Dataset

After filtering to the competitions relevant to the AIFF's Vision 2047 benchmark, the final analysis dataset consisted of:

- **51** male youth club league competitions
- **4,936** matches
- **19,718** unique players

---

## Repository Structure

```text
data_collection/
├── scraper.ipynb
├── scraper_log.csv
└── 00_import_jsons.ipynb

data_analysis/
├── 01_data_dictionary.ipynb
├── 02_classifying_competitions.ipynb
├── 03_build_player_match_dataset.ipynb
├── 04_merge_comp_metadata.ipynb
├── 05_cleaning.ipynb
├── 06_analysis.ipynb
├── analysis_dataset.csv
└── competitions_classified_AI_human.csv
```

### Notebook descriptions

**01_data_dictionary.ipynb**  
Explores the AIFF JSON structure, documents variables and identifies fields relevant for analysis.

**02_classifying_competitions.ipynb**  
Develops, tests and applies the LLM-based competition classification.

**03_build_player_match_dataset.ipynb**  
Builds the player-match dataset from all successfully scraped fixtures.

**04_merge_comp_metadata.ipynb**  
Merges the player-match dataset with the competition metadata.

**05_cleaning.ipynb**  
Performs data cleaning, validation and quality checks.

**06_analysis.ipynb**  
Contains the exploratory data analysis and final analysis used in the story.

---

## Limitations

The dataset represents all publicly accessible AIFF Match Centre fixtures found within the searched fixture ID range. It should **not** be interpreted as evidence that every competition organised by every State Member Association, or every officially played fixture, exists in the public API.

---

## Skills Used

- Reverse-engineering an undocumented API
- Web scraping
- Exploratory data analysis (EDA)
- Data cleaning (regex & pandas)
- Entity resolution
- LLM-assisted classification
- Data validation and quality assurance
- Accountability reporting
