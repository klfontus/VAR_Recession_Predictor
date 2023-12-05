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

[Link](https://www.dropbox.com/scl/fi/umx2vjq4a1ppstb2kh1yv/merged_data_mscore_wab20231031125447.csv?rlkey=jmlquopgzrbf9ggtgenja0lmh&dl=0) to final data set used in models. 


## Jupyter Notebooks For Cleaning And Feature Engineering.
There are jupyter notebooks in the 'Code' folder that we used to transform raw time series data into the clean, quarterly data used to build our final models: 

- **GDP, PPI, and Yield Curve**: GDP_PPI_YC_merging.ipynb
- **S&P 500 returns**: clean_index_data.ipynb
- **Dow Jones returns**:  clean_INDU_data.ipynb
- **Weighted average beta and market cap**: weighted_average_beta_explorer.ipynb
- **M-Score**: m_score.ipynb,  Weighted_M_Score.ipynb, M_explore.ipynb

Quarterly M-Score per company had to be calculated from various indices from an extremely messy data base, after which quarterly arithmetic averages and averages weighted by market capitalization were calculated. Average beta values for thousands of companies were also weighted by market capitalization. PPI, Yield Curve, S&P 500 returns, Dow Jones returns, average betas, and weighted average betas also had their trend, velocity, and acceleration values calculated by quarter to see if these new features provide any insight on changes in GDP.

## Model Building Process.
All of our VAR models can be found in the model1.ipynb notebook in the Code directory. These are the general steps we took to build our models:
1. Importing the 2 versions of our final datasets as pandas dataframes: **merged_data20231031125447.csv** (up to 2023) and 
**merged_data_mscore_wab20231031125447.csv** (up to 2013 and includes M-Score and Weighted Average Beta)
2. Creating a series of plots to check for errors in previous cleaning and feature engineering steps.
3. Running the Augmented Dickey-Fuller Test to check if our variables were stationary. VAR models require staionary data, which we achieved after differencing the data once.
4. Running Granger's Causality Test on the differenced data to see which variables 'lag' GDP (which variables have 'predictive causality' on GDP). We decided to start with **merged_data_mscore_wab20231031125447.csv** so we could study the effect of including M-Score and Weighted average beta. Granger causality tests with this data set show 16 variables which lag GDP at a 95% confidence level. These 16 variables were all included in our first VAR model.
5. Finding the optimal order to build our VAR model: Models with order 1-12 were built. Setting order = 5 gave models with AIC values near the first local minimum, our literature suggests that M-Score lags recession by "5 to 8 quarters" so we used order = 5 for all of our models.
6. We used VIF and Durbin Watson statistics to decide which of our 16 original variables to remove to make better models. We also did a cointegration to test of our 16 differenced variables with respect to each other and GDP. We found that many of our variables of interest are cointegrated, therefore a VECM model might have been a better choice. But we did not have time to build any VECM models.
7. Splitting data into train and test sets: We started by trying to forecast the 2008 great recession using models with different sets of variables, then we moved on to the (1981-1982), (1990-1991), 2001, and 2020 recessions. Finally we forecasted GDP beyond the 2023 dataset to see what the future holds for the US economy.
8. Models were visually compared and in many cases MSE and correlation coefficients were calculated to understand accuracy of forecasts.
9. Final Visuallizations were done in a Tableau dashboard.


