---
layout: single
title: "Dynamic Factor Modeling for CPI data across seven countries"
---

```python
%matplotlib inline

import pandas as pd
import statsmodels.api as sm
import matplotlib.pyplot as plt
import numpy as np
np.set_printoptions(precision=4, suppress=True, linewidth=120)
from pandas_datareader.data import DataReader

```


```python
import pandas as pd

# Assuming 'ind.csv' is the path to your CSV file
df = pd.read_csv('cpi.csv', parse_dates=['Date'])

# Pivot the DataFrame to wide format
df_pivoted = df.pivot(index='Date', columns='Country', values='Value')

# Optional: Handle missing values, for example, fill with NaN or 0
# df_pivoted.fillna(0, inplace=True)

# Now df_pivoted is in the format you need for your DFM model
print(df_pivoted)

```

    Country           CAN        DEU        FRA         GBR         ITA  \
    Date                                                                  
    1968-01-01   14.61680   28.05229   12.86326    7.296107    5.257903   
    1968-02-01   14.61680   28.11753   12.87393    7.332107    5.257903   
    1968-03-01   14.69581   28.11753   12.88461    7.356108    5.262920   
    1968-04-01   14.69581   28.11753   12.91663    7.488110    5.272954   
    1968-05-01   14.69581   28.11753   12.95933    7.494110    5.277971   
    ...               ...        ...        ...         ...         ...   
    2023-07-01  124.91440  123.45660  117.71000  129.000000  119.700000   
    2023-08-01  125.38850  123.87840  118.89000  129.400000  120.100000   
    2023-09-01  125.23040  124.19460  118.26000  130.100000  120.300000   
    2023-10-01  125.30950  124.19460  118.43000  130.200000  120.100000   
    2023-11-01  125.46750  123.66750  118.23000  130.000000  119.500000   
    
    Country           JPN        USA  
    Date                              
    1968-01-01   27.67744   14.38715  
    1968-02-01   27.79937   14.42935  
    1968-03-01   27.84814   14.47154  
    1968-04-01   27.92130   14.51373  
    1968-05-01   27.92130   14.55592  
    ...               ...        ...  
    2023-07-01  107.61010  128.97430  
    2023-08-01  107.81370  129.53750  
    2023-09-01  108.11910  129.85950  
    2023-10-01  109.03540  129.80970  
    2023-11-01  108.83180  129.54810  
    
    [671 rows x 7 columns]
    


```python
# Assuming df_pivoted contains all the countries' data in separate columns
cpi_us = df_pivoted[['USA']].rename(columns={'USA': 'cpi_us'})
cpi_gm = df_pivoted[['DEU']].rename(columns={'DEU': 'cpi_gm'})
cpi_uk = df_pivoted[['GBR']].rename(columns={'GBR': 'cpi_uk'})
cpi_fr = df_pivoted[['FRA']].rename(columns={'FRA': 'cpi_fr'})
cpi_it = df_pivoted[['ITA']].rename(columns={'ITA': 'cpi_it'})
cpi_ca = df_pivoted[['CAN']].rename(columns={'CAN': 'cpi_ca'})
cpi_jp = df_pivoted[['JPN']].rename(columns={'JPN': 'cpi_jp'})

# Concatenate all cpiividual country DataFrames into a single DataFrame
dta = pd.concat((cpi_us, cpi_gm, cpi_uk, cpi_fr, cpi_it, cpi_ca, cpi_jp), axis=1)

# Optionally, set the frequency of the DataFrame's cpiex if it's a time series
dta.index.freq = dta.index.inferred_freq

# Plotting each country's data in separate subplots
dta.plot(subplots=True, layout=(4, 2), figsize=(15, 10))
```




    array([[<AxesSubplot:xlabel='Date'>, <AxesSubplot:xlabel='Date'>],
           [<AxesSubplot:xlabel='Date'>, <AxesSubplot:xlabel='Date'>],
           [<AxesSubplot:xlabel='Date'>, <AxesSubplot:xlabel='Date'>],
           [<AxesSubplot:xlabel='Date'>, <AxesSubplot:xlabel='Date'>]], dtype=object)




    
![png](output_2_1.png)
    



```python
# Create log-differenced series

dta['dln_cpi_us'] = (np.log(dta.cpi_us)).diff() * 100
dta['dln_cpi_gm'] = (np.log(dta.cpi_gm)).diff() * 100
dta['dln_cpi_uk'] = (np.log(dta.cpi_uk)).diff() * 100
dta['dln_cpi_fr'] = (np.log(dta.cpi_fr)).diff() * 100
dta['dln_cpi_it'] = (np.log(dta.cpi_it)).diff() * 100
dta['dln_cpi_ca'] = (np.log(dta.cpi_ca)).diff() * 100
dta['dln_cpi_jp'] = (np.log(dta.cpi_jp)).diff() * 100


# De-mean and standardize
dta['std_cpi_us'] = (dta['dln_cpi_us'] - dta['dln_cpi_us'].mean()) / dta['dln_cpi_us'].std()
dta['std_cpi_gm'] = (dta['dln_cpi_gm'] - dta['dln_cpi_gm'].mean()) / dta['dln_cpi_gm'].std()
dta['std_cpi_uk'] = (dta['dln_cpi_uk'] - dta['dln_cpi_uk'].mean()) / dta['dln_cpi_uk'].std()
dta['std_cpi_fr'] = (dta['dln_cpi_fr'] - dta['dln_cpi_fr'].mean()) / dta['dln_cpi_fr'].std()
dta['std_cpi_it'] = (dta['dln_cpi_it'] - dta['dln_cpi_it'].mean()) / dta['dln_cpi_it'].std()
dta['std_cpi_ca'] = (dta['dln_cpi_ca'] - dta['dln_cpi_ca'].mean()) / dta['dln_cpi_ca'].std()
dta['std_cpi_jp'] = (dta['dln_cpi_jp'] - dta['dln_cpi_jp'].mean()) / dta['dln_cpi_jp'].std()
```


```python
# Get the endogenous data
endog = dta.loc['1970-01-01':'2022-10-01', 'std_cpi_us':'std_cpi_jp']

# View the first few rows
endog.tail()

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Country</th>
      <th>std_cpi_us</th>
      <th>std_cpi_gm</th>
      <th>std_cpi_uk</th>
      <th>std_cpi_fr</th>
      <th>std_cpi_it</th>
      <th>std_cpi_ca</th>
      <th>std_cpi_jp</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2022-06-01</th>
      <td>2.808277</td>
      <td>-0.621754</td>
      <td>0.402726</td>
      <td>1.018160</td>
      <td>1.283152</td>
      <td>0.779294</td>
      <td>-0.343639</td>
    </tr>
    <tr>
      <th>2022-07-01</th>
      <td>-0.921011</td>
      <td>0.653933</td>
      <td>0.254612</td>
      <td>-0.117392</td>
      <td>-0.041932</td>
      <td>-0.442003</td>
      <td>0.480201</td>
    </tr>
    <tr>
      <th>2022-08-01</th>
      <td>-0.984985</td>
      <td>0.394688</td>
      <td>0.109018</td>
      <td>0.298023</td>
      <td>0.602975</td>
      <td>-1.505977</td>
      <td>0.312700</td>
    </tr>
    <tr>
      <th>2022-09-01</th>
      <td>-0.306598</td>
      <td>4.406222</td>
      <td>-0.034455</td>
      <td>-2.232826</td>
      <td>-0.374513</td>
      <td>-0.593544</td>
      <td>0.309988</td>
    </tr>
    <tr>
      <th>2022-10-01</th>
      <td>0.207985</td>
      <td>1.364402</td>
      <td>2.032459</td>
      <td>1.723671</td>
      <td>5.330822</td>
      <td>0.922479</td>
      <td>0.632062</td>
    </tr>
  </tbody>
</table>
</div>




```python
dta.loc[:, 'std_cpi_us':'std_cpi_jp'].plot(subplots=True, layout=(4, 2), figsize=(15, 10));
```


    
![png](output_5_0.png)
    



```python
# Get the endogenous data
endog = dta.loc['1970-01-01':'2022-10-01', 'std_cpi_us':'std_cpi_jp']

# Create the model
mod = sm.tsa.DynamicFactor(endog, k_factors=1, factor_order=4, error_order=1)
initial_res = mod.fit(method='powell', disp=False)
res = mod.fit(initial_res.params, disp=False)

print(res.summary(separate_params=False))

fig, ax = plt.subplots(figsize=(10,5))

# Plot the factor
dates = endog.index._mpl_repr()
ax.plot(dates, res.factors.filtered[0], label='Factor')
ax.legend()

```

                                                                       Statespace Model Results                                                                   
    ==============================================================================================================================================================
    Dep. Variable:     ['std_cpi_us', 'std_cpi_gm', 'std_cpi_uk', 'std_cpi_fr', 'std_cpi_it', 'std_cpi_ca', 'std_cpi_jp']   No. Observations:                  634
    Model:                                                                              DynamicFactor(factors=1, order=4)   Log Likelihood               -5081.568
                                                                                                           + AR(1) errors   AIC                          10213.135
    Date:                                                                                                Fri, 09 Feb 2024   BIC                          10324.436
    Time:                                                                                                        14:08:09   HQIC                         10256.355
    Sample:                                                                                                    01-01-1970                                         
                                                                                                             - 10-01-2022                                         
    Covariance Type:                                                                                                  opg                                         
    ==================================================================================================
                                         coef    std err          z      P>|z|      [0.025      0.975]
    --------------------------------------------------------------------------------------------------
    loading.f1.std_cpi_us              0.3125      0.022     14.485      0.000       0.270       0.355
    loading.f1.std_cpi_gm              0.1968      0.019     10.224      0.000       0.159       0.235
    loading.f1.std_cpi_uk              0.3033      0.022     13.624      0.000       0.260       0.347
    loading.f1.std_cpi_fr              0.3844      0.023     16.441      0.000       0.339       0.430
    loading.f1.std_cpi_it              0.3695      0.022     16.790      0.000       0.326       0.413
    loading.f1.std_cpi_ca              0.2865      0.024     12.171      0.000       0.240       0.333
    loading.f1.std_cpi_jp              0.2298      0.021     11.174      0.000       0.189       0.270
    sigma2.std_cpi_us                  0.4777      0.021     22.862      0.000       0.437       0.519
    sigma2.std_cpi_gm                  0.8185      0.033     24.741      0.000       0.754       0.883
    sigma2.std_cpi_uk                  0.5435      0.024     22.698      0.000       0.497       0.590
    sigma2.std_cpi_fr                  0.2696      0.021     12.999      0.000       0.229       0.310
    sigma2.std_cpi_it                  0.2900      0.014     20.926      0.000       0.263       0.317
    sigma2.std_cpi_ca                  0.5963      0.026     22.898      0.000       0.545       0.647
    sigma2.std_cpi_jp                  0.7238      0.027     27.232      0.000       0.672       0.776
    L1.f1.f1                           0.6925      0.072      9.635      0.000       0.552       0.833
    L2.f1.f1                           0.0161      0.100      0.162      0.872      -0.180       0.212
    L3.f1.f1                          -0.0070      0.102     -0.069      0.945      -0.207       0.193
    L4.f1.f1                           0.2325      0.061      3.795      0.000       0.112       0.353
    L1.e(std_cpi_us).e(std_cpi_us)     0.3639      0.029     12.750      0.000       0.308       0.420
    L1.e(std_cpi_gm).e(std_cpi_gm)     0.0230      0.041      0.555      0.579      -0.058       0.104
    L1.e(std_cpi_uk).e(std_cpi_uk)     0.1882      0.027      6.995      0.000       0.135       0.241
    L1.e(std_cpi_fr).e(std_cpi_fr)    -0.1248      0.053     -2.361      0.018      -0.228      -0.021
    L1.e(std_cpi_it).e(std_cpi_it)     0.3419      0.033     10.351      0.000       0.277       0.407
    L1.e(std_cpi_ca).e(std_cpi_ca)    -0.0208      0.043     -0.487      0.626      -0.104       0.063
    L1.e(std_cpi_jp).e(std_cpi_jp)     0.1654      0.028      5.936      0.000       0.111       0.220
    ==============================================================================================================================================
    Ljung-Box (L1) (Q):     5.40, 0.35, 1.69, 1.21, 0.01, 0.14, 0.12   Jarque-Bera (JB):   176.83, 211.09, 1961.19, 26.96, 2278.82, 206.34, 808.90
    Prob(Q):                0.02, 0.56, 0.19, 0.27, 0.93, 0.71, 0.73   Prob(JB):                          0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00
    Heteroskedasticity (H): 1.55, 1.57, 0.24, 1.48, 0.48, 0.78, 0.14   Skew:                             -0.24, 0.72, 1.80, 0.01, 1.08, 0.33, 1.15
    Prob(H) (two-sided):    0.00, 0.00, 0.00, 0.00, 0.00, 0.07, 0.00   Kurtosis:                        5.54, 5.43, 10.82, 4.01, 12.03, 5.71, 8.04
    ==============================================================================================================================================
    
    Warnings:
    [1] Covariance matrix calculated using the outer product of gradients (complex-step).
    




    <matplotlib.legend.Legend at 0x202f37f9d30>




    
![png](output_6_2.png)
    



```python
fig, ax = plt.subplots(figsize=(10,5))

# Calculate the average of standardized, log-differenced CPI series
endog['avg_std_cpi'] = endog.loc[:,'std_cpi_us':'std_cpi_jp'].mean(axis=1)

# Plot the factor
dates = endog.index._mpl_repr()
ax.plot(dates, res.factors.filtered[0], label='Factor')

# Plot the average of standardized CPI series
ax.plot(dates, endog['avg_std_cpi'], label='Average of Std CPI', linestyle='--')

ax.legend()
plt.show()
```


    
![png](output_7_0.png)
    



```python
fig, ax = plt.subplots(figsize=(10,5))

# Calculate the average of standardized, log-differenced ind series
endog['avg_std_cpi'] = endog.loc[:, 'std_cpi_us':'std_cpi_jp'].mean(axis=1)

# Plot the factor
dates = endog.index._mpl_repr()
ax.plot(dates, res.factors.filtered[0], label='Factor')

# Plot the average of standardized ind series
ax.plot(dates, endog['avg_std_cpi'], label='Average of Std CPI', linestyle='--')

ax.legend()

start = '1970-01-01'
end = '2022-10-01'

# Retrieve NBER recession indicators and match to the dates range
rec = DataReader('USREC', 'fred', start=start, end=end).reindex(endog.index, method='ffill')

# Create a boolean array for recession periods matching 'dates' length
recession_indicator = rec['USREC'].values.astype(bool)

# Use the boolean array to fill recession periods
ax.fill_between(dates, ax.get_ylim()[0], ax.get_ylim()[1], where=recession_indicator, facecolor='grey', alpha=0.3)

plt.show()

```


    
![png](output_8_0.png)
    

