# Zomato Restaurant Analysis

A four-page Power BI report on 8,916 restaurants across 15 countries, built on a star schema and designed to answer where online delivery adoption actually concentrates.

Built as a project for the Purdue University / Simplilearn Post Graduate Program in Data Analytics.

## The question

Restaurant platforms tend to assume online delivery adoption rises with price: nicer restaurants, more delivery. I wanted to see whether the data supported that, and what else the ratings and cost fields would show once the countries were separated out.

## Data model

The source data arrived as eight regional Excel files plus a country code lookup. I modeled it as a star schema rather than working from a flat table:

- `Fact_Restaurants` holds one row per restaurant with cost, rating, price range, and the delivery and booking flags.
- `Dim_Country` and `Dim_Cuisines` join to the fact table as dimensions.
- `Dim_Currency` is a reference table I built to convert cost into a single currency. It joins to the fact table on country code. See the currency conversion section below.

Splitting the dimensions out meant the country and cuisine slicers filter every visual on every page consistently, and it keeps the cuisine values (which are multi-valued and messy in the raw data) from duplicating restaurant rows in the fact table.

The report is four pages of my own design, with 56 visuals, 18 slicers, a custom theme, and page navigation:

- **Overview** (15 visuals, 4 slicers)
- **Ratings Analysis** (13 visuals, 4 slicers)
- **Cost & Services** (15 visuals, 6 slicers)
- **Cuisines** (13 visuals, 4 slicers)

## Findings

**Online delivery peaks in the middle of the price range, not at the top.** Delivery availability runs about 17 percent in price tier 1, jumps to roughly 41 percent in tier 2, falls to about 31 percent in tier 3, and drops to roughly 20 percent in tier 4. The relationship is an inverted U, not a straight line. Mid-tier restaurants are the ones for whom delivery economics work: high enough ticket to absorb the fee, low enough positioning that the dining room is not the product.

**The dataset is overwhelmingly Indian.** India accounts for 8,026 of the 8,916 restaurants, or 90.02 percent. The United States is next at 425, then the United Kingdom at 80. South Africa, the UAE, and Brazil have 60 each, and Canada has 4. Any country-level comparison in this report is really a comparison between India and a set of very small samples, and I have treated it that way rather than ranking countries as if the sample sizes were comparable.

**Average rating is 2.67, but that number is misleading on its own.** 22.43 percent of restaurants fall in the White band, which in this dataset means "Not rated" rather than "rated poorly." Those unrated records drag the average down. Orange is the largest rated band at 39.23 percent, followed by Yellow at 21.48 percent, Green at 11.53 percent, Dark Green at 3.31 percent, and Red at roughly 2 percent.

**Once cost is converted to a common currency, India is the cheapest market in the dataset, not a mid-range one.** Average cost for two runs from 8.33 US dollars in India to 114.19 in Singapore, with the United States at 26.35 and the United Kingdom at 61.08. The dataset-wide average is 10.90 US dollars, which sits close to the Indian figure because 90 percent of the restaurants are Indian. Before conversion this chart was unreadable, and the section below explains why.

## Currency conversion

The cost field in the source data is recorded in each country's local currency. There is no exchange rate anywhere in the eight source files, so converting to a common unit required an external reference table, which is the only data in this project that did not come from Zomato.

**Rate basis.** All costs are converted at 2019 annual average rates, calculated as the mean of the twelve 2019 monthly averages. The dataset is a 2019 snapshot covering a full year, so a year-long average matches it. Current rates would be wrong, because several of these currencies have moved substantially since 2019.

**Sources.** Nine currencies (INR, ZAR, BRL, SGD, CAD, LKR, GBP, NZD, AUD) come from the Federal Reserve H.10 release, accessed through FRED. Three (IDR, PHP, TRY) come from European Central Bank daily reference rates, averaged across all 255 published 2019 rate days, because the Federal Reserve does not publish those three. The UAE dirham and the Qatari riyal are pegged to the US dollar rather than floating, at 3.6725 and 3.64 respectively, so no averaging applies.

**Validation.** I checked my rates against independent published figures before using them. My calculated 2019 rupee average of 70.4041 agrees with the World Bank's published figure of 70.42, and my Philippine peso average of 51.7938 agrees with an independently published 2019 average of 51.7675. Both are within one twentieth of one percent. Every rate, its source, and its quote direction are recorded in `Dim_Currency.xlsx`.

**Two source-data problems the conversion exposed.** Neither was visible until I tried to join on the currency column:

1. The currency label `Dollar($)` is used by the United States, Australia, Singapore, and Canada, which are four different currencies sharing one name. A conversion table joined on currency would have converted all four at the US dollar rate and left three of them silently wrong, so the join key is country code rather than currency.
2. The 22 Philippines restaurants are labelled `Botswana Pula(P)`. The Philippines has never used the pula. Converting at the pula rate implies an average meal for two of 151.59 US dollars in Manila; converting at the peso rate gives 31.02, which is consistent with the restaurants in the sample. I treated the label as a source error and converted at the Philippine peso rate.

## Limitations

**The cost field was originally left unconverted, and I did not catch it while building the report.** Average cost by country ran from 24.08 in Australia to 281,190.48 in Indonesia, a spread of roughly 11,700 to 1. That was not a pricing signal, it was an exchange rate: the chart was displaying the relative strength of the rupiah against the Australian dollar and calling it the cost of dinner.

I found it reviewing my own finished work rather than while building it, which is the more useful lesson of the two. Reviewing the report as a reader, rather than as the person who made it, is what surfaced it.

**It is now corrected.** The conversion described above is implemented in the report, and the same spread is 8.33 to 114.19 US dollars, or about 13 to 1. Three visuals were carrying the defect, not one: the Average Cost by Country chart, the Cost & Count by Location matrix, and the Average Cost for Two card on the Overview page. That last one was the most dangerous of the three, because it displayed a mixed-currency average with a dollar sign in front of it and looked entirely reasonable.

Cost figures in this report can now be compared across countries. Two caveats remain. Converting at a single annual average rate ignores intra-year currency movement, which matters most for the Turkish lira, the currency that moved most across 2019. And the country samples outside India are small enough that a single expensive restaurant moves the average, which is why Singapore's 114.19 sits on 20 restaurants and should be read with that in mind.

**The price range field is banded, and I have not confirmed how.** The delivery-by-price-tier finding is stated in terms of the platform's own 1 to 4 price range field rather than the cost column, so it never depended on the currency values and was unaffected by the correction. It would still be worth confirming whether that field is banded within each market or on a single global scale before extending the finding across countries. Given that 90 percent of the restaurants are Indian, the result is safest read as a finding about the Indian market.

**The ratings are platform ratings, not a quality measure.** They reflect who chose to leave a review, and the 22 percent unrated share means the rating distribution describes rated restaurants rather than all restaurants.

## Overall

The delivery-by-price-tier result is the finding I would defend. It is directional, it has an obvious business reading, and a platform expanding delivery coverage should target the middle of the price distribution rather than working down from the top.

The currency problem is the part of this project I learned the most from. I built a report, published nothing, went back to read it as an outsider would, and found a chart that was confidently wrong. Documenting it was the first step; correcting it, sourcing the rates from central bank data, validating them against independent figures, and finding two further label defects in the source along the way, was the more useful one. A number that looks plausible and is not is harder to catch than a number that looks broken, and the Average Cost for Two card is the example I would give.

## Files

| File | What it is |
|---|---|
| `Zomato_Restaurant_Analysis.pbix` | The Power BI report |
| `Zomato_ReportSummary.docx` | Full written analysis of the report and its findings |
| `Dim_Currency.xlsx` | The currency conversion table, with every rate, source, and assumption documented |
| `screenshots/` | The four report pages |

Source data is not committed to this repository.

## Tools

Power BI Desktop (star schema modeling, DAX measures, calculated columns, slicers, custom theme, page navigation), Excel.
