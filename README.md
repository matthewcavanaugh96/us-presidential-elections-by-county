# us-presidential-elections-by-county
Analyzing United States Presidential election results at the county level from 2000 to 2020.

Pardon the mess. I am still optimizing my scripts, developing new ideas, and working on a data storage structure.

The repository is also getting cluttered up with Tableau autosaves because Tableau keeps crashing. I hope to find a fix, or at least put it in a separate folder.

DATA SOURCES

1. Election results were obtained from Harvard Dataverse: https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/VOQCHQ
2. Education and unemployment data were obtained from the Economic Research Service, a sub-agency of the Department of Agriculture: https://www.ers.usda.gov/ (For education, estimates were only available for 2000, 2008-2012, and 2019-2023. I assigned 2008-12 values to 2010, and 2019-23 values to 2021, then interpolated estimates for years between.)
3. Population data were obtained from the United States Census Bureau. 2000, 2010, and 2020 data were taken from official Census figures, and other years use Census Bureau estimates: https://www.census.gov/

I intend to add more statistics such as income, life expectancy, demographics, and English profiency (not a final list).

Finding and cleaning data has been a project of its own.

I'd also like to include other factors such as the approval rating of the incumbent President (and their party), polling averages, and social media sentiment, but I'm unsure if such data are available at the county level; if not, how I might interpolate them; and if county interpolation is not desirable, whether it makes sense to simply apply nationwide or statewide figures to every county.

I've created a Tableau workbook to visualize certain statistics. It will be updated as more are added.

I have been working with machine learning models. My basic methodology is as follows:
1. Model tuning, with techniques such as GridSearchCV, K-Folds, and feature selection. Tried a GridSearch already, couldn't get it to run in 30 minutes. Will try again soon.
2. Set two targets: Republican Percentage and Democratic Percentage. This helps to smooth out vote swings that only affect one party (as notably occurred in Utah in 2016).
3. One by one, predict vote shares for both parties in each election, using the other elections as training.
4. Gather statistics such as Absolute Error, Median Error, Z-Score and more for each party for each county.
5. Take the two-party average (2PA) for each of these values for each county. For example, in a county where Democrats are overestimated by 1 point and Republicans are overestimated by 3, the 2PA Absolute Error will be 2. The 2PA averages will be regarded as the ultimate measures of performance.
7. Aggregate these stats across county, state, year, and state/year combination. In doing so, we also will assign votes to the parties in each county by multiplying actual total votes by the party's predicted percentage. This will allow us to aggregate statewide vote totals and assign electoral votes accordingly.

Once I have maximally optimized a model, I will attempt to simulate the 2024 election using the full 2000-2020 dataset as training.
My primitive version of a 2024 model predicted a Democratic victory, the opposite of the real-life result.

If I can add real 2024 results to my dataset, I will then move on to predicting 2028.

STATES AND COUNTIES EXCLUDED:
1. ALASKA - Alaska does not have counties in the traditional sense, but is instead broken into "organized" and "unorganized" boroughs. This would be fine except some data sources do not use these, but instead use "districts" with entirely different boundaries and names. Until or unless I can find a method to reconcile these differing statistical areas, I will have to exclude Alaska from the dataset.
2. COLORADO - Broomfield County did not exist until 2001. The city of Broomfield was previously spread across four counties before being granted the status of Consolidated City/County for more efficient self-governance. Thus, it is missing from most 2000 data. I am considering interpolating values based on real ones, but this would also artifically inflate Colorado's statewide vote totals for 2000 as its votes were spread across the other counties, unless *their* 2000 values could be interpolated downward to estimate figures if Broomfield had never been a part of them. For now, I have excluded it.
3. HAWAII - Kalawao County, though named as such, has no administrative functions of its own. It was established as a leprosy quarantine settlement. To this day it is inhabited only by the descendants of the initial patients, land access is only by mule trail, public visitation requires official permission, and as of 2020 its population was just 82. It was entirely absent from some of the datasets, so I have excluded it.


DATA CLEANED, BUT POSSIBLE COMPLICATIONS:
1. CONNECTICUT - In Connecticut, counties ceased most administrative functions in 1960, but remained for statistical purposes until being replaced by new "planning areas" with different boundaries in 2022. As all elections in the dataset took place prior to this change, data integrity won't be an issue for analysis. However, visualization tools such as Tableau may not know how to reconcile the old counties with the new planning areas. This will become a larger problem when/if I add 2024 data.
2. SOUTH DAKOTA - Oglala Lakota County was known as Shannon County until 2015. Entries with the old name and FIPS Code have been updated to use the new ones. For the purposes of this project, this will not cause any problems, but complications could arise if my compiled dataset is compared to older ones.
3. KANSAS CITY/JACKSON COUNTY - For some reason, Kansas City, Missouri is listed separately despite not having county-equivalent status. After 2000, its numbers are split off from Jackson County, in which it is contained. Jackson County's post-2000, statistics, in turn, only reflect parts of the county outside of Kansas City. This doesn't seem to have been done for any other non-county entity. I manually recombined the raw numbers and recalculated percentages, so they now reflect the entirety of Jackson County.
4. VIRGINIA - In Virginia, dozens of "independent cities" exist which are treated as county equivalents, and many of these share names with counties despite being separate entities. FIPS Codes have ensured data integrity, and independent cities have been re-labeled as such for clarity. Two former independent cities have given up such status since the beginning of this dataset - Clifton Forge, which merged into Allegheny County in 2001; and Bedford City, which merged into Bedford County in 2013. I have excluded them for now as I believe models would be confused by missing data.
7. NON-COUNTY VOTES - From 2012 onward, Connecticut, Maine, and Rhode Island have tabulated special types of votes (overseas, write-ins etc) which are not assigned to any particular county. I allocated these votes proportionally to each county based on the percentage of statewide votes it cast, then assigned votes proportionally within the counties by each party's percentage of the vote.



OTHER DATA COMPLICATIONS:
1. Certain states allow electoral fusion, in which a candidate can run on multiple ballot lines and receive combined credit for all. For instance, in New York in 2020, voters could vote for Joe Biden either as the Democratic nominee *or* as the Working Families nominee. These alternative ballot lines may be valuable in signaling protest votes, but are difficult to simulate as the minor parties don't participate in every election. However, merely reassigning such votes to the main party would present a data integrity challenge. I may have to alter my data structure to account for this.
2. How to handle the Green and Libertarian Parties? The election dataset provides separate entries for these parties, but they have not been on the ballot in every state in every election since 2000. On one hand, the absence of a minor party could cause the model to correctly assign higher vote shares to the major ones, but I also fear the model could make incorrect political inferences about why votes dropped to 0. There are other minor parties which appear on ballots even less commonly - should each of them have their own columns as well? For now, they have been rolled into a generic "Other" category for vote totals and percentages.
3. Maine and Nebraska allocate electoral votes differently, giving two statewide and one for each of its Congressional districts, which do not precisely correspond with county lines. I will have to devise a method to reconcile these boundaries. Until then, I am using the winner-take-all system of other states.
4. Some counties are showing suspiciously high third-party vote shares in certain years. Need to investigate and ensure no errors are occurring during cleaning.



OTHER QUIRKS:
You may notice that Loving County, TX, frequently has more votes than residents, causing turnout above 100%. This is a well-documented phenomenon which has resulted in lawsuits and is not a data problem.


IDEAS AND TO DO LIST
1. Need more visualizations
2. IDEA - After model results are compiled, print the results with each year excluded
3. IDEA - Interpolate data between the election years and simulate them as if an election took place.
4. IDEA - SMOTER (regression equivalent of SMOTE, in which synthetic data points are generated to strengthen the relationship between features)
5. Color code columns for parties. Tried a primitive version of this, didn't work. Background color, or just text?
6. Rename percentage columns to include the % symbol
7. Potentially rename the "Bachelor's degree or higher" to remove the apostrophe, as it is causing quote escape problems
8. In the Tableau workbook, I may need to alter how I calculate the mean values. Currently I am taking a mean of county values, which may not be fair as counties are not equally populated. Instead, I may need to do something like (SUM Population) / (SUM(Population by county * High school graduate))
9. NLP?
10. IDEA - Add to the main dataframe a new column showing for each value the change from the previous year.
11. IDEA - Adjust error by state relative to its margin. If a party is predicted to win a state by 30 points but only win it by 20, that may be less significant than a 5-point error that changes the outcome of the state.
12. IDEA - Try classification as well. I've built basic binaries by comparing predicted with actual values, but I am curious how a Random Forest Classifier could handle this data. Would it be worth trying to remove each party's vote totals and percentages, and train only on the winner from the other years?
13. IDEA - Adjust errors for total votes? So that larger counties would be punished more?
14. Distribution curve visualization of "correct" predictions - some of which had large errors in vote share despite choosing the correct party.






