# Zomato Restaurant Analysis

A four-page Power BI report on 9,542 restaurants across 15 countries, built on a star schema and designed to answer where online delivery adoption actually concentrates.

Built as a project for the Purdue University / Simplilearn Post Graduate Program in Data Analytics.

## The question

Restaurant platforms tend to assume online delivery adoption rises with price: nicer restaurants, more delivery. I wanted to see whether the data supported that, and what else the ratings and cost fields would show once the countries were separated out.

## Data model

The source data arrived as eight regional Excel files plus a country code lookup. I modeled it as a star schema rather than working from a flat table:

- `Fact_Restaurants` holds one row per restaurant with cost, rating, price range, and the delivery and booking flags.
- `Dim_Country` joins to the fact table on country code.
- `Dim_Currency` is a reference table I built to convert cost into a single currency. It joins on country code. See the currency conversion section below.
- `Dim_Cuisines` is a bridge table. Restaurants list their cuisines as a comma-separated string, so the query splits that into one row per restaurant-and-cuisine pair, 19,705 of them across 145 distinct cuisines. Restaurants average 2.07 cuisines each, up to 8. The bridge is what lets a cuisine slicer filter restaurant-level measures without duplicating restaurant rows in the fact table.

The report is four pages of my own design, with a custom theme and page navigation: **Overview**, **Ratings Analysis**, **Cost & Services**, and **Cuisines**.

## Findings

**Online delivery peaks in the middle of the price range, and falls away sharply at the top.**

| Price tier | Restaurants | Offering delivery | Rate |
|---|---|---|---|
| 1 | 4,438 | 701 | 15.8% |
| 2 | 3,113 | 1,286 | **41.3%** |
| 3 | 1,405 | 411 | 29.3% |
| 4 | 586 | 53 | 9.0% |

The relationship is an inverted U, not a straight line, and it is steep: tier 2 runs 2.6 times the rate of tier 1 and 4.6 times tier 4. Mid-tier restaurants are the ones for whom delivery economics work, with a high enough ticket to absorb the fee and a low enough positioning that the dining room is not the product.

**But price is the wrong axis to plan on. What a restaurant cooks predicts delivery adoption far better than what it charges.** Across the 23 cuisines with at least 150 restaurants, the correlation between a cuisine's average cost and its delivery rate is **0.04**, which is no relationship at all. The spread by cuisine, however, is enormous:

| Lowest adoption | | Highest adoption | |
|---|---|---|---|
| Mithai (Indian sweets) | 9.7% | Healthy Food | 68.0% |
| Street Food | 13.3% | Thai | 46.6% |
| Seafood | 15.5% | Burger | 45.0% |
| Beverages | 20.2% | Biryani | 44.6% |

Mithai averages $3.66 for two and Seafood averages $32.75. They sit nine times apart on price and land within six points of each other on delivery. Burger and Mexican are within $3 of each other and 15 points apart. The plausible reading is that adoption depends on whether the food survives a journey and whether eating it is an occasion, and sweets, street food and seafood are none of those things. A platform planning delivery expansion should be building cuisine lists, not price lists.

The largest single block of under-penetrated supply is **North Indian: 3,960 restaurants at 28.8%**, below Chinese at a near-identical price point.

**The dataset is overwhelmingly Indian.** India accounts for 8,652 of the 9,542 restaurants, or 90.67 percent. The United States is next at 425, then the United Kingdom at 80. South Africa, the UAE and Brazil have 60 each, and Canada has 4. Any country-level comparison here is really a comparison between India and a set of very small samples, and I have treated it that way rather than ranking countries as if the sample sizes were comparable.

**Average rating is 2.67, but that number is misleading on its own.** 22.51 percent of restaurants fall in the White band, which in this dataset means "Not rated" rather than "rated poorly," and those records drag the average down. Orange is the largest rated band at 39.13 percent, followed by Yellow at 21.97, Green at 11.30, Dark Green at 3.14 and Red at 1.95.

**Converted to a common currency, India is the cheapest market in the dataset, not a mid-range one.** Average cost for two runs from $8.85 in India to $114.19 in Singapore, with the United States at $26.35 and the United Kingdom at $61.08. The dataset-wide average is $11.21, close to the Indian figure because 90 percent of the restaurants are Indian.

## Currency conversion

The cost field in the source data is recorded in each country's local currency, and there is no exchange rate anywhere in the eight source files. Converting to a common unit required an external reference table, which is the only data in this project that did not come from Zomato.

**Rate basis.** All costs are converted at 2019 annual average rates, calculated as the mean of the twelve 2019 monthly averages. The dataset is a 2019 snapshot covering a full year, so a year-long average matches it. Current rates would be wrong, because several of these currencies have moved substantially since 2019.

**Sources.** Nine currencies (INR, ZAR, BRL, SGD, CAD, LKR, GBP, NZD, AUD) come from the Federal Reserve H.10 release, accessed through FRED. Three (IDR, PHP, TRY) come from European Central Bank daily reference rates, averaged across all 255 published 2019 rate days, because the Federal Reserve does not publish those three. The UAE dirham and the Qatari riyal are pegged to the US dollar rather than floating, at 3.6725 and 3.64 respectively, so no averaging applies.

**Validation.** I checked my rates against independent published figures before using them. My calculated 2019 rupee average of 70.4041 agrees with the World Bank's published 70.42, and my Philippine peso average of 51.7938 agrees with an independently published 51.7675. Both within one twentieth of one percent. Every rate, its source and its quote direction are recorded in `Dim_Currency.xlsx`.

## Auditing my own finished work

I built this report, then went back and read it as an outsider would rather than as the person who made it. That pass found three defects. All three are corrected in what is published here, and this section is the most useful part of the project.

**1. Cost was never converted to a common currency.** Average cost by country ran from 24.08 in Australia to 281,190.48 in Indonesia, a spread of roughly 11,700 to 1. That was not a pricing signal, it was an exchange rate: the chart was displaying the relative strength of the rupiah against the Australian dollar and calling it the cost of dinner. **I did not catch this while building the report.** After conversion the same spread is $8.85 to $114.19, about 13 to 1.

Three visuals were carrying it, not one: the Average Cost by Country chart, the Cost & Count by Location matrix, and the Average Cost for Two card on the Overview page. That last one was the most dangerous, because it displayed a mixed-currency average with a dollar sign in front of it and looked entirely reasonable.

Building the conversion table surfaced two further problems in the source data. The currency label `Dollar($)` is shared by the United States, Australia, Singapore and Canada, four different currencies under one name, so a conversion table joined on currency would have converted all four at the US dollar rate and left three of them silently wrong. The join is therefore on country code. Separately, the 22 Philippines restaurants are labelled `Botswana Pula(P)`, a currency the Philippines has never used. Converting at the pula rate implies an average meal for two of $151.59 in Manila against $31.02 at the peso rate, so I treated the label as a source error and converted at the Philippine peso rate.

**2. A deduplication step was deleting real restaurants.** My regional query for Asia ended with a `Table.Distinct` on the restaurant address column alone. Any two restaurants sharing an address string collapsed into one, and Power Query kept whichever happened to sort first. That silently removed **626 restaurants, all Indian**, because Indian addresses in this dataset are mall, market and hotel names that many restaurants share exactly, while addresses elsewhere carry distinct street numbers. The bug was always present; it only had material to bite on in one country.

What it deleted was not noise. Dilli Haat, a Delhi food market with a stall per Indian state, went in as 11 restaurants and came out as 1. The Imperial hotel lost 8 of its 9 restaurants. A single food court in Noida lost 7 of 8.

I found this by checking the report's restaurant count against the source row count and asking where the difference went, which is the same check that caught a join defect in my capstone project.

**3. The cuisine bridge table was collapsing to nothing.** `Dim_Cuisines` was built correctly, splitting each restaurant's cuisine list into 19,705 restaurant-and-cuisine pairs, and then a final `Table.Distinct` on the cuisine column alone reduced it to 145 rows, one per cuisine type, each carrying an arbitrary restaurant. Every measure on the Cuisines page was therefore describing 145 records rather than 9,542 restaurants. Same mistake as the second defect, one query later: deduplicating on a column that was never unique.

**The pattern worth naming:** all three defects produced output that looked completely normal. Nothing errored, no warning appeared, and every number was neatly formatted. A number that looks plausible and is wrong is far harder to catch than a number that looks broken, and the Average Cost for Two card is the example I would give.

## Limitations

**The currency conversion carries assumptions.** A single annual average rate ignores intra-year movement, which matters most for the Turkish lira, the currency that moved most across 2019. And the country samples outside India are small enough that one expensive restaurant moves the average, which is why Singapore's $114.19 rests on 20 restaurants.

**The price range field is banded, and I have not confirmed how.** The delivery-by-price-tier finding uses the platform's own 1 to 4 band rather than the cost column, so it never depended on the currency values. It is still worth confirming whether that field is banded within each market or on a single global scale before extending the finding across countries. Given that 90 percent of the restaurants are Indian, the result is safest read as a finding about the Indian market.

**The ratings are platform ratings, not a quality measure.** They reflect who chose to leave a review, and the 22.51 percent unrated share means the rating distribution describes rated restaurants rather than all restaurants.

**Nine restaurants are excluded.** Nine US records have no cuisine listed and are filtered out, leaving 9,542 of the 9,551 rows in the source.

## Overall

The delivery findings are the ones I would defend. Delivery peaks in the middle of the price range, but price explains almost nothing once you look at cuisine, and for a platform deciding where to spend on delivery partnerships that distinction is the difference between a useful plan and a wasted quarter.

The more valuable outcome was the audit. Three defects, all in my own finished work, all found by reading the report as a stranger and checking numbers against what they should have been rather than against what looked reasonable.

## Files

| File | What it is |
|---|---|
| `Zomato_Restaurant_Analysis.pbix` | The Power BI report |
| `Zomato_ReportSummary.docx` | Full written analysis of the report and its findings |
| `Dim_Currency.xlsx` | The currency conversion table, with every rate, source and assumption documented |
| `screenshots/` | The four report pages |

Source data is not committed to this repository.

## Tools

Power BI Desktop (star schema modeling, bridge tables, Power Query, DAX measures and calculated columns, slicers, custom theme, page navigation), Excel.
