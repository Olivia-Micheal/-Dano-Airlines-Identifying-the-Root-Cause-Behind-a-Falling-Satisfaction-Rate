# Dano Airlines: Identifying the Root Cause Behind a Falling Satisfaction Rate

A Power BI dashboard that analyzes passenger satisfaction survey data for Dano Airlines, built to find the exact cause behind a satisfaction rate that fell below 50 percent for the first time in the airline's history, and to show which passengers are being let down the most.

## Opening Hook

129,880 passengers surveyed, and the biggest driver of dissatisfaction is not what most airlines would guess. It is not flight delays. It is Online Boarding, a step most passengers barely think about, and it is failing one specific group of travelers far more than everyone else.

## Table of Contents
- [Project Overview](#project-overview)
- [Business Problem](#business-problem)
- [Tools and Skills](#tools-and-skills)
- [Dataset Description](#dataset-description)
- [Data Cleaning and Transformation](#data-cleaning-and-transformation)
- [Data Model](#data-model)
- [DAX Measures and Design Decisions](#dax-measures-and-design-decisions)
- [Dashboard Walkthrough](#dashboard-walkthrough)
- [Key Insights and Findings](#key-insights-and-findings)
- [Summary and Conclusion](#summary-and-conclusion)
- [Recommendations](#recommendations)
- [How to Explore This Dashboard](#how-to-explore-this-dashboard)
- [Live Dashboard Link](#live-dashboard-link)
- [Author and Contact](#author-and-contact)

## Project Overview

Dano Airlines saw its passenger satisfaction rate drop below 50 percent for the first time. This project analyzes a full passenger survey dataset to find the exact cause, rather than guessing. The brief did not specify a tool or a chart type, only a single page report. I chose Power BI, and built a one page executive summary that identifies the single biggest driver of dissatisfaction and names exactly which passengers are affected most.

## Business Problem

Leadership at Dano Airlines needed a data driven answer to one question: why is satisfaction falling, and what should be done about it? A vague answer like "improve customer service" would not be useful. This project was built to give a specific, measurable answer that points to one clear action.

## Tools and Skills

**Tools used:** Power BI Desktop, Power Query, DAX, Power BI Service

**Skills demonstrated in this project:**
- Structuring a 24 column survey dataset into clear categories before starting any analysis
- Auditing data quality and making a defensible decision for missing values and zero ratings, instead of deleting rows
- Choosing median over mean for a heavily skewed column, and explaining why
- Building a data model with a reference table and an unpivoted table to compare 13 different service ratings without duplicating logic
- Writing a DAX measure that finds the true root cause of dissatisfaction by comparing two groups directly, rather than relying on a single average
- Designing a single page executive dashboard where every number earns its place
- Diagnosing and fixing a real data bug where a chart was reading from the wrong table, producing a false result

## Dataset Description

The dataset, `Airline_data.xlsx`, contains 129,880 rows and 24 columns, one row per passenger survey response. I grouped the columns into four categories before starting any analysis:

| Group | Columns |
|---|---|
| Profile | ID, Gender, Age, Customer Type, Type of Travel, Class |
| Experience | Flight Distance, Departure Delay, Arrival Delay |
| Perception | 13 service ratings, each scored 0 to 5: Departure and Arrival Time Convenience, Ease of Online Booking, Check-in Service, Online Boarding, Gate Location, On-board Service, Seat Comfort, Leg Room Service, Cleanliness, Food and Drink, In-flight Service, In-flight Wifi Service, In-flight Entertainment, Baggage Handling |
| Outcome | Satisfaction, recorded as Satisfied or Neutral or Dissatisfied |

*Figure 1: Raw Dataset*

## Data Cleaning and Transformation

Here is what I found in the data, and the decision I made for each issue:

| Issue found | What I did and why |
|---|---|
| No duplicate passenger IDs | No action needed |
| Rating columns containing a value of 0 | A rating of 0 means the service was not applicable to that passenger, not that they rated it zero. I kept these rows in the dataset and excluded the 0 values only inside the rating measures, never by deleting or converting them |
| 393 missing Arrival Delay values | I left these blank instead of filling them with 0. Filling a missing value with 0 would falsely claim the flight had no delay at all. Power BI's median and average functions ignore blanks automatically, so leaving them blank was the correct choice |
| Arrival Delay outliers reaching over 1,500 minutes | These are real extreme delays, not data errors, so I kept them in the dataset. Because they are extreme, the mean delay was skewed to 15.1 minutes, which does not reflect the typical passenger experience. I used the median instead, which is not distorted by a small number of extreme outliers |
| Passenger age ranging from 7 to 85 | This is a realistic range for airline passengers, so no action was needed |

*Figure 2: Data Cleaning*

## Data Model

I built two tables. The first is the Main table, left untouched, with one row per passenger. It powers the KPI cards and the three segment charts.

The second is Service Ratings Unpivoted, built as a Power Query reference of the Main table, not a duplicate. I kept the ID, Class, Type of Travel, Customer Type, and Satisfaction columns, then unpivoted the 13 individual rating columns into two columns: Service Area and Rating. I connected the two tables through a one to many relationship on passenger ID, filtering in a single direction.

I made a deliberate choice never to sum the 13 rating columns together. Each one measures something different, from seat comfort to wifi quality, and summing them would destroy the specific signal each one carries.

*Figure 3: Data Model*

## DAX Measures and Design Decisions

The core measures in this project are Total Passengers Surveyed, Satisfied Passengers, Dissatisfied Passengers, Overall Satisfaction Rate, and Median Arrival Delay, all built directly from the Main table.

The more important measures live in the Service Ratings Unpivoted table. I built Average Rating, filtered to exclude the 0 values that represent "not applicable," and I built this exclusion logic in exactly one place. Every other measure that depends on an average rating inherits this exclusion automatically, so I never had to repeat the same filter condition across multiple formulas.

From that base measure, I built Satisfied Avg Rating and Dissatisfied Avg Rating, each filtered to their respective passenger group, and then Rating Gap, which is simply Satisfied Avg Rating minus Dissatisfied Avg Rating, calculated separately for each of the 13 service areas.

This Rating Gap measure is the single most important decision in this project. A small gap means satisfied and dissatisfied passengers feel about the same way toward that service, so it is not driving the split between them. A large gap means that service is a genuine differentiator between happy and unhappy passengers. I did not rely on a single average rating to find the root cause, because a service can have a mediocre average rating and still not be the actual problem. Comparing the two groups directly is what actually reveals a root cause.

## Dashboard Walkthrough

*Figure 4: Full Dashboard*

Start with the four cards at the top left. Passenger Surveyed shows 130,000, giving scale to everything else on the page. Satisfaction Rate shows 43 percent, the number that triggered this entire project. Arrival Delay shows a median of 0 minutes, which already tells you something important before you look at a single chart: most passengers were not meaningfully delayed at all.

Now look at the fourth card, Root Cause. It shows Online Boarding with a rating gap of 1.45, filtered specifically to that one service area out of the 13 measured. This card exists because of the chart beside it.

Look at that chart now, the 13 bar comparison across every service area. Each pair of bars shows the satisfied passengers' average rating next to the dissatisfied passengers' average rating for that service. Scan across all 13 pairs and one gap is visibly wider than the rest: Online Boarding, at 4.2 for satisfied passengers against 2.7 for dissatisfied passengers. Compare that to Departure and Arrival Time Convenience on the right side of the chart, where both bars sit close together. That is the proof behind the Root Cause card. Delays barely separate happy passengers from unhappy ones. Online Boarding separates them more than anything else measured.

Now look at the three charts along the bottom. Satisfaction by Type of Travel shows Business travelers at 58 percent satisfied against Personal travelers at only 10 percent, the widest gap on the entire page. Satisfaction by Class shows the same pattern, Business Class at 69 percent against Economy at just 19 percent. Satisfaction by Customer Type shows Returning customers at 48 percent against First-time customers at 24 percent. Every one of these three charts points at the same group: newer, lower class, personal travelers are the least satisfied passengers by a wide margin.

**Key takeaway:** The story on this dashboard is not about delays, even though delays are the easiest thing to blame. It is about a digital boarding process that fails specific groups of passengers far more than others, and any fix aimed at the wrong problem will not move the satisfaction number at all.

## Key Insights and Findings

Online Boarding has the widest satisfied versus dissatisfied rating gap of all 13 services measured, at 1.45, making it the clearest driver of dissatisfaction in this dataset. Flight delays, despite being the most commonly assumed culprit for airline complaints, show almost no gap between satisfied and dissatisfied passengers, meaning they are not actually driving the split. Personal travelers, Economy passengers, and first time customers are consistently the least satisfied groups across every single segment chart on this dashboard, each sitting between 24 and 48 percentage points behind their counterparts.

## Summary and Conclusion

This project set out to find one specific answer: why did Dano Airlines' satisfaction rate fall below 50 percent, and what should be done about it? The answer is Online Boarding, identified not by looking at a single average rating, but by directly comparing what satisfied and dissatisfied passengers experienced. That same comparison ruled out flight delays as the cause, despite being the obvious first guess. The three segment charts then narrow the fix even further, showing that Personal travelers, Economy passengers, and first time customers are the groups most in need of it. A generic customer service improvement plan would miss all of this. This dashboard replaces a guess with a specific, defensible answer.

## Recommendations

- Prioritize a full redesign of the Online Boarding process before addressing any other service area, since it is the clearest driver of dissatisfaction in this dataset
- Do not prioritize delay reduction as a satisfaction fix. The data shows delays are not a meaningful factor in whether a passenger ends up satisfied or dissatisfied
- Target the Online Boarding fix specifically at Personal travelers, Economy passengers, and first time customers first, since these three groups are consistently the least satisfied across every segment measured

## How to Explore This Dashboard

Use the Class, Type of Travel, and Customer Type slicers at the top right of the dashboard to filter the entire page down to a specific passenger segment. This lets you check whether the Online Boarding gap holds true even within a single group, such as Economy passengers alone, rather than only looking at the citywide total.

## Live Dashboard Link

[Insert Power BI published link, or a note that the dashboard is available as an image or PDF in this repository]

## Author and Contact

Olivia Anetoh is a Data Analyst who works with real world data to uncover business risks, performance patterns, and insights that support better decisions. This project reflects that approach from start to finish, from choosing median over mean to avoid a misleading average, to building a rating gap measure that finds a genuine root cause instead of settling for a surface level number.

[LinkedIn](https://www.linkedin.com/in/olivia-anetoh-955b94328)
[GitHub](https://github.com/Olivia-Micheal)
Email: anetohchinecherem@gmail.com
