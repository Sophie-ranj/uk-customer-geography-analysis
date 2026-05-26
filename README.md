Encoding UK Customer Geographies with Python

This project implements the Chapter 2 Python analysis from The Well-Grounded Data Analyst by David Asboth. The goal is to extract city information from messy customer address data and use it to compare customer spending in London versus the rest of the United Kingdom.

Project Overview

The dataset contains customer company records with addresses stored as free text. There is no clean city column, so the main challenge is to create one from the address field.

The analysis answers two business questions:

1. Which UK cities appear underserved based on customer spending?
2. Are customers primarily based in London, or is spending spread across the rest of the UK?

This project follows a results-driven analytical approach:

- Understand the business question
- Define the minimum viable output
- Inspect and clean the data
- Create a city classification rule
- Analyze spending by city
- Highlight caveats and possible improvements

Dataset

The project uses two CSV files:
File	Description
addresses.csv	Customer records containing company ID, address, and total spend
cities.csv	Reference list of UK cities used to identify city names in addresses

Main columns in addresses.csv
Column	Description
company_id	Unique identifier for each customer company
address	Customer address stored as free text
total_spend	Total amount spent by the customer in GBP

Tools and Libraries

- Python
- PyCharm
- pandas
- matplotlib

Project Structure

well-grounded-data-analyst-chapter-2/
│
├── data/
│   ├── addresses.csv
│   └── cities.csv
│
├── outputs/
│   └── total_customer_spend_by_city.png
│
├── chapter_2_encoding_geographies.py
├── requirements.txt
└── README.md


Analysis Workflow

1. Load the customer data

The customer dataset is loaded with pandas and checked for shape, column structure, and missing values.

import pandas as pd

customers = pd.read_csv("./data/addresses.csv")
print(customers.shape)


2. Handle missing addresses

Rows with missing addresses are removed because the city cannot be extracted without address information.

customers = customers.dropna(subset=["address"])


3. Standardize address text

The address column is converted to uppercase so the matching process is consistent.

customers["address_clean"] = customers["address"].str.upper()


4. Explore address structure

Addresses are split by line breaks to understand how many lines each address contains.

customers["address_lines"] = (
    customers["address_clean"]
    .str.split(",\n")
    .apply(len)
)


This step shows that addresses do not all follow one fixed format, so extracting the city based only on line position would not be reliable.

5. Clean the UK city list

The city reference file is cleaned by removing country headings, removing asterisks, and converting city names to uppercase.

cities = pd.read_csv("./data/cities.csv", header=None, names=["city"])

countries_to_remove = ["England", "Scotland", "Wales", "Northern Ireland"]
cities_to_remove = cities[cities["city"].isin(countries_to_remove)].index
cities = cities.drop(index=cities_to_remove)

cities["city"] = cities["city"].str.replace("*", "", regex=False)
cities["city"] = cities["city"].str.upper()


6. Create the city column

Each address is checked against the cleaned city list. If a city name is found in the address, that city is assigned to the new city column. If no city is found, the address is categorized as OTHER.

for city in cities["city"].values:
    customers.loc[
        customers["address_clean"].str.contains(f"\n{city},", regex=True),
        "city"
    ] = city

customers["city"] = customers["city"].fillna("OTHER")


7. Fix the Hull naming issue

The official city name KINGSTON-UPON-HULL may appear in addresses as HULL, so the city classification is manually adjusted.

customers.loc[
    customers["address_clean"].str.contains("\nHULL,", regex=True),
    "city"
] = "HULL"


8. Analyze spending by city

Customer spending is grouped by the newly created city column.

top_20_spend = (
    customers
    .groupby("city")["total_spend"]
    .sum()
    .sort_values(ascending=False)
    .head(20)
)


9. Visualize the top spending cities

A horizontal bar chart is created to show the cities with the highest total customer spend.

from matplotlib.ticker import FuncFormatter
import matplotlib.pyplot as plt

def millions(x, pos):
    return "£%1.1fM" % (x * 1e-6)

formatter = FuncFormatter(millions)

fig, axis = plt.subplots()

top_20_spend = (
    customers
    .groupby("city")["total_spend"]
    .sum()
    .sort_values(ascending=False)
    .head(20)
    .sort_values(ascending=True)
)

top_20_spend.plot.barh(ax=axis)

axis.xaxis.set_major_formatter(formatter)
axis.set(
    title="Total customer spend by city",
    xlabel="Total spend"
)

plt.tight_layout()
plt.savefig("./outputs/total_customer_spend_by_city.png")
plt.show()


Key Findings

- London has the highest customer spending among identified UK cities.
- The OTHER category is very large, which suggests many customers are based outside official UK cities or have addresses that cannot be classified using this simple method.
- London customers generate nearly as much spending as all other major identified UK cities combined.
- Some important locations may be missed because the method depends on whether the city name appears directly in the address.

Business Interpretation

This analysis suggests that the customer base is strongly London-centric when comparing London with other major identified cities. However, the large OTHER category means the result should be presented with caution.

The analysis provides a useful minimum viable answer, but it should not be treated as a perfect geography classification model.

Limitations

This first version has several limitations:

- It only uses official UK city names.
- Large towns and London districts may be categorized as OTHER.
- Addresses without a city name cannot be classified accurately.
- The method may miss alternative spellings or abbreviations.
- Some addresses may contain a city name in a different context, such as a street name.

Future Improvements

Possible next steps:

- Use a UK postcode lookup dataset to identify cities more accurately.
- Expand the reference list to include towns, boroughs, and districts.
- Add population data to calculate spend per capita by city.
- Compare customer spending with regional demographics.
- Use geocoding carefully, while considering privacy and data protection requirements.
- Turn the script into reusable functions with tests.

How to Run This Project

1. Clone the repository.

git clone https://github.com/YOUR-USERNAME/well-grounded-data-analyst-chapter-2.git
cd well-grounded-data-analyst-chapter-2


2. Create a virtual environment.

python -m venv .venv


3. Activate the virtual environment.

On macOS or Linux:

source .venv/bin/activate


On Windows:

.venv\Scripts\activate


4. Install the required libraries.

pip install -r requirements.txt


5. Run the Python script.

python chapter_2_encoding_geographies.py


Requirements

Example requirements.txt:

pandas
matplotlib


What I Practiced

Through this project, I practiced:

- Cleaning messy real-world text data
- Creating a derived city column from unstructured addresses
- Using a reference dataset for classification
- Making analytical decisions under uncertainty
- Grouping and aggregating data with pandas
- Creating a business-focused visualization
- Documenting caveats and future improvements

Project Source

This project is based on Chapter 2, "Encoding Geographies", from The Well-Grounded Data Analyst by David Asboth.
