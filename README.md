# Abdulrahman
Projects 
import pandas as pd
import numpy as np
data = pd.read_csv('projects\owid-covid-data.csv')
print("Dataset Loaded\n", data.head())
data = data[['location', 'date', 'total_cases', 'total_deaths', 'new_cases', 'new_deaths', 'people_fully_vaccinated_per_hundred']]
data['date'] = pd.to_datetime(data['date'])
data = data.dropna(subset=['total_cases', 'total_deaths', 'new_cases', 'new_deaths'])
data.fillna(0, inplace=True)
print("Data Cleaned\n", data.info())
data['case_fatality_rate'] = (data['total_deaths'] / data['total_cases']) * 100
global_data = data[data['location'] == 'World']
total_global_cases = global_data['total_cases'].max()
total_global_deaths = global_data['total_deaths'].max()
print(f"Global Total Cases: {total_global_cases}, Global Total Deaths: {total_global_deaths}")
top_countries_by_cases = data.groupby('location')['total_cases'].max().sort_values(ascending=False).head(5)
top_countries_by_vaccination = data.groupby('location')['people_fully_vaccinated_per_hundred'].max().sort_values(ascending=False).head(5)
print("Top 5 Countries by Total Cases:\n", top_countries_by_cases)
print("Top 5 Countries by Vaccination Rate:\n", top_countries_by_vaccination)
print("Conclusions:")
print(f"The global case fatality rate is {np.mean(data['case_fatality_rate']):.2f}%.")
print(f"The highest number of total cases was observed in:\n{top_countries_by_cases}")
print(f"The highest vaccination rate was observed in:\n{top_countries_by_vaccination}")
