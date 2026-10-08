# World Happiness Report — Power BI Dashboard

A 5-page Power BI dashboard built around one question: what actually drives happiness across world regions, and how did that change between 2015 and 2019?

## Business question

What drives happiness across world regions, and how have these factors evolved from 2015 to 2019?

## Data source

[World Happiness Report (2015–2019)](https://www.kaggle.com/datasets/unsdsn/world-happiness) from Kaggle, originally published by the UN Sustainable Development Solutions Network. The data comes as 5 separate Excel sheets, one per year, and none of them use the same column layout.

## Tools

- Power Query for cleaning and standardizing the 5 sheets
- Excel (VLOOKUP) to backfill missing Region values
- DAX for the rankings, year-over-year comparisons, and KPI cards
- Power BI Desktop for the modeling and the dashboard itself

## Cleaning and modeling

The raw sheets were a mess in a few specific ways, and most of the actual work on this project was getting them into shape before any charts happened:

- Column names and order weren't consistent year to year, so I standardized all 5 sheets to the same structure, then appended them into one combined table ("Complete Table") with a Year column marking which year each row came from
- Several years didn't have a Region column at all. I used VLOOKUP with absolute references to pull Region in from the years that did have it, then manually researched and filled the handful of countries no sheet covered
- A few countries were spelled differently depending on the year ("Trinidad and Tobago" vs. "Trinidad & Tobago"), which quietly broke every year-over-year calculation until I caught it and standardized the names
- Unpivoted the six happiness-factor columns (GDP, Social Support, Health, Freedom, Generosity, Trust) into a long format so the Factors page could let users pick a factor and see it plotted live
- Built a separate Year table and related it to the main fact table to support the time-based measures

## Pages

1. **Region** — average happiness score by region, plus top 5 / bottom 5 countries and the happiest/least happy country and region
2. **Factors** — which factors actually move the needle on happiness, with a scatter plot where you pick the factor
3. **Winners & Losers** — which countries moved the most, up or down, from 2015 to 2019
4. **The Paradox** — whether money buys happiness (mostly, but not always)
5. **Country Deep Dive** — pick any of the 156 countries and see its score, rank, 5-year trend, and factor breakdown

## What stood out

- Social Support predicts happiness better than GDP per capita does
- The top 10 happiest countries are almost all Western Europe and North America — happiness clusters geographically more than I expected
- Benin and Venezuela went in opposite directions over the 5 years: one of the poorest countries on the list climbed, one of the wealthiest fell apart
- Costa Rica ranks 13th in happiness despite being 67th in wealth. Qatar is the reverse — 1st in wealth, only 28th in happiness. The gap seems to come down to things money doesn't buy directly: belonging, trust, freedom

## A couple of the DAX measures

The year-over-year score change needed to ignore the slicer's default filter context and read the selected year manually, otherwise it couldn't look back a year for comparison:

```dax
Dynamic Score Change = 
VAR SelectedYear = SELECTEDVALUE('Year Table'[Year], 2019)
RETURN
IF(
    SelectedYear = 2015,
    0,
    VAR CurrentScore = CALCULATE(AVERAGE('Complete Table'[Happiness Score]), REMOVEFILTERS('Year Table'), 'Complete Table'[Year] = SelectedYear)
    VAR PrevScore = CALCULATE(AVERAGE('Complete Table'[Happiness Score]), REMOVEFILTERS('Year Table'), 'Complete Table'[Year] = SelectedYear - 1)
    RETURN IF(ISBLANK(CurrentScore) || ISBLANK(PrevScore), BLANK(), CurrentScore - PrevScore)
)
```

And the Biggest Climber card, which needed to return an actual country name rather than a number, something a plain Top N filter on a card visual won't do reliably:

```dax
Biggest Climber = 
VAR ScoreChanges = 
    ADDCOLUMNS(
        VALUES('Complete Table'[Country]),
        "Change", CALCULATE([2019 happiness score] - [2015 happiness score])
    )
VAR CleanChanges = FILTER(ScoreChanges, NOT ISBLANK([Change]))
VAR MaxChange = MAXX(CleanChanges, [Change])
RETURN
    MAXX(FILTER(CleanChanges, [Change] = MaxChange), 'Complete Table'[Country])
```

## Opening it yourself

Download `Happiness_Analysis_Report.pbix` from `/dashboard` and open it in Power BI Desktop. Year filter, country slicer, and factor selector are all interactive.

## Screenshots

*(page screenshots go here — see `/screenshots`)*

---

Built by Moath.
