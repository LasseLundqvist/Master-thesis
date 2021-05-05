# Master-thesis
Code associated with my master thesis


#Calibration methodlogy 
# 1. Import relative modules and load vairables and parameters
import numpy as np
from scipy.stats import norm
from scipy.optimize import minimize
import pandas as pd

parameters = pd.read_excel("til rollo.xlsx")
V_equity = parameters["E"].values
sigma_equity = parameters["sigma_E"].values
default_point = parameters["D"].values
rf_rate = parameters["r"].values
T = parameters["T"].values

# 2.Create empty list to collect values of A_t and sigma_A,t
V_asset_list = []
sigma_asset_list = []

# 3. Create loop to solve 2x2 nonlinear system of eqautions
for i in range(len(V_equity)):
  def equation(x):
      d1 = (np.log(x[0]/default_point[i]) + (rf_rate[i]+x[1]**2/2)*T[i])/(x[1] * np.sqrt(T[i]))
      d2 = d1 - x[1] * np.sqrt(T[i])
      res1 = x[0] * norm.cdf(d1) - np.exp(-rf_rate[i]*T[i]) * default_point[i] * norm.cdf(d2) - V_equity[i]
      res2 = x[0] * norm.cdf(d1) * x[1] - V_equity[i] * sigma_equity[i]
      return(res1**2+res2**2)

  result = minimize(equation, [V_equity[i],sigma_equity[i]])
  V_asset = list(result['x'])[0]
  sigma_asset = list(result['x'])[1]
  #print('\nThe asset of the firm is %10.3f' % V_asset)
  #print('\nThe volatility of the firm asset is %10.5f' % sigma_asset)
  V_asset_list.append(V_asset)
  sigma_asset_list.append(sigma_asset)

# 4. Collect values of A_t and sigma_A,t
parameters["V_asset"]=V_asset_list
parameters["sigma_asset"]=sigma_asset_list

parameters.to_csv("All_asstets.csv",sep=";")
