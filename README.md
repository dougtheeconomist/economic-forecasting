# economic-forecasting
My home on Git for predictive modeling of the U.S. economy.

At this time the up-to-date information needed to create the most accurate forecasts of the near future is not as available as it otherwise has been in the past. Therefore the present status of this exercise is as a documentation for the methodology behind my forecasting efforts rather than a useful report of the coming months. My hope is that once our workforce can once again move into offices and collaborate more freely, without the constraint of distance work, that more current information will once again be made publicly available by official sources, from which I can utilize the techniques outlined herein to provide a more useful analysis. At such a time, that analysis will either live here, or another repo will be established for that purpose, and a link will be provided in place of this statement. 

## Project breakdown

As an applied economist one of my long standing on and off projects has been to forecast the short-run trajectory of the U.S. economy, as measured by gross domestic product. To accomplish this I have used an ensemble of two vector auto regression or var models. This is a linear based technique that involves taking a number of related factors and using them to form predictions of each other once each variable in the model has been predicted for one period out, these predictions are then treated as the most recent period of data to repeat the process to generate predictions for the second period into the future, and so on and so forth until we have predicted as far into the future as we intend to. Traditionally I have done this using the statistical analysis program Stata, however having now transitioned to using Python as my primary analytical environment with my shift into the field of data science, I wanted to recreate this process in Python. 

There are several advantages to this; first I don’t have to switch programs from Python to Stata if I want to utilize this forecasting technique – I can continue to make Python my one stop shop for working with data. Second, Python is much more flexible than Stata as it is a programming language in and of itself and isn’t limited to the real of data analysis. This allows me to much more easily write custom code to automate complex processes. The final reason of course is simply because I can, so why not? 

There are several major steps involved in the forecasting process I use in this project and I’ll discuss my methods for each of them, as well as ways in which this process changed with the utilization of Python here. These general steps are; model specification, error measurement and tuning, Calibration of forecast intervals, and graphing/reporting results.

## Data Pipeline

In order to generate a forecast of the future of the economy I need data on past levels of economic output as well as correlated variables that help give indications of the economies movement and the varying components that are measured and aggregated when calculating gross domestic product. I use two official sources for the various data series that I utilize; the St. Louis Federal Reserve Economic Database commonly abbreviated as FRED, and the U.S. Energy Information Administration database. Working in Python allows me to access each of these databases via api call and then parse the relevant data into new rows in the pandas dataframe object where previously collected data is stored. From there I conduct the minimal data cleaning required and perform my feature engineering. This dataframe is then fed into the two models I utilize via the Statsmodels library for modeling, tuning and calibration. Once this process is complete the output comes in the form of NumPy arrays which then become inputs to the Matplotlib graphing library for easy visual interpretation. 

## Modeling

This step was largely done previously and so didn’t take any effort this time around as I didn’t make significant changes to the model specifications that I used when conducting this analysis in the past using Stata. I will explain the two methods that I use here, for those who have not read my past blog posts on the subject. 

Essentially there are two different broad methods for forecasting a countries GDP. The first is to forecast it outright as a single variable based on past values of GDP itself as well as other factors
that tend to be good indicators. The second is to forecast the different elements of an economy that are aggregated in the calculation of a country’s GDP. These pieces are consumer spending, government spending, investment spending, and net exports. Instead of choosing one or the other, I have long made it my practice to generate a forecast using each of these methods and then combining the results of the two models by way of weighted averaging to form my final output. 

Between the two models that I use, I incorporate data on the GDP variables in question as well as measures of overall economic health such as employment, industrial production, energy utilization, the housing market and indicators of investment health. Each model has a different mix of explanatory variables which further serves to create a bit of distinction between their outputs. This is useful preventing an overfitting problem as can be done with a single model, and while there certainly aren’t enough outputs being averaged to get the same benefit in this way that a random forest model achieves over a single decision tree, it is still more useful than relying on a single model specification. 

In recreating these models in Python I use the Pandas library to store the data and Python’s Statsmodels library for model creation as it has premade functionality to build VAR models. The only real difference in the specification of each model between Stata and Python is the number of lagged values of previous data that the models consider. While I used to use a full half year of data in building future predictions, when using the built in grid searching functionality of statsmodels to find the ideal model fit using the Bayesian Information Criterion, I now find that using a single step into the past achieves more accurate results. This is a surprise to me as I essentially used a less automated method of finding the optimal lookback length in Stata using the same criterion, however it may be that the amount of updated data I have is affecting this hyperparameter. 

## Error Measurement and Tuning

In order to measure the model error and calibration, I backtest the model for the previous hundred periods, in this case months, and record the models output at each point for comparison to the actual historic data series. This process is essentially the reverse of the walk forward method of time series model validation; instead of adding one row of new data at a time, I iteratively subtract the most recent row of data and then collect results run on the increasingly smaller set of data. It should be noted that if I were using a modeling process where the model was saved after training to run a trained model to evaluate new data, I would want to run this validation in standard order; that is train the model on old data and then consecutively add and measure on newer data. Since the VAR method instantiates a model from scratch each time it is called and does not "remember" training like more standard Machine Learning algorithms, this is not a necessity in this case. 

In measuring error when using my models to forecast historical values for comparison, I use the MAPE metric; mean absolute percentage error. I use this error metric as my decision criteria for how I weight my models when ensembling together. Essentially I do a grid search of the different possibilities from 100% model one and 0% model two, all the way to the opposite extreme, moving at 1% intervals. I conduct this grid search for each of the six monthly time periods out from the present that my model creates predictions for, each time using 100 forecasts of historic data for comparison. What I find is that as I forecast further out from the present period, the optimal mix of the two models changes. My GDP as a whole model is more heavily favored in most cases, especially in the first three months ahead. 

Here is one area in which my methodology has improved with the move to Python from Stata; whereas I did ensemble the two VAR models together and report this result before, I did not optimize the weights of each model via grid search in the past. Before I had weighted each model according to it’s reported error metric, again using MAPE, but had not systematically checked the error metrics of each different combination of weights. This would have been possible to do in Stata if I had thought to do so (optimization by grid search was not a method that I had learned in my studies as an economist), but it would have been much more difficult to set up in Stata to properly record the results. 

## Interval Calibration

The other way that Python has made this project much easier to complete is in allowing for heavier automation of the process of identifying whether or not my models interval forecasts are properly calibrated. By this I mean, when the model predicts that the actual GDP value has a 95% chance of falling within the upper and lower intervals reported, does the actual value land within that range 95% of the time when tested against historical data? If the range that my model is 95% confident in only captures the actual value 80% of the time, this would be a problem. However, by measuring for this miss-calibration, the model output can then be adjusted to make the interval either wider or narrower to bring the expectation in line with reality. Alternatively, another approach would simply be to leave the size of the interval as is, and report the actual percentage of time that this interval captures the actual value of the forecasted variable.

 Whereas using Stata this was not a fully automated process, in Python, once I have written the code to capture and compare these intervals for past values, I can completely automate the entire process, and even update the models current output to reflect this methods findings automatically. 

## Model Output

I have coded my models in Python for options to either return forecasting outputs in terms of actual values of GDP or in percentage change format; another handy feature to modeling in Python. 

While again this information is based on old data and therefore doesn't actually tell us anything about the future at this point, the output in both formats can be viewed in the following graphs. 

![percentage_change_graph](/images/pc_forecast_graph.png)

![real_value_graph](/images/real_forecast_graph.png)

For further context the chart below shows the predictions for each month alongside the historic averages for each month as well as the same month from the previous year. 

![monthly_breakdown](/images/monthly_growth_perc.png)


## Project Future

There are two directions I intend to go with this material moving forward. The first obviously is to use the models and processes showcased here to generate economic forecasts in the future, when I can obtain recent enough data to do so in a way that is actually relevant. This may involve migrating to the use of alternative data sources with more frequent updates if possible.

The second step to take moving forward starts by making the code I use for model calibration more flexible. There are things hardcoded into the functions I have built to do this that should ideally take arguments instead; chiefly the number of periods into the future to predict. The project predicts six months into the future, but that might not be enough for other endeavors in the future, and by taking the number of periods to calibrate as an argument I can directly import this code into other applications, or make it more useful for others.

Once this functionality is added, it is then my intention to create a calibration class object so that this functionality could be imported and accessed as a library in any future project where it may be needed. It is coded is such a way that it is not limited to Vector Auto Regressions but will be applicable to other methods of generating forecasts as well such as ARIMA or Deep Learning based models. 
