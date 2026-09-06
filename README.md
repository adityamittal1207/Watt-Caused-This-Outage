# Watt Caused This Outage?
Identifying factors related to the cause of power outages across the U.S. 

Final Project for DSC 80

# Introduction

This project will revolve around the analysis of dataset contraining major power outage data in the continental U.S. from January 2000 to July 2016. This data for the dataset was acquired through different publicly avaliable datasets, coming from: OE-417 form Schedule 1 published by DOE׳s Office of Electricity Delivery and Energy Reliability, the Energy Information Administration (EIA) [form EIA-826 and EIA-861], the National Oceanic and Atmospheric Administration (NOAA) and National Climatic Data Center (NCDC), the U.S. Department of Labor, the Bureau of Labor Statistics, and the U.S. Census Bureau. 

The dataset is comprised of 1534 relevant rows, with 56 columns of different columns to us.

The main question we will be answering in this is: *Can we predict outage categories from purely demographical info?* This dataset and problem are particularly interesting, as they have real-world relevance. If we are truly able to predict outage categories from demographical info, decision-makers across the U.S. will be able to retroactively predict outage types and effectively prioritize the right areas to support the power grid with - improving quality of life and saving government money. 

The columns we will be primarily analysing revolve around answering this question, so are related to either demographic info or outage category. The columns are listed below (descriptions sourced from [https://www.sciencedirect.com/science/article/pii/S2352340918307182](https://www.sciencedirect.com/science/article/pii/S2352340918307182)):

| Plain English | Column | Description |
|---|---|---|
| State population | `POPULATION` | Population of the state in that year |
| Urban population share | `POPPCT_URBAN` | Share of the state's total population living in urban areas (%) |
| Urban cluster population share | `POPPCT_UC` | Share of the state's total population living in urban clusters (%) |
| Urban population density | `POPDEN_URBAN` | Population density of urban areas (persons per square mile) |
| Urban cluster population density | `POPDEN_UC` | Population density of urban clusters (persons per square mile) |
| Rural population density | `POPDEN_RURAL` | Population density of rural areas (persons per square mile) |
| Urban land share | `AREAPCT_URBAN` | Share of the state's land area made up of urban areas (%) |
| Urban cluster land share | `AREAPCT_UC` | Share of the state's land area made up of urban clusters (%) |
| Share of U.S. land area | `PCT_LAND` | State's land area as a share of total continental U.S. land area (%) |
| Share of U.S. water area | `PCT_WATER_TOT` | State's water area as a share of total continental U.S. water area (%) |
| Share of U.S. inland water | `PCT_WATER_INLAND` | State's inland water area as a share of total continental U.S. inland water area (%) |
| Per capita state GDP | `PC.REALGSP.STATE` | Per capita real gross state product, in 2009 chained dollars |
| State GDP vs. national | `PC.REALGSP.REL` | State per capita real GSP as a fraction of U.S. per capita real GDP |
| Year-over-year GDP growth | `PC.REALGSP.CHANGE` | Change in per capita real GSP from the previous year (%) |
| Utility share of state GDP | `UTIL.CONTRI` | Portion of the state's total real GDP contributed by the utility industry (%) |
| State share of U.S. utility earnings | `PI.UTIL.OFUSA` | State utility sector earnings as a share of total U.S. utility sector earnings (%) |
| Residential customer share | `RES.CUST.PCT` | Share of the state's electricity customers that are residential (%) |
| Commercial customer share | `COM.CUST.PCT` | Share of the state's electricity customers that are commercial (%) |
| Industrial customer share | `IND.CUST.PCT` | Share of the state's electricity customers that are industrial (%) |
| Total customers served | `TOTAL.CUSTOMERS` | Annual count of all electricity customers served in the state |
| Grid reliability region | `NERC.REGION` | North American Electric Reliability Corporation region involved in the outage |
| Climate region | `CLIMATE.REGION` | NOAA climate region — one of nine climatically consistent regions in the continental U.S. |
| El Niño / La Niña phase | `CLIMATE.CATEGORY` | Climate episode for the year: warm, cold, or normal, using a ±0.5 °C threshold on the Oceanic Niño Index |
| Outage cause | `CAUSE.CATEGORY` | Category of event that caused the major outage |

# Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The data cleaning steps I took were centered around the goal of making the data in the dataset as usable as possible when basing models off it in the future. Note that I opted out of dropping columns that I didn't list before as there are still some analyses I might conduct with them to answer different smaller questions. The steps I took are listed below:

1. I begin by converting numeric columns from the previous object data type to numeric data types (float64 and int), using pandas' built in `to_numeric` function. The numeric columns I identified were: `[
    'YEAR', 'MONTH', 'ANOMALY.LEVEL', 'OUTAGE.DURATION', 'DEMAND.LOSS.MW',
    'CUSTOMERS.AFFECTED', 'RES.PRICE', 'COM.PRICE', 'IND.PRICE', 'TOTAL.PRICE',
    'RES.SALES', 'COM.SALES', 'IND.SALES', 'TOTAL.SALES', 'RES.PERCEN',
    'COM.PERCEN', 'IND.PERCEN', 'RES.CUSTOMERS', 'COM.CUSTOMERS',
    'IND.CUSTOMERS', 'TOTAL.CUSTOMERS', 'RES.CUST.PCT', 'COM.CUST.PCT',
    'IND.CUST.PCT', 'PC.REALGSP.STATE', 'PC.REALGSP.USA', 'PC.REALGSP.REL',
    'PC.REALGSP.CHANGE', 'UTIL.REALGSP', 'TOTAL.REALGSP', 'UTIL.CONTRI',
    'PI.UTIL.OFUSA', 'POPULATION', 'POPPCT_URBAN', 'POPPCT_UC',
    'POPDEN_URBAN', 'POPDEN_UC', 'POPDEN_RURAL', 'AREAPCT_URBAN',
    'AREAPCT_UC', 'PCT_LAND', 'PCT_WATER_TOT', 'PCT_WATER_INLAND']`

2. Following this, I got concrete datatime objects for the outage start times and restoration times. Before, these were each divided between their date and time within the day, which wasn't specific enough. To do this, I got the start dates from the `OUTAGE.START.DATE` and `OUTAGE.RESTORATION.DATE` columns, and merged them with the times from the `OUTAGE.START.TIME` and `OUTAGE.RESTORATION.TIME` into one date/time string. I then used the built in `pd.to_datetime` function to convert these to datetime objects. Finally, I verified that all null values from the individual columns carried to the updated merged columns, then dropped the original 4 columns.

3. Next, on the same note of checking durations, I checked the outage duration column for 0 values, and ensured that none of them signified null durations. I verified this by checking if the timedelta between start and restoration time between an outage equalled 0 for all outage durations listed as 0. This was true, so I did not have to replace any 0s will null values in this column.

4. Finally, I created an additional demographic column that I decided might be useful for future analysis, `IS.WEEKEND`, which signifies if an outage started on a weekend. I created this simply by checking if the start day of the week was greater or equal than 5 (meaning Saturday or Sunday). This will be useful for future analyses as more outages of certain types (like intentional attacks) may happen more on weekends than weekdays, and this project revolves around outage type.

The head of the new cleaned dataframe is below:

<div style="overflow-x: auto; white-space: nowrap;" markdown="1">

| OBS | YEAR | MONTH | U.S._STATE | POSTAL.CODE | NERC.REGION | CLIMATE.REGION | ANOMALY.LEVEL | CLIMATE.CATEGORY | CAUSE.CATEGORY | CAUSE.CATEGORY.DETAIL | HURRICANE.NAMES | OUTAGE.DURATION | DEMAND.LOSS.MW | CUSTOMERS.AFFECTED | RES.PRICE | COM.PRICE | IND.PRICE | TOTAL.PRICE | RES.SALES | COM.SALES | IND.SALES | TOTAL.SALES | RES.PERCEN | COM.PERCEN | IND.PERCEN | RES.CUSTOMERS | COM.CUSTOMERS | IND.CUSTOMERS | TOTAL.CUSTOMERS | RES.CUST.PCT | COM.CUST.PCT | IND.CUST.PCT | PC.REALGSP.STATE | PC.REALGSP.USA | PC.REALGSP.REL | PC.REALGSP.CHANGE | UTIL.REALGSP | TOTAL.REALGSP | UTIL.CONTRI | PI.UTIL.OFUSA | POPULATION | POPPCT_URBAN | POPPCT_UC | POPDEN_URBAN | POPDEN_UC | POPDEN_RURAL | AREAPCT_URBAN | AREAPCT_UC | PCT_LAND | PCT_WATER_TOT | PCT_WATER_INLAND | OUTAGE.START | OUTAGE.RESTORATION | IS.WEEKEND |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | 2011 | 7 | Minnesota | MN | MRO | East North Central | -0.3 | normal | severe weather | NaN | NaN | 3060 | NaN | 70000 | 11.6 | 9.18 | 6.81 | 9.28 | 2332915 | 2114774 | 2113291 | 6562520 | 35.5491 | 32.225 | 32.2024 | 2308736 | 276286 | 10673 | 2595696 | 88.9448 | 10.644 | 0.411181 | 51268 | 47586 | 1.07738 | 1.6 | 4802 | 274182 | 1.75139 | 2.2 | 5348119 | 73.27 | 15.28 | 2279 | 1700.5 | 18.2 | 2.14 | 0.6 | 91.5927 | 8.40733 | 5.47874 | 2011-07-01 17:00:00 | 2011-07-03 20:00:00 | False |
| 2 | 2014 | 5 | Minnesota | MN | MRO | East North Central | -0.1 | normal | intentional attack | vandalism | NaN | 1 | NaN | NaN | 12.12 | 9.71 | 6.49 | 9.28 | 1586986 | 1807756 | 1887927 | 5284231 | 30.0325 | 34.2104 | 35.7276 | 2345860 | 284978 | 9898 | 2640737 | 88.8335 | 10.7916 | 0.37482 | 53499 | 49091 | 1.08979 | 1.9 | 5226 | 291955 | 1.79 | 2.2 | 5457125 | 73.27 | 15.28 | 2279 | 1700.5 | 18.2 | 2.14 | 0.6 | 91.5927 | 8.40733 | 5.47874 | 2014-05-11 18:38:00 | 2014-05-11 18:39:00 | True |
| 3 | 2010 | 10 | Minnesota | MN | MRO | East North Central | -1.5 | cold | severe weather | heavy wind | NaN | 3000 | NaN | 70000 | 10.87 | 8.19 | 6.07 | 8.15 | 1467293 | 1801683 | 1951295 | 5222116 | 28.0977 | 34.501 | 37.366 | 2300291 | 276463 | 10150 | 2586905 | 88.9206 | 10.687 | 0.392361 | 50447 | 47287 | 1.06683 | 2.7 | 4571 | 267895 | 1.70627 | 2.1 | 5310903 | 73.27 | 15.28 | 2279 | 1700.5 | 18.2 | 2.14 | 0.6 | 91.5927 | 8.40733 | 5.47874 | 2010-10-26 20:00:00 | 2010-10-28 22:00:00 | False |
| 4 | 2012 | 6 | Minnesota | MN | MRO | East North Central | -0.1 | normal | severe weather | thunderstorm | NaN | 2550 | NaN | 68200 | 11.79 | 9.25 | 6.71 | 9.19 | 1851519 | 1941174 | 1993026 | 5787064 | 31.9941 | 33.5433 | 34.4393 | 2317336 | 278466 | 11010 | 2606813 | 88.8954 | 10.6822 | 0.422355 | 51598 | 48156 | 1.07148 | 0.6 | 5364 | 277627 | 1.93209 | 2.2 | 5380443 | 73.27 | 15.28 | 2279 | 1700.5 | 18.2 | 2.14 | 0.6 | 91.5927 | 8.40733 | 5.47874 | 2012-06-19 04:30:00 | 2012-06-20 23:00:00 | False |
| 5 | 2015 | 7 | Minnesota | MN | MRO | East North Central | 1.2 | warm | severe weather | NaN | NaN | 1740 | 250 | 250000 | 13.07 | 10.16 | 7.74 | 10.43 | 2028875 | 2161612 | 1777937 | 5970339 | 33.9826 | 36.2059 | 29.7795 | 2374674 | 289044 | 9812 | 2673531 | 88.8216 | 10.8113 | 0.367005 | 54431 | 49844 | 1.09203 | 1.7 | 4873 | 292023 | 1.6687 | 2.2 | 5489594 | 73.27 | 15.28 | 2279 | 1700.5 | 18.2 | 2.14 | 0.6 | 91.5927 | 8.40733 | 5.47874 | 2015-07-18 02:00:00 | 2015-07-19 07:00:00 | True |

</div>

### Univariate Analyses

I conducted two univariate analyses using plots, shown below:

<iframe
  src="assets/cause-category-pie.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The plot above is a pie chart displaying the percentages of each outage category in the dataset. This shows that intentional attacks cause much more outages than other categories, so we should be wary of that in the future, especially while evaluating our predictive model.

<iframe
  src="assets/outage-duration-hist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The plot above is a histogram displaying the distribution of outage duration in the dataset. This shows that the dataset heavily skews right, meaning that the majority of outage durations lie on the lower end, which might be influenced by other columns like predominant outage causes.

### Bivariate Analyses

I conducted two bivariate analyses using plots, shown below:

<iframe
  src="assets/duration-vs-category-hist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The plot above is a normalized histogram displaying the percentage of duration buckets that are attributed to different outage causes. This shows that lower durations are comprised of more mixed outage causes, but very quickly as outage durations rise, the majority cause becomes severe weather.

<iframe
  src="assets/commercial-percentage-vs-total-price-hist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The plot above is a scatterplot displaying relationship between the percentage of commercial electricity consumption of total electricity consumption in the state and the average monthly electricity price in the state (in cents/kilowatt-hour). This shows a positive relation between the commercial consumption percentage and electricity price, which at first glance makes sense as if more electricity is needed commercially, it can be charged for more to generate more money.

### Interesting Aggregates

On top of these visualizations, I made a few tables showing interesting aggregations of the dataset. I'll display one below:

<table>
  <tr><th></th><th colspan="7">CAUSE.CATEGORY</th></tr>
  <tr><th>CLIMATE.CATEGORY</th><th>equipment failure</th><th>fuel supply emergency</th><th>intentional attack</th><th>islanding</th><th>public appeal</th><th>severe weather</th><th>system operability disruption</th></tr>
  <tr><td>cold</td><td>4.0</td><td>4.0</td><td>25.8</td><td>3.2</td><td>4.7</td><td>50.5</td><td>7.8</td></tr>
  <tr><td>normal</td><td>3.8</td><td>3.5</td><td>30.4</td><td>2.3</td><td>4.6</td><td>47.6</td><td>7.9</td></tr>
  <tr><td>warm</td><td>3.2</td><td>1.6</td><td>22.7</td><td>4.5</td><td>4.2</td><td>53.9</td><td>9.7</td></tr>
</table>


This cross-tabulation shows the normalized cause category percentages of different climate categories. This is interesting, as it shows that some climates have different rates of cause categories. One that stands out is that it seems like normal climate has less severe weather than normal and warm weather (statistical significance isn't evaluated), which would make sense as normal climate is the least likely to have severe weather attacks.

# Assessment of Missingness

There are multiple columns with missing data in this dataset that may be Missing Not at Random (MNAR). The one we will focus on is OUTAGE.RESTORATION (the outage restoration time). I believe this could be MNAR as low restoration times might not be reported as they might've felt too short to report, and high restoration times might not be reported as it would be too much work for the data collectors to piece together the data that led to restoration time calculations. If we collected more information on the data collection process and how restoration times are calculated for longer outages, we could help understand some of the data that lead to missing values here, and it would help explain the missingness to make this feature MAR.

However, as we don't have any of this additional information, we will instead try analyzing if outage restoration time is MAR. The results are below:

### Dependent on: Outage restoration time's missingness depends on anomaly level

I conducted a permutation test to assess the missingness of outage restoration time related to anomaly level (the oceanic El Niño/La Niña (ONI) index).

Null Hypothesis: The missingness of outage restoration time is the same for different anomaly levels.

Alternate Hypothesis: The missingness of outage restoration time is not the same for different anomaly levels.

We get a p-value of 0.0. This means that at a significance level of 0.05, we can reject the null hypothesis. Therefore, it is extremely likely that the missingness of outage restoration time is not the same for different anomaly levels, meaning that outage restoration time is MAR, influenced by anomaly level.

### Not dependent on: Outage restoration time's missingness does not depend on state population

I conducted a permutation test to assess the missingness of outage restoration time related to state population.

Null Hypothesis: The distribution of missingness of outage restoration time is the same for all state populations.

Alternate Hypothesis: The distribution missingness of outage restoration time is not the same for all state populations.

We get a p-value of 0.423. This means that at a significance level of 0.05, we cannot reject the null hypothesis. Therefore, we can state that the population of a state does not likely influence the missingness of the outage restoration time values. This is also visually backed up by the layered histogram below, which shows heavy overlap between the probability density of values being missing or not across state populations. 


<iframe
  src="assets/missingness-viz.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

# Hypothesis Testing

After conducting the bivariate analysis, I became very interested in the question of how cause category influences outage duration. Therefore I conducted a hypothesis test with the hypotheses below:

Null Hypothesis: Outage duration has the same distribution for system operability disruption and intentional attack outages.

Alternative Hypothesis: System operability disruption come from a different outage duration distribution than intentional attack outages.

I chose a test-statistic of absolute difference in means, as we want to determine if these two distributions are different, without any specific direction. We will operate at a significance level of 0.05, as that is standard. 

The p-value of this test was 0.0612, meaning that for this question, we cannot reject the null hypothesis. This means is it likely that outage duration has the same distribution for system operability disruption and intentional attack outages. This permutation test null distribution compared to our observed value is visualized below, which shows that indeed, about 6.12% of the null distribution lies after our observed value. 

<iframe
  src="assets/duration-perm-test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

# Framing a Prediction Problem

Continuing on the core theme of working with outage category, the remainder of this project will be based around the prediction problem below:

Can the cause category of an outage be predicted from purely demographical information of the region the outage took place in?

This will require a classifier, performing multiclass classification for the different outage types. Our response variable is the outage cause (CAUSE.CATEGORY), as that is what we are trying to predict and what this project has been centered around. The metric we are evaluating on is f1, as from what we've seen during the previous analyses, the outage cause is very imbalanced, so f1 will best blend precision and recall to deal with this imbalance. 

Additionally, we don't have to worry about data leakage, as the demographic info has no relation to the outage cause, and will be the same before, during, and after the outage.

# Baseline Model

The model I stuck with to tackle this prediction problem is a `DecisionTreeClassifier` with a `max_depth` of 4, alongside median imputation through a `SimpleImputer`. I decided to only use quantitiative demographic features for this baseline model, coming out to `["POPULATION", "POPPCT_URBAN", "POPPCT_UC",
             "POPDEN_URBAN", "POPDEN_UC", "POPDEN_RURAL",
             "AREAPCT_URBAN", "AREAPCT_UC",
             "PCT_LAND", "PCT_WATER_TOT", "PCT_WATER_INLAND",
             "PC.REALGSP.STATE", "PC.REALGSP.REL", "PC.REALGSP.CHANGE",
             "UTIL.CONTRI", "PI.UTIL.OFUSA",
             "RES.CUST.PCT", "COM.CUST.PCT", "IND.CUST.PCT", "TOTAL.CUSTOMERS"]`.

We will evaluate this model by training/testing it on a basic `train_test_split` with a `test_size` of 0.25 in order to ensure we are testing on unseen data.

This model had a test f1 (weighted) of 0.5526, which I would say means that the model as it is already is good. I say this as if we created the most basic classifier, which always predicts the most dominant category of intentional attacks, we would have an f1 score of 0.117. This means that the baseline model outperforms that pretty well, which is solid initial signal. 

# Final Model

I made quite a few changes between the baseline model and the final model, centered around giving the model extra useful information and tuning the model itself.

### Added Features

I began by adding the following categorial columns encoding them with sklearn's `OneHotEncoder`: `['NERC.REGION', 'CLIMATE.REGION', 'CLIMATE.CATEGORY']`. I chose to add these as `NERC.REGION` provides us information on overall power grid reliability in the area which can influence what causes outages there; `CLIMATE.REGION` gives insights on standardized climates that could influence the types of natural events that lead to outages; `CLIMATE.CATEGORY` provides climate divisions that we already observed seem to have a correlation to outage cause. 

Additionally, I created custom ratio-based columns: `URBAN_RURAL_RATIO` (`POPDEN_URBAN`/`POPDEN_RURAL`) and `CUST_PER_CAPITA` (`TOTAL.CUSTOMERS`/`POPULATION`). These will be useful as they give more standardized insights that relate less to just the state the outage took place in, allowing the model to have more holistic features to work with. 

### Model and Performance

The final model I ended up using was a `RandomForestClassifier` with the following hyperparameters: `mean` imputation, a max depth of `20`, `sqrt` max features per tree, `5` min sample split, and `300` estimators. This got us a final f1 score, on the same test data as the baseline, of **0.6473**. We ended up choosing this model and hyperparameters through a `GridSearchCV` with 5 folds. We ran this with the following combinations:

Decision Tree: `dt_grid = {
    'criterion': ['gini', 'entropy'],
    'max_depth': [3, 4, 5, 7, 10, None],
    'min_samples_split': [2, 5, 10],
}`

Random Forest: `rf_grid = {
    'n_estimators': [100, 300],
    'max_depth': [5, 10, 20, None],
    'min_samples_split': [2, 5, 10],
    'max_features': ['sqrt', 'log2'],
}`.

The best CV f1 score we got for the random forest was 0.6129, compared to 0.5884 for the decision tre, which led to me picking the random forest as our final model which ran against the same test data as the baseline. Note that we didn't evaluate on that same test data until after choosing our final model and hyperparameters as to avoid data leakage.

The final model's performance is an improvement over the baseline, as it has an f1 score just over .06 greater than the baseline model on the same test data. With the good fit of f1 score to evaluate this imbalanced outage cause data, we can be confident that the increased f1 score truly represents higher model quality. 

Additionally, a confusion matrix is included below to show more in depth performances of this final model. It's clear that it's best at predicting severe weather, which does make sense, as high outage durations were almost entirely comprised of severe weather causes, allowing simple predictions at those outage durations.

<iframe
  src="assets/confusion-matrix.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

# Fairness Analysis

The final part of this analysis will be checking how fair the model is across different groups. I chose urban vs. rural populations as our groups, as the urbanity of an area could heavily influence how consistent our predictions are in that area. Urbanity is determined by if the urban population percent is greater than or equal to it's median, or less than it's median.

My thoughts are that rural areas would mostly comprise of natural outage causes, while urban areas would be more hectic in terms of the people, the power grid, and types of problems, so urban areas would include more variance in the type of causes that cause individual outages. However, I'll plan to run a non-directed test, as there is also a possibility that urban areas have better prevention for some types of outage causes as compared to rural areas.

I chose an evaluation metric of weighted precision. This is because if we want this model to influencereal-world decisions, we need to minimize false positives as it false positives would influence decision-makers to waste resources to stopping the wrong causes. Additionally, we weighted this precision due to the imbalance of the outage causes in this dataset. To evaluate this fairness, I conducted a permutation test. I chose to use a test statistic of absolute difference of means as we wanted a two-sided test to simply test if the distributions were different. We will be operating with a significance level of 0.05, as that is standard. I used the following hypothesis:

Null Hypothesis: Our model is fair. Its weighted precision for urban and rural areas are roughly the same, and any differences are due to random chance.
Alternate Hypothesis: Our model is unfair. Its weighted precision for urban areas is different than its weighted precision for rural areas.

Our resulting p-value is 0.0, which is below the significance level of 0.05. This means that we reject the null hypothesis, so it is very likely that our model is unfair, and it's weighted precision for urban areas is different than its weighted precision for rural areas. The visualization below gives more insight on how our observed absolute difference in means compares to the null distribution of the absolute difference of the weighted precisions between the predictions in urban vs. rural areas.

<iframe
  src="assets/fairness-perm-test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
