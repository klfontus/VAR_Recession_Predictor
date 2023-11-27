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
Raw csv data for features extracted from the sources listed above, and
Processed csv data with added features that has been explored and cleaned using a series of jupyter notebooks. Many of the processed data sets include new features called 'trend' which captures the moving avearge trend of the raw time series data assuming quarterly seasonality. There are also 'velocity' and 'acceleration' features which are the 1st and 2nd derivative of their 'trend' feature respectively.

The following are the csv files that house the transformed variables whose end of quarter values make up our final data set:

- **GDP, PPI, and Yield Curve**: GDP_PPI_YC_20231025203618.csv
- **S&P 500 returns**: clean_index_data202310251597.csv
- **Dow Jones returns**:  clean_INDU_data202310308286.csv
- **Weighted Average Beta and Market Capitalization**: MCAP_WAB20231031123947.csv
- **M-Score**: final_M.csv

We ended up with 2 versions of our final data set:

- **merged_data20231031125447.csv** : (1976 Q3 to 2023 Q1) Contains most features in quarterly date format and runs all the way up to the first quarter of 2023. But it does not include m-score or weighted average beta.
- **merged_data_mscore_wab20231031125447.csv** : (1977 Q2 2013 Q2) Contains all data in quarterly date format including M-score and weighted average beta. This is the dataset used to make all of our final forecasts.


## Jupyter Notebooks For Cleaning And Feature Engineering.
There are jupyter notebooks in the 'Code' folder that we used to transform raw time series data into the clean, quarterly data used to build our final models: 

- **GDP, PPI, and Yield Curve**: GDP_PPI_YC_merging.ipynb
- **S&P 500 returns**: clean_index_data.ipynb
- **Dow Jones returns**:  clean_INDU_data.ipynb
- **Weighted average beta and market cap**: weighted_average_beta_explorer.ipynb
- **M-Score**: m_score.ipynb,  Weighted_M_Score.ipynb, M_explore.ipynb

Quarterly M-Score per company had to be calculated from various indices from an extremely messy data base, after which quarterly arithmetic averages and averages weighted by market capitalization were calculated. Average beta values for thousands of companies were also weighted by market capitalization. PPI, Yield Curve, S&P 500 returns, Dow Jones returns, average betas, and weighted average betas also had their trend, velocity, and acceleration values calculated by quarter to see if these new features provide any insight on changes in GDP.

## Model Building Process.
The creation and testing of our first set of VAR models can be found in the model1.ipynb notebook. The following steps were taken to bulid model1:
1. Importing the 2 versions of our final datasets (**merged_data20231031125447.csv** and 
**merged_data_mscore_wab20231031125447.csv**) as pandas dataframes.
2. Creating a series of plots to check for errors in previous cleaning and feature engineering.
3. Running the Augmented Dickey-Fuller Test to check if our variables were stationary. VAR models require staionary data, which we achieved after differencing the data once.
4. Running Granger's Causality Test on the differenced data to see which variables 'lag' GDP (which variables have 'predictive causality' on GDP). We found that data set **merged_data20231031125447.csv**, which goes to 2023 and includes the 2020 mini recession, has few variables that lag GDP. Therefore we decided to continue on with **merged_data_mscore_wab20231031125447.csv** which runs until 2013 and includes M-Score and Weighted average beta. Granger causality tests with this data set show 16 variables which lag GDP at a 95% confidence level. These 16 variables were all included in our first VAR model.
5. Finding the optimal order to build our VAR model: Models with order 1-12 were built, with order=3 having the AIC value with the first local minima.
6. Make a training split of the data, with it ending at the end of 2007, right before the beginning of the Great Recession in 2008. The rest of the data was used to test the model.
7. Creating the model (model1) with the differenced training data, then making a forecast with the test data. Forecasts are made using the differenced data but the results must be 'un-differenced' to create a final real forecast. Plots were used to visually compare test data with forecast data, and . It shows that our results were not promising so we explored the variables some more.
8. We did a cointegration to test of our 16 differenced variables are cointegrated with GDP, with the null hypothesis being that there is a long running, statistically significant relationship between variables. All the tests were false, meaning we can proceed with the VAR model.
9. A VIF test revealed the presence of multi-collinearity in the variables. A Durbin-Watson test revealed that there are serially correlated residuals with some variables after creating the initial model. Analyzing the results of both tests together revealed which variables to keep for our next model.
10. Another model (model2) was created with 7 remaining variables, with the resulting forecast being a lot closer to the test GDP values.
11. Durbin Watson and VIF tests revealed a smaller chance of correlated variables.
12. Model evaluation was done to compare the models.
13. Final model was built with a smaller train set to see which recessions we can predict.


