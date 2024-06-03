Indian Start-Up Funding Data Analysis
Table of Contents
Project Description
Business Understanding
Data Understanding
Hypothesis
Research Questions
Installation
Data Preparation
Exploratory Data Analysis (EDA)
Conclusion and Recommendations
Project Description
This project seeks to gain insight into the funding received by start-up companies in India between 2018 and 2021. The goal is to advise a team trying to venture into the Indian start-up ecosystem by proposing the best course of action. This is achieved by developing a unique story from this dataset, stating and testing a hypothesis, asking questions, performing analysis, and sharing insights with relevant visualizations.

Business Understanding
Start-up funding plays a crucial role in providing essential capital to nurture new ventures that drive economic growth and technological advancement. The Indian startups ecosystem spans across various sectors and domains, such as e-commerce, fintech, edtech, healthtech, and agritech. This project aims to equip the team with the knowledge and strategic insights necessary to make informed decisions and successfully engage with the dynamic and rapidly evolving Indian start-up landscape.

Data Understanding
The datasets contain information about startup funding from 2018 to 2021. It includes various attributes such as the company’s name, sector, funding amount, stage, investor details, and location.

Key Attributes
Company/Brand: Name of the company/start-up
Founded: Year the start-up was founded
Sector: Sector of service
What it does: Description about the company
Founders: Founders of the company
Investor: Investors
Amount($): Raised fund
Stage: Round of funding reached
Headquarters: Location of the start-up company
Hypothesis
Null Hypothesis (H0): Funding to start-ups is centralized around specific locations and sectors.
Alternative Hypothesis (H1): Funding to start-ups is spread across different locations and sectors.
Research Questions
How has funding to startups changed over the period of time?
What is the average amount of funding for start-ups in 2019 and 2021?
Which headquarter is the most preferred startup location?
Which sectors are most favored by investors?
What are the most common funding stages among Indian startups?
Installation
To install the required packages, run:

bash
Copy code
pip install pyodbc python-dotenv pandas seaborn matplotlib
Data Preparation
Loading .env File and Creating a Connection to the Database
python
Copy code
import pyodbc
from dotenv import dotenv_values
import pandas as pd
import numpy as np
import warnings
warnings.filterwarnings('ignore')

# Load environment variables from .env file into a dictionary
environment_variables = dotenv_values('.env')

# Get the values for the credentials you set in the '.env' file
server = environment_variables.get("server")
database = environment_variables.get("database")
username = environment_variables.get("Login")
password = environment_variables.get("password")

# Create a connection string
connection_string = f"DRIVER={{SQL Server}};SERVER={server};DATABASE={database};UID={username};PWD={password}"
connection = pyodbc.connect(connection_string)
Accessing Data
python
Copy code
# Accessing 2021 data from the database
query = "SELECT * FROM dbo.LP1_startup_funding2021"
data_2021 = pd.read_sql(query, connection)

# Accessing 2020 data from the database
query = "SELECT * FROM dbo.LP1_startup_funding2020"
data_2020 = pd.read_sql(query, connection)

# Accessing a csv file containing 2019 data from the root directory of this project
data_2019 = pd.read_csv("startup_funding2019.csv")

# Accessing a csv file containing 2018 data from the root directory of this project
data_2018 = pd.read_csv("startup_funding2018.csv")

# Close the database connection
connection.close()
Data Cleaning and Preparation
python
Copy code
# Add 'Funding_Year' column
data_2021["Funding_Year"] = 2021
data_2020["Funding_Year"] = 2020
data_2019["Funding_Year"] = 2019
data_2018["Funding_Year"] = 2018

# Rename columns for consistency
data_2021.rename(columns={"Company_Brand": "CompanyName", "Amount": "Fund_Amount"}, inplace=True)
data_2020.rename(columns={"Company_Brand": "CompanyName", "Amount": "Fund_Amount"}, inplace=True)
data_2019.rename(columns={"Company/Brand": "CompanyName", "Amount($)": "Fund_Amount"}, inplace=True)
data_2018.rename(columns={"Company Name": "CompanyName", "Amount": "Fund_Amount", "Industry": "Sector", "Round/Series": "Stage", "Location": "HeadQuarter", "About Company": "What_it_does"}, inplace=True)

# Add missing columns in data_2018 for consistency
data_2018['Founders'] = np.nan
data_2018['Investor'] = np.nan
data_2018['Founded'] = np.nan

# Merge all datasets into one
joined_data = pd.concat([data_2018, data_2019, data_2020, data_2021], ignore_index=True)
joined_data.to_csv("joined_data.csv", index=False)

# Load the combined data
combined21_18 = pd.read_csv('joined_data.csv')

# Handle missing values
combined21_18["Sector"] = combined21_18["Sector"].fillna("Unknown").replace({"": "Unknown", "—": "Unknown"})
split_sector = combined21_18["Sector"].str.split(", ").explode()
most_common_sector = split_sector.value_counts().idxmax()
combined21_18["Sector"] = combined21_18["Sector"].apply(lambda x: most_common_sector if pd.isna(x) or (isinstance(x, str) and ',' not in x) else x)

combined21_18["Stage"] = combined21_18["Stage"].fillna("Unknown").replace("", "Unknown")

combined21_18["Fund_Amount"] = combined21_18["Fund_Amount"].replace(['Undisclosed', '$Undisclosed', '—', '-'], np.nan)

exchange_rates = {2018: 0.0146, 2019: 0.0142, 2020: 0.0135, 2021: 0.0135}

def convert_currency(row):
    amount = row["Fund_Amount"]
    year = row["Funding_Year"]
    
    if pd.isna(amount):
        return np.nan
    amount = str(amount)
    
    if "₹" in amount:
        cleaned_amount = amount.replace("₹", "").replace(",", "")
        return float(cleaned_amount) * exchange_rates[year] if cleaned_amount.isdigit() else np.nan
    elif "$" in amount:
        cleaned_amount = amount.replace("$", "").replace(",", "")
        return float(cleaned_amount) if cleaned_amount.isdigit() else np.nan
    else:
        cleaned_amount = amount.replace(",", "")
        return float(cleaned_amount) if cleaned_amount.isdigit() else np.nan

combined21_18["Fund_Amount"] = combined21_18.apply(convert_currency, axis=1)
combined21_18["Fund_Amount"].fillna(combined21_18["Fund_Amount"].median(), inplace=True)

most_common_headquarter = combined21_18["HeadQuarter"].mode()[0]
combined21_18["HeadQuarter"].fillna(most_common_headquarter, inplace=True)

combined21_18.to_csv("clean_data.csv", index=False)
Exploratory Data Analysis (EDA)
Distribution of Funding Amounts
python
Copy code
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

# Apply log transformation to the 'Fund_Amount' column to handle skewness
log_fund_amount = np.log1p(combined21_18['Fund_Amount'])

plt.figure(figsize=(10, 6))
sns.histplot(log_fund_amount, kde=True)
plt.title('Distribution of Funding Amounts (Log Scale)')
plt.xlabel('Log of Funding Amount')
plt.ylabel('Frequency')
plt.show()
Average Funding Amount for Start-ups in 2019 and 2021
python
Copy code
filtered_data = combined21_18[combined21_18['Funding_Year'].isin([2019, 2021])]
average_funding = filtered_data.groupby('Funding_Year')['Fund_Amount'].mean().reset_index()

plt.figure(figsize=(10, 6))
sns.barplot(data=average_funding, x='Funding_Year', y='Fund_Amount')
plt.title('Average Funding Amount for Start-ups in 2019 and 2021')
plt.xlabel('Year')
plt.ylabel('Average Funding Amount')
plt.tight_layout()
plt.show()
Top 10 Most Preferred Startup Locations by Headquarter
python
Copy code
headquarter_counts = combined21_18['HeadQuarter'].value_counts().reset_index()
headquarter_counts.columns = ['HeadQuarter', 'Count']
top_10_headquarters = headquarter_counts.head(10)

plt.figure(figsize=(14, 8))
sns.barplot(data=top_10_headquarters, x='HeadQuarter', y='Count')
plt.title('Top 10 Most Preferred Startup Locations by Headquarter')
plt.xlabel('Headquarter')
plt.ylabel('Number of Startups')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
Top 10 Most Favored Sectors by Investors
python
Copy code
sector_counts = combined21_18['Sector'].value_counts().reset_index()
sector_counts.columns = ['Sector', 'Count']
top_10_sectors = sector_counts.head(10)

plt.figure(figsize=(14, 8))
sns.barplot(data=top_10_sectors, x='Sector', y='Count')
plt.title('Top 10 Most Favored Sectors by Investors')
plt.xlabel('Sector')
plt.ylabel('Number of Startups')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
Top 10 Most Common Funding Stages among Indian Startups
python
Copy code
stage_counts = combined21_18['Stage'].value_counts().reset_index()
stage_counts.columns = ['Stage', 'Count']
top_10_stages = stage_counts.head(10)

plt.figure(figsize=(14, 8))
sns.barplot(data=top_10_stages, x='Stage', y='Count')
plt.title('Top 10 Most Common Funding Stages among Indian Startups')
plt.xlabel('Funding Stage')
plt.ylabel('Number of Startups')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
Conclusion and Recommendations
Conclusion
Funding Trends: The total funding amount has increased steadily over the years, with a significant spike in 2021.
Top Locations: The most funded locations are Mumbai, Bangalore, and New Delhi.
Key Sectors: Sectors such as Fintech, E-commerce, and Edtech attract the most funding.
Funding Stages: Early-stage funding rounds (Seed, Series A) are the most common among startups.
Recommendations
Focus on Key Locations: Invest in startups based in Mumbai, Bangalore, and New Delhi as these locations show high funding activity.
Invest in High-Potential Sectors: Target investments in Fintech, E-commerce, and Edtech sectors as they have shown strong funding trends.
Stage-wise Investment Strategy: Develop tailored strategies for early-stage funding rounds, as these are the most common and critical for startup growth.
Monitor Trends: Continuously monitor funding trends and adjust investment strategies accordingly to stay ahead in the dynamic startup ecosystem.