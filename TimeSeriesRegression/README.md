# TIME SERIES REGRESSION TEMPLATE (XGBOOST VS. ARIMA)


## Specification
. Template should allow user to input time series data with firstly named ‘Date’ column and a second target Y column. 

. Template should take data, process into sliding window technique for XGBoost, perform data split specified by user, 
  fit XGBoost & ARIMA model on training data to give a mean absolute error score. 
  
. Once done the template will forecast a specified number of steps defined by the user to give n predictions from the
  XGBoost and ARIMA models outputting these predictions rounded to the nearest whole number back to separate csv files
  subsequently named, ‘xgbPredictions.csv’ and ‘arimaPredictions.csv’. 
  
. At the end of the template both MAE scores will be printed for evaluation, so the user has a clear visual on what model
  fits their data best for regression purposes.
