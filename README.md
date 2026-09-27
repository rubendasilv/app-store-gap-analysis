# Profitable App Profiles: Finding a Gap in the Market

Which kind of free app should a developer build to attract the most users on **both** Google Play and the Apple App Store?

This project analyses free English-language apps from both stores (8,669 on Google Play and 3,117 on the App Store). It builds an automated outlier-handling pipeline so the recommendation isn't driven by a handful of giant apps.

**Recommendation: a free Weather app.** Once dominant apps and outliers are dealt with, Weather has the best combined ranking across the two stores: 1st of 11 trusted genres on the App Store and 8th of 41 on Google Play. The evidence is moderate rather than conclusive. The App Store result rests on only about 27 apps, and Weather is borderline on both concentration measures (a ratio of 33.9% against a 35% threshold, and an HHI of 1,807 against a 1,800 line), so it is the best candidate to test first, not a sure bet.

---

## The question

The app will be free and earn money from in-app ads, so revenue depends on the number of users. The goal is to find a genre with:

- **high demand**: lots of installs or ratings per app
- **low competition**: a small share of apps in that genre
- **no single dominant app**: demand spread across many apps, not held by one

## Data

| Dataset | Source | Demand measure |
|---|---|---|
| Google Play (~10,800 apps) | [Kaggle: Google Play Store Apps](https://www.kaggle.com/datasets/lava18/google-play-store-apps) | `Installs` (banded, e.g. 1,000,000+) |
| App Store (~7,200 apps) | [Kaggle: Mobile App Store](https://www.kaggle.com/datasets/ramamet4/app-store-apple-data-set-10k-apps) | `rating_count_tot` (used in place of installs) |

## Approach

### 1. Data cleaning
- Removed a Google Play row with an invalid rating (above 5).
- Removed 1,181 duplicate Google Play entries, keeping the row with the most reviews for each app.
- Removed non-English apps with a custom character whitelist. It allows emoji and common symbols, so English names with emoji aren't wrongly dropped.
- Kept free apps only: 8,669 on Google Play and 3,117 on the App Store.

### 2. Baseline analysis
- Frequency tables of genres in each store.
- Average installs or ratings per genre.
- A first **opportunity score**: `average demand ÷ frequency %`.

### 3. Outlier-handling pipeline
The baseline score was topped by Navigation, Reference and Book, so I built a pipeline to test whether those results were real.

| Step | What it does |
|---|---|
| 1. Group | Groups every app's installs or ratings by genre |
| 2. Concentration ratio | Biggest app ÷ genre total, showing how much one app dominates a genre |
| 3. HHI | Sum of every app's squared % share: a second measure that counts all apps, not just the biggest |
| 4. Three-way split | Genres under **10 apps** are set aside as too small to judge, genres at or above **35%** are flagged (marked, not deleted), and trusted genres with an HHI of **1,800+** are labelled borderline |
| 5. IQR filter | Removes high outliers (above Q3 + 1.5 × IQR) from trusted genres only |
| 6. Averages | Cleaned averages for trusted genres, raw averages for flagged genres |
| 7. Opportunity score | `average demand ÷ frequency %`, recalculated on the cleaned data |
| 8. Ranked output | Numbered rankings per store, with trusted and flagged genres kept separate |
| 9. Sensitivity check | Re-runs the ranking with other thresholds and minimum sizes to show how much the settings matter |

A design decision worth noting: `statistics.quantiles(..., method='inclusive')` is used. The default `exclusive` method let a single huge value pull Q3 up high enough to survive its own filter.

The settings borrow from competition law. The European Commission's guidance says a company is unlikely to be dominant below a 40% market share, and EU case law presumes dominance above 50%, so a 35% line is slightly stricter than the Commission's 40% level. The HHI line of 1,800 comes from the US 2023 Merger Guidelines, which treat markets above it as highly concentrated. The 10-app minimum exists because a genre with *n* apps can't have a concentration ratio below 1 ÷ n, so very small genres would be flagged whatever their data says.

## Key findings

- **The original top 3 App Store genres don't survive.** In Reference (73.1%) and Book (45.3%), one app holds much of the demand, and Navigation has only 5 free apps, too few to judge. Their high scores don't show a real gap.
- **The pipeline finds outliers on its own.** Facebook and WhatsApp were removed by hand for the baseline score only. The pipeline starts from the full list and flags Social Networking (39.2%), Facebook's genre, without help.
- **Weather held up.** It was the strongest trusted genre before the pipeline (62,548) and is still 1st of 11 after it (14,514). It has the highest cleaned average of any trusted genre (12,572 ratings per app), and fewer than 1% of free apps are Weather apps.
- **Weather also performs on Google Play.** It ranks 8th of 41 trusted genres (6th among standalone genres), with a low concentration ratio (13.9%) and HHI (865).
- **Other App Store leaders fail on Google Play.** Business is 2nd on the App Store but last on Google Play, and Finance is 3rd vs 33rd.
- **Shopping is the runner-up.** It ranks 5th on the App Store and 14th on Google Play.
- **The App Store result depends on the settings.** In the sensitivity check, Weather stays 1st on the App Store in four of seven settings but drops out under a 30% threshold, a 30-app minimum or the strict HHI rule. On Google Play it ranks between 4th and 13th under every setting.
- **Some candidates collapsed or were flagged.** Music fell from 27,894 to 3,450 after outlier removal, and Food & Drink was flagged (35.1%).
- **Games are saturated.** Some game genres score well on Google Play, but Games ranks last on the App Store.

## Visualisations

The notebook includes six charts:

1. Distribution of installs and ratings (log scale): why outlier handling is needed
2. Concentration ratio by App Store genre, with the 35% threshold
3. Box plots of rating counts, showing the outliers removed by the IQR rule
4. Demand vs competition quadrant chart, with Weather in the high-demand, low-competition corner
5. Ranked opportunity scores for trusted App Store genres
6. Opportunity scores before and after outlier handling

## Limitations

- **Close to both thresholds:** Weather's App Store concentration ratio (33.9%) is just under the 35% cut-off and its HHI (1,807) is just over the 1,800 line.
- **Small sample:** only about 27 free English App Store apps are Weather apps.
- **Different demand measures:** Google Play uses installs and the App Store uses rating counts, so genres are compared across stores by **rank**, not raw score.
- **Banded installs:** Google Play installs are grouped (1,000+, 10,000+, ...), so averages are approximate.
- **Small genres:** Google's combined sub-genres (e.g. `Casual;Pretend Play`) contain few apps, which inflates their scores. The 10-app minimum sets the smallest aside, but some just above it still top the ranking. Grouping by `Category` would avoid them.
- **Judgement calls:** the 10-app minimum, the 35% threshold, the 1,800 HHI line and the 1.5 × IQR rule are conventions, not fixed rules. The sensitivity check shows how much they matter.

## Tools

- Python 3 (standard library: `csv`, `statistics`)
- matplotlib and NumPy for the charts
- Jupyter Notebook

The data cleaning and pipeline are written in plain Python, without pandas, as a way to practise the underlying logic.

## How to run

```bash
git clone https://github.com/rubendasilv/<repo-name>.git
cd <repo-name>
pip install matplotlib numpy jupyter
jupyter notebook Basics.ipynb
```

Download both CSVs from the Kaggle links above into the same folder as the notebook (`AppleStore.csv` and `googleplaystore.csv`), then run all cells in order.

## Author

**Ruben Da Silva**
[LinkedIn](https://www.linkedin.com/in/ruben-da-silva-/) · [GitHub](https://github.com/rubendasilv)

Built as part of the Dataquest guided project *Profitable App Profiles for the App Store and Google Play Markets*. I extended it with my own outlier-handling pipeline and visualisations.
