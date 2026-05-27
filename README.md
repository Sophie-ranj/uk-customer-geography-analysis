# Customer Geography Spend Analysis with Python

This project analyzes UK customer address data to understand how customer spending is distributed across cities, with a specific focus on comparing London-based customers with the rest of the United Kingdom.

The main challenge in this project is that the dataset lacks a clean `city` column. Instead, each customer address is stored as free text, so the city must be extracted using Python text-cleaning and matching logic.

## Project Goal

The goal of this project is to create a minimum viable analysis that answers two business questions:

- Which UK cities appear underserved based on customer spending?
- Are customers mainly London-based, or is spending distributed across other parts of the UK?

## Dataset

The project uses two CSV files:

| File | Description |
| --- | --- |
| `addresses.csv` | Customer records including company ID, address, and total spend |
| `cities.csv` | Reference list of official UK cities used for city matching |

### Main Customer Table

| Column | Description |
| --- | --- |
| `company_id` | Unique customer company identifier |
| `address` | Customer address stored as free text |
| `total_spend` | Total customer spending in GBP |

## Tools Used

- Python
- PyCharm
- pandas
- matplotlib

## Data Preparation

The first step was to inspect the dataset and check whether the address and spending fields were usable for analysis.

Main preparation steps:

- Loaded the customer dataset using pandas
- Checked dataset shape and missing values
- Removed rows with missing addresses
- Converted address text to uppercase for consistent matching
- Created an `address_lines` column to inspect address structure
- Checked `total_spend` summary statistics to confirm there were no negative values

```python
customers = pd.read_csv("./data/addresses.csv")
customers = customers.dropna(subset=["address"])
customers["clean_add"] = customers["address"].str.upper()
```

## City Extraction Logic

Because the customer address field is unstructured, the city could not be extracted by using a fixed address line position.

Instead, this project uses a reference list of UK cities and searches for city names inside the cleaned address text.

City list cleaning steps:

- Removed country headings such as `England`, `Scotland`, `Wales`, and `Northern Ireland`
- Removed trailing asterisks from city names
- Converted city names to uppercase
- Used the cleaned list as a reference table for matching


```python

## Derived Columns

| Column | Purpose |
| --- | --- |
| `clean_add` | Standardized uppercase version of the address |
| `address_lines` | Number of lines in each address |
| `city` | Extracted city name based on the UK city reference list |

Addresses that could not be matched to a city were categorized as `OTHER`.

```

## Manual Data Quality Fix

During validation, the official city name `KINGSTON-UPON-HULL` did not appear in the extracted city results because many addresses used the shorter name `HULL`.

A manual adjustment was made to correctly classify these records.

```python
customers.loc[
    customers["clean_add"].str.contains("\nHULL,", regex=True),
    "city"
] = "HULL"
```

## Analysis

After creating the `city` column, customer spending was grouped by city.

```python
top_20_spend = (
    customers
    .groupby("city")["total_spend"]
    .sum()
    .sort_values(ascending=False)
    .head(20)
)
```

The project also compares:

- Total customer spend
- Total spend from London customers
- Total spend outside London
- Total spend outside London, excluding the `OTHER` category

## Visualization

A horizontal bar chart was created to show the top 20 cities by total customer spending.

![Total Customer Spend by City](images/Chart.png)

## What I Practiced

- London has the highest customer spending among the identified UK cities.
- The `OTHER` category is very large, showing that many customers are outside official UK cities or cannot be classified with this method.
- London customers generate nearly as much spending as all other identified major UK cities combined.
- The analysis suggests the customer base is London-centric, but the result should be interpreted carefully because of the large `OTHER` category.


## Project Source

This project is based on Chapter 2, **Encoding Geographies**, from *The Well-Grounded Data Analyst* by David Asboth.


## Author

**Sophie Ranj**

Data Analyst  
[Portfolio](https://sophie-ranj.github.io/)
