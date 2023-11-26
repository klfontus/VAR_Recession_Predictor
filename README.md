# Team-36
 Team 36's group project GitHub repository for MGT 6203 (Canvas) Fall of 2023 semester.

## Background
The purpose of this project is to build a recession predictor by way of predicting GDP using both traditional and modern economic variables. The yield curve has been the de-facto indicator for looming economic troubles, and economists are signaling that it is not working as well in modern times. We built a Vector Auto-Regressive Model after testing the effectiveness of a wide spread of historical variables in predicting GDP.

## Data Sources
The features used in our VAR models are time series data from a variety of sources:
- **Macro Economic Variables**: Real GDP (our "response" variable), Producer Price Index, and Yield Curve are all traditional indicators of recessions. Data for these 3 features was downloaded from the [Federal Reserve Bank of St. Louis' website](https://fred.stlouisfed.org/) 
- **Stock Market Returns**: Market returns are also a classic variable used to forecast recessions. We used S&P 500 and Dow Jones Industrial Average historical data in our models that came from Bloomberg databases queried through the [Bloomberg Anywhere App](https://bba.bloomberg.net/) 
- **Financial Statements**: We used Wharton Research Data Services to query databases containing historical financial data from thousands of publicly traded companies. Data on [beta values](https://wrds-www.wharton.upenn.edu/login/?next=/data-dictionary/contrib_general/), and [market capitalization](https://wrds-www.wharton.upenn.edu/login/?next=/data-dictionary/crsp_q_stock/) was used to engineer a "weighted average beta" feature examined in our models, and [Compustat Snapshot data](https://wrds-www.wharton.upenn.edu/login/?next=/data-dictionary/compsamp_snapshot/wrds_csq_unrestated/) was used to calculate Beneish M-Score (a more modern variable which has attracted recent interest by economists looking for better methods to forecast recessions).

## What's In The Data Folder? 
The 'Data' folder in this repository contains 2 types of csv data files:
1) Raw csv data for features extracted from the sources listed above.
2) Processed csv data with added features that has been explored and cleaned using a series of jupyter notebooks (described below). 
3) **merged_data20231031125447.csv** : (1976 Q3 to 2023 Q1) Contains most features in quarterly date format and runs all the way up to the first quarter of 2023. But it does not include m-score or weighted average beta.
4) **merged_data_mscore_wab20231031125447.csv** : (1977 Q2 2013 Q2) Contains all data in quarterly date format including M-score and weighted average beta. This is the dataset used to make all of our final forecasts.



## Jupyter Notebooks For Cleaning, Exploring, And Feature Engineering.
There are Python Notebooks in the Code folder that hold the transformations from the raw data to the clean, quarterly data used to build our final model. Quarterly M-Score per company had to be calculated from various indices from an extremely messy data base, after which quarterly arithmetic averages and averages weighted by market capitalization were calculated. Market betas were averaged over quarters and their weighted averages were also found. The PPI, Yield Curve, S&P 500 returns, Dow Jones returns, average betas, and weighted average betas also had their trend, velocity, and acceleration values calculated by quarter to see if they provide any insight on changes in GDP.

- **GDP**: GDP_PPI_YC_merging.ipynb
- **PPI**: GDP_PPI_YC_merging.ipynb
- **Yield Curve**: GDP_PPI_YC_merging.ipynb
- **S&P 500 returns**: clean_index_data.ipynb
- **Dow Jones**:  clean_INDU_data.ipynb
- **Market Capitalization**: weighted_average_beta_explorer.ipynb
- **Market Beta**: weighted_average_beta_explorer.ipynb
- **M-Score**: m_score.ipynb,  Weighted_M_Score.ipynb, M_explore.ipynb
## 
There are csv files in the Data folder that house the raw data for each of these variables. The following are the csv files that house the transformed variables whose end of quarter values make up our final data set:

- **GDP**: GDP_PPI_YC_20231025203618.csv
- **PPI**: GDP_PPI_YC_20231025203618.csv
- **Yield Curve**: GDP_PPI_YC_20231025203618.csv
- **S&P 500 returns**: clean_index_data202310251597.csv
- **Dow Jones**:  clean_INDU_data202310308286.csv
- **Market Capitalization**: MCAP_WAB20231031123947.csv
- **Market Beta**: MCAP_WAB20231031123947.csv
- **M-Score**: final_M.csv

In the end, two "final" data sets were created. The file merged_data20231031125447.csv holds data that runs from 1976 to 2023. Our M-Score and market beta data only goes up to 2013, so we created a truncated version,  merged_data_mscore_wab20231031125447.csv, so we could explore all the variables at our disposal.
 



The final data set was created in final_merge.ipynb. All other notebooks have miscellaneous tests and explorations of the data.

## Model
The creation and testing of our final model can be found in the model1.ipynb notebook. The steps taken were:
1. Loading in the two data sets.
2. Creating a series of visualzations for each variable type.
3. Running the Augmented Dickey-Fuller Test to check if our variables were stationary. VAR models require staionary data, which we achieved after differencing the data once.
4. Running Granger's Causality Test on the differenced data to see which variables lag GDP (aka which variables could cause GDP). The unpredictability of the 2020 Recession indicated that the data set that runs into 2023 has few variables that lag GDP. Therefore we decided to continue on with the data set that runs until 2013, since it has more total variables anyways, with 16 total being selected.
5. Finding the optimal order to build our VAR model. Models with order 1-12 were built, with order=3 having the AIC value with the first local minima.
6. Make a training split of the data, with it ending at the end of 2007, right before the beginning of the Great Recession in 2008. The rest of the data was used to test the model.
7. Create the model (model1) with the differenced train data, then make a forecast with the test data. The latter is a multi-step process, where you make a forecast with the differenced data, and then it must be inverted and un-differenced. Then a visualization was used to compare the test data and the forecasted data. It shows that our results were not promising so we explored the variables some more.
8. We did a cointegration to test of our 16 differenced variables are cointegrated with GDP, with the null hypothesis being that there is a long running, statistically significant relationship between variables. All the tests were false, meaning we can proceed with the VAR model.
9. A VIF test revealed the presence of multi-collinearity in the variables. A Durbin-Watson test revealed that there are serially correlated residuals with some variables after creating the initial model. Analyzing the results of both tests together revealed which variables to keep for our next model.
10. Another model (model2) was created with 7 remaining variables, with the resulting forecast being a lot closer to the test GDP values.
11. Durbin Watson and VIF tests revealed a smaller chance of correlated variables.
12. Model evaluation was done to compare the models.
13. Final model was built with a smaller train set to see which recessions we can predict.


