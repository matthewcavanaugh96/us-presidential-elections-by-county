# us-presidential-elections-by-county
Analyzing United States Presidential election results at the county level from 2000 to 2020.

Pardon the mess. I am still optimizing my scripts, developing new ideas, and working on a data storage structure. The main notebook will be split into several smaller ones at some stage in the future.

DATA SOURCES

1. Election results were obtained from Harvard Dataverse: https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/VOQCHQ
2. Education and unemployment data were obtained from the Economic Research Service, a sub-agency of the Department of Agriculture: https://www.ers.usda.gov/ (For education, estimates were only available for 2000, 2008-2012, and 2019-2023. I assigned 2008-12 values to 2010, and 2019-23 values to 2021, then interpolated estimates for years between.)
3. Population data were obtained from the United States Census Bureau. 2000, 2010, and 2020 data were taken from official Census figures, and other years use Census Bureau estimates: https://www.census.gov/

I intend to add more statistics such as income, life expectancy, demographics, and English profiency (not a final list).

Finding and cleaning data has been a project of its own.

I'd also like to include other factors such as the approval rating of the incumbent President (and their party), polling averages, and social media sentiment, but I'm unsure if such data are available at the county level; if not, how I might interpolate them; and if county interpolation is not desirable, whether it makes sense to apply nationwide figures to every county.

I've created a Tableau workbook to visualize certain statistics. It will be updated as more are added.

I have been working with machine learning models. My basic methodology is as follows:
1. Set two targets: Republican percentage and Democratic percentage. This helps to partially account for errors that only affect one party (as notably occurred in Utah in 2016).
3. Predict each county for election, using the other elections as training.
4. Gather statistics such as Absolute Error, Median Error, Z-Score and more for each party for each county.
5. Take the two-party average (2PA) for each of these values for each county. For example, in a county where Democrats are overestimated by 1 point and Republicans are overestimated by 3, the 2PA Absolute Error will be 2. The 2PA averages will be regarded as the ultimate measures of performance.
6. Aggregate these stats across county, state, year, and state/year combination.
I am still experimenting with validation techniques such as GridSearchCV, K-Folds, and feature selection.

Once I have maximally optimized a model, I will attempt to simulate the 2024 election using the full 2000-2020 dataset as training. From county results, statewide totals will be summed and electoral votes assigned accordingly.
My primitive version of this 2024 model predicted a Democratic victory, the opposite of the real-life result.

If I can add real 2024 results to my dataset, I will then move on to predicting 2028.

STATES AND COUNTIES EXCLUDED, NO OBVIOUS FIX:

1. ALASKA - Alaska does not have counties in the traditional sense, but is instead broken into "organized" and "unorganized" boroughs. This would be fine except some data sources do not use these, but instead use "districts" with entirely different boundaries and names. Until or unless I can find a method to reconcile these differing statistical areas, I will have to exclude Alaska from the dataset.
2. VIRGINIA - In Virginia, dozens of "independent cities" exist which are treated as county equivalents, and many of these share names with counties despite being separate entities. FIPS Codes have ensured data integrity, and independent cities have been re-labeled as such for clarity. Two former independent cities have given up such status since the beginning of this dataset - Clifton Forge, which merged into Allegheny County in 2001; and Bedford City, which merged into Bedford County in 2013. Though data exist for both prior to their status changes, I have excluded them as I believe models would be confused by missing data.
3. COLORADO - Broomfield County did not exist until 2001. The city of Broomfield was previously spread across four counties before being granted the status of Consolidated City/County for more efficient self-governance. Thus, it is missing from most 2000 data. I am considering interpolating values based on real ones, but this would also artifically inflate Colorado's statewide vote totals for 2000 as its votes were spread across the other counties, unless *their* 2000 values could be interpolated downward to estimate figures if Broomfield had never been a part of them. For now, I have excluded it.
4. HAWAII - Kalawao County, though named as such, has no administrative functions of its own. It was established as a leprosy quarantine settlement. To this day it is inhabited only by the descendants of the initial patients, land access is only by mule trail, public visitation requires official permission, and as of 2020 its population was just 82. It was entirely absent from some of the datasets, so I have excluded it.
5. NON-COUNTY VOTES - From 2012 onward, Connecticut, Maine, and Rhode Island have tabulated special types of votes such as overseas and write-ins which are not assigned to any particular county. I wonder if I could assign these votes proportionally to counties by population, then assign votes proportionally within the counties by each party's percentage.

NOT A PROBLEM FOR NOW, BUT COULD CAUSE COMPLICATIONS:
1. CONNECTICUT - In Connecticut, counties ceased most administrative functions in 1960, but remained for statistical purposes until being replaced by new "planning areas" with different boundaries in 2022. As all elections in the dataset took place prior to this change, data integrity is not an issue. However, visualization tools such as Tableau may not know how to reconcile the old counties with the new planning areas. This will become a larger problem when/if I add 2024 data.
2. SOUTH DAKOTA - Oglala Lakota County was known as Shannon County until 2015. Entries with the old name have been updated to use the new one. For the purposes of this project, this will not cause any problems, but complications could arise if my compiled dataset is compared to older ones.

ACCIDENTALLY EXCLUDED, HOPE TO FIX:
1. KANSAS CITY/JACKSON COUNTY - Kansas City, Missouri was listed in the county column of the Harvard election dataset despite not being a county equivalent, so I dropped it. However, this has caused Jackson County, in which it is contained, to have inaccurate results, showing GOP victories when in reality it has not voted Republican since 1972. The two may have been split for some reason, as a result of which the Jackson County entry would only reflect its non-Kansas City portion. If this is the case, recombining their totals should be easy. If not, there may be a deeper problem with the data. It is strange, as this doesn't seem to have been done for any other non-county equivalent municipality.
2. SPELLING ERRORS - St. Mary's County, Maryland, and the District of Columbia ended up absent from the final dataset. Both had spelling inconsistencies which arose repeatedly during cleaning, and which I thought I had resolved, but I will have to check again.

OTHER QUIRKS:
You may notice that Loving County, TX, frequently has more votes than residents, causing turnout above 100%. This is a well-documented phenomenon which has resulted in lawsuits and is not a data problem.

TO DO LIST AND IDEAS:
1. Need more visualizations.
2. Restructure data. Rename workbooks and data source files for clarity. Split main notebook into sections, named numerically for sorting.
3. IDEA - After model results are compiled, print the results with each year excluded
4. IDEA - Interpolate data between the election years and simulate them as if an election took place.
5. IDEA - SMOTER (regression equivalent of SMOTE, in which synthetic data points are generated to strengthen the relationship between features)
6. How to handle the Green and Libertarian Parties? The election dataset provides separate columns for these parties, but they have not been on the ballot in every state in every election since 2000. On one hand, the absence of a minor party could cause the model to correctly assign higher vote shares to the major ones, but I also fear the model could make incorrect political inferences about why votes dropped to 0. For now, they have been rolled into a generic "Other" category for vote totals and percentages.
7. Color code columns for parties. Tried a primitive version of this, didn't work. Background color, or just text?
8. Rename percentage columns to include the % symbol
9. Potentially rename the "Bachelor's degree or higher" to remove the apostrophe, as it is causing quote escape problems
10. In the Tableau workbook, I need to alter how I calculate the mean education rates by county. Currently I am taking a mean of county values, which is not fair as counties are not equally populated. Instead I need to do something like (SUM Population) / (SUM(Population by county * High school graduate)).







