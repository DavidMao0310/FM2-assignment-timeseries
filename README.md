# FM2-assignment-timeseries 

Here is the [Reference](http://web.vu.lt/mif/a.buteikis/wp-content/uploads/2019/02/02_StationaryTS_Python.html)

**We use ARIMA model to predict the Close Price of US T-Bond futures**
----
First we check stationary of the data, if not, then we calculate its difference and recheck it until it tends to stationary.

Notice that the P-value of ADF test indicates there is enough evidence to say it is stationary.

Use Ljung-Box test to check whether it is a White Noise Process.

Auto-select the order of ARIMA then build the model.

Then we fit the model and do the residuals analysis. From the plot we can see the model fit well.

***Packages***

```
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
sns.set_style('whitegrid')
from pandas.plotting import lag_plot
from statsmodels.tsa.stattools import adfuller
from statsmodels.stats.diagnostic import acorr_ljungbox
from statsmodels.graphics.tsaplots import plot_acf
from statsmodels.graphics.tsaplots import plot_pacf
from statsmodels.graphics.gofplots import qqplot
from statsmodels.tsa.stattools import arma_order_select_ic
from statsmodels.tsa.arima_model import ARIMA
```
***Check stationary***
```
def autocorr(x):
    plt.figure()
    lag_plot(x, lag=1, alpha=0.2, c='cornflowerblue')
    plt.title('Autocorrelation plot with lag = 1')
    plt.show()

def look(x):
    plt.plot(x)
    plt.title('Close Price')
    plt.show()
```
***ADF test***
```
def ADF(x):
    t = adfuller(x)
    output = pd.DataFrame(index=['Test Statistic Value', "p-value", "Lags Used", "Number of Observations Used", "Critical Value(1%)",
               "Critical Value(5%)", "Critical Value(10%)"], columns=['value'])
    output['value']['Test Statistic Value'] = t[0]
    output['value']['p-value'] = t[1]
    output['value']['Lags Used'] = t[2]
    output['value']['Number of Observations Used'] = t[3]
    output['value']['Critical Value(1%)'] = t[4]['1%']
    output['value']['Critical Value(5%)'] = t[4]['5%']
    output['value']['Critical Value(10%)'] = t[4]['10%']
    print(output)
```    
We need bulid stationary series from ADF test

***Difference data***
```
def diff1(x):
    t = x.diff(1).dropna()
    return t
```    
***White Noise test***
```
def WN(x):
    wn_pvalue = acorr_ljungbox(x, lags=1, return_df=True)
    print("White Noise Test", wn_pvalue)
```
***model buliding***
```
def look_order(t):
    plot_acf(t, lags=20)
    plot_pacf(t, lags=20)
    plt.show()

def auto_order(x,ar,ma):   #max_ar, max_ma
    z = arma_order_select_ic(x, max_ar=ar, max_ma=ma, ic='aic')
    print(z)

def arima_buildfit(x, order):
    model = ARIMA(x, order=order)
    t = model.fit()
    return t
```    
***Analysis residuals***
```
def tsdisplay(y, figsize = (14, 8), lags = 10):
    tmp_data = pd.Series(y)
    fig = plt.figure(figsize = figsize)
    #Plot the time series
    tmp_data.plot(ax = fig.add_subplot(311), title = "Time Series of Residuals", legend = False, c='seagreen')
    #Plot the ACF:
    plot_acf(tmp_data, lags = lags, zero = False, ax = fig.add_subplot(323))
    plt.xticks(np.arange(1,  lags + 1, 1.0))
    #Plot the PACF:
    plot_pacf(tmp_data, lags = lags, zero = False, ax = fig.add_subplot(324))
    plt.xticks(np.arange(1,  lags + 1, 1.0))
    #Plot the QQ plot of the data:
    qqplot(tmp_data, line='s', ax = fig.add_subplot(325), c='cornflowerblue')
    plt.title("QQ Plot")
    #Plot the residual histogram:
    fig.add_subplot(326).hist(tmp_data, bins = 40, density=True, range=[-5,5])
    plt.title("Histogram")
    #Fix the layout of the plots:
    plt.tight_layout()
    plt.show()
```

***Prediction***
```
def predict(fit, oridata):
    fig = plt.figure(figsize=(12, 8))
    data_size = oridata.shape[0]
    fit.plot_predict(1, data_size-1 , ax=fig.add_subplot(211), alpha=0.05, plot_insample=True)
    plt.legend().remove()
    fit.plot_predict(data_size-30, data_size+20, ax=fig.add_subplot(212))
    plt.tight_layout()
    plt.show()
```
`plot_predict` is a function in `statsmodels.tsa.arima_model` which can be used to predict the future values.









