# App Insights Unlocked: Solution Guide (Power BI)

Solution guide for the **App Insights Unlocked** data analytics case study, built on the Google Play Store dataset with **Power BI Desktop** (Power Query + DAX).

- **Dataset:** [Google Play Store Apps (Kaggle)](https://www.kaggle.com/datasets/lava18/google-play-store-apps), licensed CC BY 3.0
- **Files used:** `googleplaystore.csv` (apps) and `googleplaystore_user_reviews.csv` (review text and sentiment)
- **Main table in the model:** `googleplaystore_cleaned`
- **Dashboard:** one-page dashboard, `App_Insights_PowerBI_Dashboard.pbix`

---

## 1. Data Preparation Summary

| Step | What was done | Result |
|---|---|---|
| Corrupt row | Removed the row where columns were shifted (Category = `1.9`) | 1 row removed |
| Duplicates | Removed duplicate rows on `App` (first occurrence kept) | 10,841 rows to **9,659 unique apps** |
| Installs | Removed `,` and `+`, converted to Whole Number (`Installs_clean`) | Numeric |
| Price | Removed `$`, converted to Decimal Number | Numeric |
| Size | Removed `M`, converted to Decimal Number (`Size_MB`); "Varies with device" left blank | 1,227 apps have no size |
| Reviews | Converted to Whole Number | Numeric |
| Last Updated | Converted to Date | 21 May 2010 to 8 Aug 2018 |
| Rating | Missing values left blank (not filled) | 1,463 apps have no rating |

**Headline KPIs (match the dashboard cards):** Average Rating **4.17**, Total Apps **9.66K**, Categories **33**, Total Installs **75 bn**, Free Apps **92%**.

**Notes on method**
- `Installs` in the source is a range label (for example `1,000,000+`), so every install figure here is a lower bound, not an exact count.
- Averages ignore blank ratings and blank sizes automatically.
- All answers below were cross-checked with an independent Python (pandas) calculation on the same cleaning rules. Sizes given in KB in the source (314 apps) were converted to MB (divided by 1,024) for this check. If your `Size_MB` column skips them, size averages can differ by about 0.5 to 1 MB.

---

## 2. Dashboard Coverage Map

The single-page dashboard contains 5 KPI cards, 3 slicers (Category, Type, Content Rating), a scatter chart, a treemap, a category rating bar chart, an update-trend line chart and a Top 5 installed apps chart.

| # | Question | On dashboard? |
|---|---|---|
| B1 | Average rating | Yes (KPI card) |
| B2 | Unique categories | Yes (KPI card) |
| B3 | App size distribution | No, needs a size-bin chart |
| B4 | Free vs paid apps | Yes (Free Apps % card + Type slicer) |
| B5 | Most common content rating | Partly (slicer lists them, no counts) |
| B6 | Top 5 most installed apps | Yes (see the tie note in B6) |
| B7 | Apps rated 4.0 and above | No |
| B8 | Avg reviews, free vs paid | No |
| B9 | Avg size per category | No |
| B10 | Apps updated in 2018 | Yes (line chart, 2018 point) |
| M1 | Installs vs rating correlation | Partly (scatter uses installs as bubble size) |
| M2 | Categories by average rating | Yes (bar chart) |
| M3 | Price vs rating | No |
| M4 | Rating by content rating | No |
| M5 | Genres with most 1M+ apps | No (treemap is by category) |
| M6 | Update frequency | No |
| M7 | Size vs installs | No |
| M8 | Most reviewed apps and ratings | No |
| M9 | Content rating, free vs paid | Partly (use Type + Content Rating slicers) |
| M10 | Top 5 categories by installs | Yes (treemap) |
| A1 | Top 10 highest rated apps | No |
| A2 | Update trend over time | Yes (line chart) |
| A3 | Rating by install bins | No |
| A4 | Sentiment analysis | No |
| A5 | Genre vs rating | No |

Answers marked "No" can be reproduced with the method given under each question.

---

## 3. Basic-Level Questions

### B1. What is the average rating of apps in the dataset?
**Answer:** **4.17** (mean of 8,196 rated apps; median 4.3).

**Power BI:** `Avg Rating = AVERAGE(googleplaystore_cleaned[Rating])` shown in a Card visual.

**Business impact:** 4.17 is the benchmark. A new app should aim for 4.3 or higher (the median) to stand above the typical app.

### B2. How many unique categories of apps are there?
**Answer:** **33** categories.

**Power BI:** `Total Categories = DISTINCTCOUNT(googleplaystore_cleaned[Category])`

**Business impact:** The market is broad, so there is room to target niche categories instead of only crowded ones.

### B3. What is the distribution of app sizes?
**Answer:** Based on the 8,432 apps that report a size: mean **20.4 MB**, median **12 MB**, range 0.01 to 100 MB. Most apps are small.

| Size band | Apps | Share |
|---|---|---|
| 0 to 5 MB | 2,316 | 27.5% |
| 5 to 10 MB | 1,615 | 19.2% |
| 10 to 20 MB | 1,534 | 18.2% |
| 20 to 50 MB | 2,091 | 24.8% |
| 50 to 100 MB | 876 | 10.4% |

**Power BI:** Create a calculated column `Size Bin` with `SWITCH(TRUE(), ...)` on `Size_MB`, then a column chart of `Count of App` by `Size Bin`.

**Business impact:** About two thirds of apps are under 20 MB. Keeping an app small is a normal expectation; going above 50 MB puts it in the top 10% by size.

### B4. How many free vs paid apps are there?
**Answer:** **Free 8,902 (92.2%)**, **Paid 756 (7.8%)**. One app (Command & Conquer: Rivals) has no Type value.

**Power BI:** Donut chart with `Type` as Legend and `Count of App` as Values. The dashboard also has `Free Apps % = DIVIDE(CALCULATE(COUNTROWS(googleplaystore_cleaned), googleplaystore_cleaned[Type]="Free"), COUNTROWS(googleplaystore_cleaned))`.

**Business impact:** The market is overwhelmingly free. A paid app needs a clear reason to charge; otherwise a free model with ads or in-app purchases matches user expectations.

### B5. What is the most common content rating?
**Answer:** **Everyone**, with 7,903 apps (81.8%).

| Content rating | Apps | Share |
|---|---|---|
| Everyone | 7,903 | 81.8% |
| Teen | 1,036 | 10.7% |
| Mature 17+ | 393 | 4.1% |
| Everyone 10+ | 322 | 3.3% |
| Adults only 18+ | 3 | 0.03% |
| Unrated | 2 | 0.02% |

**Power BI:** Bar chart of `Count of App` by `Content Rating`.

**Business impact:** Designing for an "Everyone" rating reaches the widest audience and avoids age-restriction limits.

### B6. What are the top 5 most installed apps?
**Answer:** The top install bucket is **1,000,000,000+**, and **20 apps** are tied in it, so a strict "top 5" is not unique. The dashboard's Top 5 chart shows Facebook, Gmail, Google, Google Chrome and Google Drive, which is one valid set, but the bars are all the same length for this reason.

The 20 apps in the 1B+ bucket include WhatsApp, Instagram, YouTube, Subway Surfers, Google Photos, Maps, Hangouts, Skype, Messenger and others.

**A more meaningful ranking** uses reviews as a tie-breaker among the 1B+ apps: **Facebook (78.2M reviews), WhatsApp (69.1M), Instagram (66.6M), Messenger (56.6M), Subway Surfers (27.7M).**

**Power BI:** Bar chart, `App` vs `Sum of Installs_clean`, Top N filter = 5. To break ties, sort by `Sum of Reviews` instead, or add a filter `Installs_clean = 1000000000` and then use Top N by Reviews.

**Business impact:** The biggest apps come from large platform owners (Google, Meta), and one mobile game (Subway Surfers) is the only non-platform app to break into the top tier on reviews.

### B7. How many apps have a rating of 4.0 and above?
**Answer:** **6,286 apps**, which is **76.7% of the 8,196 rated apps** (65.1% of all 9,659 apps, since 1,463 have no rating).

**Power BI:** `High Rated Apps = CALCULATE(COUNTROWS(googleplaystore_cleaned), googleplaystore_cleaned[Rating] >= 4)`

**Business impact:** Most published apps are rated 4.0+, so 4.0 is the minimum to be competitive, not a differentiator.

### B8. What is the average number of reviews for free vs paid apps?
**Answer:** Free apps average **234,270 reviews**; paid apps average **8,725**. Free apps get about **27 times more reviews**.

**Power BI:** `Avg Reviews = AVERAGE(googleplaystore_cleaned[Reviews])` by `Type` in a column chart.

**Business impact:** Free apps get far more feedback and visibility. Paid apps should plan additional ways to collect reviews (in-app prompts, a free trial or lite version).

### B9. What is the average app size for each category?
**Answer:** Largest: **GAME 41.9 MB**, FAMILY 27.2 MB, TRAVEL_AND_LOCAL 24.2 MB, SPORTS 24.1 MB, ENTERTAINMENT 23.0 MB. Smallest: **TOOLS 8.8 MB**, LIBRARIES_AND_DEMO 10.6 MB, PERSONALIZATION 11.2 MB, COMMUNICATION 11.3 MB, PRODUCTIVITY 12.3 MB.

**Power BI:** `Avg Size = AVERAGE(googleplaystore_cleaned[Size_MB])` by `Category`, sorted descending.

**Business impact:** Games need the most storage and download budget; utility and communication apps are expected to be light.

### B10. How many apps were last updated in 2018?
**Answer:** **6,284 apps**, which is **65.1%** of the dataset (data runs to 8 Aug 2018).

**Power BI:** `Updated 2018 = COUNTROWS(FILTER(googleplaystore_cleaned, YEAR(googleplaystore_cleaned[Last Updated]) = 2018))`, or read the 2018 point on the line chart.

**Business impact:** Two thirds of apps were touched in the latest snapshot year, which shows regular maintenance is the norm.

---

## 4. Medium-Level Questions

### M1. What is the correlation between the number of installs and the app rating?
**Answer:** **r = 0.04** (Pearson, 8,196 rated apps). Spearman rank correlation is 0.03, and using log(installs) gives 0.09. Essentially **no meaningful linear relationship**.

**Power BI:** A Pearson correlation measure (DAX has no `CORREL`), for example:
```
Corr Installs Rating =
VAR T = FILTER(googleplaystore_cleaned, NOT ISBLANK(googleplaystore_cleaned[Rating]))
VAR N = COUNTROWS(T)
VAR Sx = SUMX(T, googleplaystore_cleaned[Installs_clean])
VAR Sy = SUMX(T, googleplaystore_cleaned[Rating])
VAR Sxy = SUMX(T, googleplaystore_cleaned[Installs_clean] * googleplaystore_cleaned[Rating])
VAR Sx2 = SUMX(T, googleplaystore_cleaned[Installs_clean] ^ 2)
VAR Sy2 = SUMX(T, googleplaystore_cleaned[Rating] ^ 2)
RETURN DIVIDE(N * Sxy - Sx * Sy, SQRT((N * Sx2 - Sx ^ 2) * (N * Sy2 - Sy ^ 2)))
```
The value should come out close to 0.04. A scatter chart of Installs vs Rating shows the same lack of pattern.

**Business impact:** More installs do not by themselves mean higher ratings, and a good rating does not guarantee downloads. Rating quality and user acquisition need separate strategies.

### M2. Which app categories have the highest average rating?
**Answer:** Top 7: **EVENTS 4.44, EDUCATION 4.36, ART_AND_DESIGN 4.36, BOOKS_AND_REFERENCE 4.35, PERSONALIZATION 4.33, PARENTING 4.30, BEAUTY 4.28**. Lowest: DATING 3.97, MAPS_AND_NAVIGATION 4.04, TOOLS 4.04.

**Power BI:** `Avg Rating` by `Category`, sorted descending (the dashboard shows the top 15).

**Business impact:** Niche, content-led categories satisfy users most; crowded utility categories score lowest. Higher ratings are easier to reach in less competitive categories.

### M3. How does the price of an app affect its average rating?
**Answer:** For the 604 paid apps that have a rating, correlation between price and rating is **-0.11**, a weak negative relationship. Rating drops as price rises:

| Price band | Apps | Avg rating |
|---|---|---|
| $0.01 to $1 | 106 | 4.30 |
| $1 to $2 | 101 | 4.29 |
| $2 to $5 | 282 | 4.26 |
| $5 to $10 | 59 | 4.23 |
| $10 to $50 | 40 | 4.22 |
| Above $50 | 16 | 3.91 |

Paid apps overall average 4.26 vs 4.17 for free apps. Median paid price is $2.99.

**Power BI:** Scatter (or column chart on a Price Band column) of `Price` vs `Avg Rating`, with a visual-level filter `Type = Paid`.

**Business impact:** Low price points ($0.99 to $4.99) carry the best satisfaction. Premium pricing above $50 comes with higher expectations and lower ratings, based on a small sample (16 apps).

### M4. What is the distribution of app ratings across different content ratings?
**Answer:**

| Content rating | Rated apps | Avg rating | Median |
|---|---|---|---|
| Everyone | 6,618 | 4.17 | 4.3 |
| Everyone 10+ | 305 | 4.23 | 4.3 |
| Teen | 912 | 4.23 | 4.3 |
| Mature 17+ | 357 | 4.12 | 4.2 |
| Adults only 18+ | 3 | 4.30 | 4.5 |
| Unrated | 1 | 4.10 | 4.1 |

Differences are small (4.12 to 4.23). The 18+ and Unrated groups are too small to interpret.

**Power BI:** Column chart of `Avg Rating` by `Content Rating`.

**Business impact:** Content rating has little effect on satisfaction, so the choice can follow the target audience rather than rating expectations.

### M5. Which genres have the most apps with over 1 million installs?
**Answer:** 1,978 apps have more than 1,000,000 installs. Top genres: **Tools 172, Action 128, Photography 123, Communication 99, Productivity 91, Entertainment 83, Arcade 81, Sports 79, Shopping 72, Social 67**. Including the "1,000,000+" bucket (3,395 apps), the order is similar (Tools 272, Action 180, Photography 173).

**Power BI:** Visual-level filter `Installs_clean > 1000000`, then `Count of App` by `Genres`, Top N = 10. Note that `Genres` contains combined values like `Art & Design;Pretend Play`, which are counted as separate values.

**Business impact:** Tools, action games and photography are the most proven routes to a 1M+ app.

### M6. How frequently do apps get updated? Calculate the average time between updates.
**Answer:** The dataset stores **one `Last Updated` date per app**, so the gap *between* consecutive updates cannot be computed. As a substitute, this measures how recently apps were updated, relative to the latest date in the data (8 Aug 2018): **average 281 days, median 96 days** since the last update.

| Time since last update | Apps | Share |
|---|---|---|
| Up to 30 days | 2,935 | 30.4% |
| 31 to 90 days | 1,829 | 18.9% |
| 91 to 180 days | 1,156 | 12.0% |
| 181 to 365 days | 1,321 | 13.7% |
| Over 1 year | 2,418 | 25.0% |

**Power BI:**
```
Days Since Update =
DATEDIFF(googleplaystore_cleaned[Last Updated],
  CALCULATE(MAX(googleplaystore_cleaned[Last Updated]), ALL(googleplaystore_cleaned)), DAY)
```
Add this as a calculated column, then use `AVERAGE` and `MEDIAN` of it.

**Business impact:** About half of apps are updated within 3 months. A monthly or quarterly release cycle keeps an app inside the most active half of the market.

### M7. What is the impact of app size on the number of installs?
**Answer:** Larger apps have higher installs: correlation is weakly positive (**r = 0.13**), but the pattern by band is clear.

| Size band | Apps | Avg installs | Median installs |
|---|---|---|---|
| 0 to 5 MB | 2,316 | 0.76M | 5,000 |
| 5 to 10 MB | 1,615 | 1.63M | 10,000 |
| 10 to 20 MB | 1,534 | 4.08M | 50,000 |
| 20 to 50 MB | 2,091 | 4.47M | 100,000 |
| 50 to 100 MB | 876 | 13.0M | 1,000,000 |

**Power BI:** Column chart of `Size Bin` vs `Average of Installs_clean` and `Median of Installs_clean`.

**Business impact:** Size itself does not drive downloads. Bigger apps are usually feature-rich or games, which are more popular, so this is a category effect. Do not shrink an app at the cost of features.

### M8. Which apps have the highest number of reviews, and what are their ratings?
**Answer:**

| App | Reviews | Rating |
|---|---|---|
| Facebook | 78.2M | 4.1 |
| WhatsApp Messenger | 69.1M | 4.4 |
| Instagram | 66.6M | 4.5 |
| Messenger | 56.6M | 4.0 |
| Clash of Clans | 44.9M | 4.6 |
| Clean Master | 42.9M | 4.7 |
| Subway Surfers | 27.7M | 4.5 |
| YouTube | 25.7M | 4.3 |
| Security Master | 24.9M | 4.7 |
| Clash Royale | 23.1M | 4.6 |

**Power BI:** Table visual with `App`, `Sum of Reviews`, `Average of Rating`, Top N = 10 by Reviews.

**Business impact:** Every top-reviewed app has a rating of 4.0 or above, and the mobile games and utility apps sit at 4.5 to 4.7. High engagement and high satisfaction go together at scale.

### M9. How does the content rating distribution differ between free and paid apps?
**Answer:**

| Content rating | Free | Free % | Paid | Paid % |
|---|---|---|---|---|
| Everyone | 7,248 | 81.4% | 655 | 86.6% |
| Teen | 984 | 11.1% | 52 | 6.9% |
| Mature 17+ | 375 | 4.2% | 18 | 2.4% |
| Everyone 10+ | 290 | 3.3% | 31 | 4.1% |
| Adults only 18+ | 3 | 0.03% | 0 | 0% |
| Unrated | 2 | 0.02% | 0 | 0% |

**Power BI:** 100% stacked column chart, `Type` on the axis and `Count of App` by `Content Rating` as the legend.

**Business impact:** Paid apps skew more toward "Everyone" (family-friendly) and less toward Teen and Mature content, so paid marketing should focus on general and family audiences.

### M10. What are the top 5 categories with the most installs?
**Answer:** **GAME 13.88 bn, COMMUNICATION 11.04 bn, TOOLS 8.00 bn, PRODUCTIVITY 5.79 bn, SOCIAL 5.49 bn.** Together they make up **58.8%** of the 75.1 bn total installs.

**Power BI:** `Sum of Installs_clean` by `Category`, Top N = 5 (the dashboard treemap shows the same ranking).

**Business impact:** Demand is concentrated. Games and communication alone account for about a third of all installs, which shows where user demand is highest and competition is strongest.

---

## 5. Advanced-Level Questions

### A1. What are the top 10 apps with the highest ratings, and how do their reviews and installs compare?
**Answer:** **271 apps have a perfect 5.0 rating, but 98.9% of them have 100 reviews or fewer** (median 4 reviews, median 100 installs), so they are not meaningful. Filtering to apps with at least 1,000 reviews (4,800 apps) gives a fair top 10, all at **4.9**:

| App | Reviews | Installs |
|---|---|---|
| JW Library | 922,752 | 10M+ |
| Six Pack in 30 Days - Abs Workout | 272,337 | 10M+ |
| Tickets + PDA 2018 Exam | 197,136 | 1M+ |
| Learn Japanese, Korean, Chinese Offline & Free | 133,136 | 1M+ |
| StrongLifts 5x5 Workout Gym Log & Personal Tra... | 66,791 | 1M+ |
| PixPanda - Color by Number Pixel Art Coloring ... | 55,723 | 1M+ |
| ipsy: Makeup, Beauty, and Tips | 49,790 | 1M+ |
| Hungry Hearts Diner: A Tale of Star-Crossed Souls | 46,253 | 500K+ |
| Lose Belly Fat in 30 Days - Flat Stomach | 38,098 | 5M+ |
| Solitaire: Decked Out Ad Free | 37,302 | 500K+ |

**Power BI:** Table visual with a filter `Reviews >= 1000`, sorted by `Average of Rating` then `Sum of Reviews`, Top N = 10.

**Business impact:** Truly top-rated apps combine strong ratings with tens of thousands of reviews. Health and fitness, education and hobby apps are common among them, which suggests focused, single-purpose apps earn the best reception.

### A2. Analyze the trend of app updates over time. Are there any noticeable patterns or seasonal trends?
**Answer:** Updates grow steadily every year and sharply in the last two:

| Year | Apps last updated |
|---|---|
| 2010 to 2012 | 42 |
| 2013 | 108 |
| 2014 | 203 |
| 2015 | 449 |
| 2016 | 779 |
| 2017 | 1,794 |
| 2018 (to 8 Aug) | 6,284 |

Within 2018, **July alone has 2,320 apps (24% of the dataset)**, up from 913 in June. In 2017, updates rise from about 100 per month early in the year to 200+ per month in Oct to Dec.

**Important limitation:** Each app stores only its *most recent* update, and the data was collected in August 2018. That makes recent months look much larger than older ones. The trend shows how current the dataset is, not a true seasonal pattern. A real seasonality study would need the full update history for each app.

**Power BI:** Line chart, `Last Updated` (Year, then Month) vs `Count of App`.

**Business impact:** Regular updating is the norm. Releasing at least every quarter keeps an app in line with the active majority.

### A3. How does the average rating of apps change with the number of installs? Create a binned analysis.
**Answer:**

| Installs bin | Apps | Avg rating |
|---|---|---|
| Under 10K | 1,761 | 4.16 |
| 10K to 100K | 1,444 | 4.04 |
| 100K to 1M | 1,598 | 4.13 |
| 1M to 10M | 2,022 | 4.22 |
| 10M+ | 1,371 | 4.32 |

The average rating dips at 10K to 100K, then rises with popularity to 4.32 at 10M+.

**Power BI:** Add a calculated column:
```
Installs Bin = SWITCH(TRUE(),
  googleplaystore_cleaned[Installs_clean] < 10000, "1) Under 10K",
  googleplaystore_cleaned[Installs_clean] < 100000, "2) 10K-100K",
  googleplaystore_cleaned[Installs_clean] < 1000000, "3) 100K-1M",
  googleplaystore_cleaned[Installs_clean] < 10000000, "4) 1M-10M",
  "5) 10M+")
```
Then a column chart of `Installs Bin` vs `Average of Rating`. The number prefixes keep the bins in order.

**Business impact:** Small apps with just 10K to 100K installs are rated lowest, which is the stage where quality problems most affect growth. Very popular apps are rated highest, which suggests larger teams, more polish, and survivorship (weak apps don't reach 10M+). This is an association, not proof that ratings cause installs.

### A4. Perform sentiment analysis on app reviews to determine the common themes in high and low-rated apps.
**Answer:** The review file has 64,295 rows across 1,074 apps. 37,432 rows (865 apps) have a sentiment label; the rest have no review text. Of the labelled reviews: **Positive 64.1%, Negative 22.1%, Neutral 13.8%** (average polarity 0.18). After joining to the main table, 816 apps have both reviews and a rating.

| App group | Apps | Positive | Negative | Neutral |
|---|---|---|---|---|
| High rated (4.5 and above) | 263 | 70.0% | 20.8% | 9.2% |
| Mid rated (4.0 to 4.4) | 452 | 62.7% | 22.1% | 15.2% |
| Low rated (below 4.0) | 101 | 54.0% | 27.6% | 18.4% |

At app level, average review polarity correlates with rating at **r = 0.26**, so positive review language tracks ratings but only moderately.

**Common themes** (word frequency in the review text, not a full topic model):
- **Positive reviews:** *love, great, good, easy, best, fun, nice, free*
- **Negative reviews:** *ads, update, fix, bad, money, annoying, hate, level*, which point to intrusive advertising, buggy updates and monetisation frustration (many reviews are about games)
- Words like *game* and *time* appear in both, so they are subject words and not sentiment signals.

**Power BI:** Load `googleplaystore_user_reviews.csv`, relate it to `googleplaystore_cleaned` on `App` (many-to-one, one `App` per row after de-duplication). Then use a column chart of `Count of Sentiment` by `Sentiment`, and a 100% stacked chart by a rating-group column.

**Business impact:** Users praise ease of use and fun, and complain about ads, bugs after updates and paywalls. Testing updates thoroughly, limiting ad frequency and keeping monetisation fair are the clearest ways to protect ratings.

**Limitation:** Review coverage is only 816 of 9,659 apps (about 8%), and it leans toward games, so these themes may not generalise to all categories.

### A5. What is the relationship between app genre and user ratings? Are certain genres consistently rated higher or lower?
**Answer:** Across genres with at least 20 rated apps (49 genres), the average rating varies only between **3.87 and 4.44**.

| Highest rated | Avg (median) | Rated apps |
|---|---|---|
| Events | 4.44 (4.5) | 45 |
| Puzzle | 4.37 (4.4) | 100 |
| Art & Design | 4.36 (4.4) | 55 |
| Books & Reference | 4.35 (4.5) | 169 |
| Word | 4.34 (4.3) | 22 |
| Parenting | 4.34 (4.5) | 40 |
| Personalization | 4.33 (4.4) | 298 |

| Lowest rated | Avg (median) | Rated apps |
|---|---|---|
| Educational | 3.87 (3.95) | 32 |
| Dating | 3.97 (4.1) | 134 |
| Maps & Navigation | 4.04 (4.2) | 118 |
| Tools | 4.04 (4.2) | 717 |
| Trivia | 4.04 (4.25) | 28 |
| Video Players & Editors | 4.05 (4.2) | 147 |
| Travel & Local | 4.07 (4.2) | 186 |
| Card | 4.07 (4.2) | 44 |

**Power BI:** Table or bar chart with `Genres`, `Average of Rating`, `Median of Rating` and `Count of App`, with a visual filter on count of apps of at least 20. Mean and median tell the same story, so the ranking is stable.

**Business impact:** Creative, hobby and content genres (Events, Puzzle, Art & Design, Books) are rated highest. Utility and location-heavy genres (Tools, Maps, Travel, Dating) are rated lowest, which likely reflects higher user expectations and technical issues. Note that "Educational" and "Education" are separate genre values in the source data.

---

## 6. Key Findings and Recommendations

1. **Set a quality bar of 4.3+.** The average rating is 4.17 and the median is 4.3; 76.7% of rated apps are 4.0 or higher, so 4.0 is only the entry point.
2. **Choose category with volume and satisfaction in mind.** Games, Communication and Tools drive the most installs (58.8% of installs are in the top 5 categories), while Events, Education, Art & Design and Books & Reference earn the highest ratings.
3. **Price low if charging.** Paid apps are only 7.8% of the market and get about 27x fewer reviews. Ratings hold up at $0.99 to $5 and fall for $50+.
4. **Popularity and rating are only loosely linked** (r = 0.04). Invest in both quality and acquisition rather than assuming one produces the other.
5. **Maintain the app.** 65% of apps were updated in 2018 and 25% have not been updated in over a year. Update at least quarterly, and test releases carefully because reviewers complain about updates and bugs.
6. **Manage ads and monetisation.** Ads, money and paywalls are recurring negative themes. Low-rated apps have 27.6% negative reviews vs 20.8% for high-rated apps.
7. **Do not sacrifice features to save size.** Larger apps have higher installs, but this mostly reflects category (games), not size itself.

---

## 7. Limitations

- Installs are range labels (lower bounds), so totals and averages of installs are approximate.
- 1,463 apps have no rating and 1,227 have no size; they are excluded from those averages.
- Only one `Last Updated` date is stored per app, and the data ends 8 Aug 2018, so update-frequency and seasonality results are indicative only.
- The reviews file covers about 8% of apps, so sentiment findings are directional.
- Correlations and patterns describe association, not cause. Small groups (for example the 16 apps priced above $50, and the 3 "Adults only 18+" apps) should not be over-interpreted.

---

## 8. Additional Resources

- Dataset: https://www.kaggle.com/datasets/lava18/google-play-store-apps
- Power BI documentation: https://learn.microsoft.com/en-us/power-bi/
- DAX function reference: https://dax.guide
- Power Query (M) reference: https://learn.microsoft.com/en-us/powerquery-m/
