# COVID-SIR-Analysis
This Python script allows the user to select a country, and then produces a visualisation showing the SIR model, alongside the true infection figures.

The largest source of uncertainty in the script is the method for assigning a value to beta (a constant within the SIR equations). 
It represents the number of human contacts an infected person makes, with a larger value causing a faster rate of spread through a population. 
This lead me to use the average population density for the chosen country as a parameter to calculate a value for beta, dividing by the largest population density (Monaco), and then multiplying by a scaling constant. 

The main issue with this method is that there are far more complex factors that contribute to the true value of beta, so my method leads to similar SIR predictions between countries with similar population densities, that in reality have very different case profiles (see Belgium and Japan). A more reasonable calculation of beta would lead to improvements of the SIR model in this program.
