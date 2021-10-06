import scipy.integrate
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from datetime import timedelta

# Importing the two data files used - COVID and country data
covid_data = pd.read_csv("who_covid_data.csv")
country_data = pd.read_csv("countries of the world.csv")


# Cleaning the two data sources - changing data format and removing unrequired columns
covid_data["Date_reported"] = pd.to_datetime(covid_data['Date_reported'])

country_data = country_data.drop(["Region"], axis = 1)
country_data = country_data.drop(["Area (sq. mi.)"], axis = 1)
country_data = country_data.drop(["Coastline (coast/area ratio)"], axis = 1)
country_data = country_data.drop(["Net migration"], axis = 1)
country_data = country_data.drop(["Infant mortality (per 1000 births)"], axis = 1)
country_data = country_data.drop(["GDP ($ per capita)"], axis = 1)
country_data = country_data.drop(["Literacy (%)"], axis = 1)
country_data = country_data.drop(["Phones (per 1000)"], axis = 1)
country_data = country_data.drop(["Arable (%)"], axis = 1)
country_data = country_data.drop(["Crops (%)"], axis = 1)
country_data = country_data.drop(["Other (%)"], axis = 1)
country_data = country_data.drop(["Climate"], axis = 1)
country_data = country_data.drop(["Birthrate"], axis = 1)
country_data = country_data.drop(["Deathrate"], axis = 1)
country_data = country_data.drop(["Agriculture"], axis = 1)
country_data = country_data.drop(["Industry"], axis = 1)
country_data = country_data.drop(["Service"], axis = 1)

country_data["Country"] = country_data["Country"].astype('string')


# User can select a country to model and the length of the model
country = input("Please choose a country: ")
model_days = int(input("Please enter the number of days to model: "))

##########################################################################          SIR MODEL           ####################################################################

# Finding the chosen country's population density, and the largest listed population density value, to assign a custom variable to beta
B = country_data.loc[country_data["Country"] == country]["Pop. Density (per sq. mi.)"].iloc[0]
Bmax = country_data["Pop. Density (per sq. mi.)"].max()


# SIR equations used to model COVID behaviour, along with values for initial conditions and constants used
def sir(y, t, beta, gamma):
    S, I, R = y
    dS_dt = -beta*S*I
    dI_dt = beta*S*I - gamma*I
    dR_dt = gamma*I
    return([dS_dt, dI_dt, dR_dt])

S0 = 0.9
I0 = 0.1
R0 = 0
beta = (B/Bmax) * 100
gamma = 0.2


# Creation of time vector based on the model length chosen by user
t = np.linspace(0, model_days, model_days*100)


# Solutions of SIR equations with respect to the time vector, and the conversion of the results into an array
solution = scipy.integrate.odeint(sir, [S0, I0, R0], t, args = (beta, gamma))
solution = np.array(solution)

##########################################################################          ACTUAL DATA ANALYSIS            ############################################################

# Filtering the COVID data to only include chosen country and exclude the days listed with zero cases reported
covid_data = covid_data[covid_data["Country"] == country]
covid_data = covid_data[covid_data["Cumulative_cases"] != 0]


# Finding the date with the first reported case and the end date based on the length of the model 
first_date = covid_data["Date_reported"].min()
last_date = (first_date + timedelta(days = model_days))


# Finding the population of the chosen country
population = country_data.loc[country_data["Country"] == country]["Population"].iloc[0]

##########################################################################          VISUALISATION           ###################################################################


# Creating 2 subplots, the first showing the modelled SIR data, with the second showing the true recorded data
fig,axs = plt.subplots(2, 1, sharex = False)
plt.suptitle(f"Predicted vs. Real COVID Data for {country}")
axs[0].plot(t, solution[:,0], color = 'blue', label = "S(t)")
axs[0].plot(t, solution[:,1], color = 'green', label = "I(t)")
axs[0].plot(t, solution[:,2], color = 'red', label = "R(t)")
axs[0].legend()
axs[0].set_ylabel("Population (%)")
axs[0].set_xlabel("Number of days")

axs[1].plot(covid_data["Date_reported"], covid_data["New_cases"]/population, color = 'black', label = 'True Infected')
axs[1].set_xlim(first_date, last_date)
axs[1].set_ylabel("Population (%)")
axs[1].set_xlabel("Date")
axs[1].legend()
plt.xticks(rotation = 30)
plt.show()