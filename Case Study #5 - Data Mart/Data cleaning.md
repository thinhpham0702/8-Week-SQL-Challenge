# 🛒Case Study #5 - Data Mart

## Data Cleaning

**Table: weekly_sales (first 10 rows)**

| week_date | region | platform | segment | customer_type | transactions | sales    |
|-----------|--------|----------|---------|---------------|--------------|----------|
| 31/8/20   | ASIA   | Retail   | C3      | New           | 120631       | 3656163  |
| 31/8/20   | ASIA   | Retail   | F1      | New           | 31574        | 996575   |
| 31/8/20   | USA    | Retail   | null    | Guest         | 529151       | 16509610 |
| 31/8/20   | EUROPE | Retail   | C1      | New           | 4517         | 141942   |
| 31/8/20   | AFRICA | Retail   | C2      | New           | 58046        | 1758388  |
| 31/8/20   | CANADA | Shopify  | F2      | Existing      | 1336         | 243878   |
| 31/8/20   | AFRICA | Shopify  | F3      | Existing      | 2514         | 519502   |
| 31/8/20   | ASIA   | Shopify  | F1      | Existing      | 2158         | 371417   |
| 31/8/20   | AFRICA | Shopify  | F2      | New           | 318          | 49557    |
| 31/8/20   | AFRICA | Retail   | C3      | New           | 111032       | 3888162  |

In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales:
- Convert the week_date to a DATE format
- Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
- Add a month_number with the calendar month for each week_date value as the 3rd column
- Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values
- Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value

|segment	|age_band    |
|---------|------------| 
|1	      |Young Adults|
|2	      |Middle Aged |
|3 or 4	  |Retirees    |

- Add a new demographic column using the following mapping for the first letter in the segment values:

|segment	|demographic |
|---------|------------| 
|C	      |Couples     |
|F	      |Families    |

- Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns
- Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record

**Table: clean_weekly_sales (first 10 rows)**

| week_date  | week_number | month_number | calendar_year | region        | platform | segment | age_band     | demographic | customer_type | transactions | sales   | avg_transaction | record_id |
|------------|-------------|--------------|---------------|---------------|----------|---------|--------------|-------------|---------------|--------------|---------|-----------------|-----------|
| 2019-04-01 | 14          | 4            | 2019          | USA           | Shopify  | C2      | Middle Aged  | Couples     | New           | 160          | 27259   | 170.37          | 1         |
| 2019-04-01 | 14          | 4            | 2019          | SOUTH AMERICA | Shopify  | C4      | Retirees     | Couples     | Existing      | 7            | 1416    | 202.29          | 2         |
| 2019-04-01 | 14          | 4            | 2019          | CANADA        | Retail   | C4      | Retirees     | Couples     | New           | 13163        | 460052  | 34.95           | 3         |
| 2019-04-01 | 14          | 4            | 2019          | SOUTH AMERICA | Retail   | unknown | unknown      | unknown     | New           | 2327         | 83196   | 35.75           | 4         |
| 2019-04-01 | 14          | 4            | 2019          | ASIA          | Retail   | C2      | Middle Aged  | Couples     | New           | 76117        | 2039638 | 26.80           | 5         |
| 2019-04-01 | 14          | 4            | 2019          | SOUTH AMERICA | Retail   | unknown | unknown      | unknown     | Existing      | 668          | 29789   | 44.59           | 6         |
| 2019-04-01 | 14          | 4            | 2019          | OCEANIA       | Retail   | C2      | Middle Aged  | Couples     | New           | 96192        | 2740509 | 28.49           | 7         |
| 2019-04-01 | 14          | 4            | 2019          | ASIA          | Retail   | F1      | Young Adults | Families    | New           | 30023        | 986441  | 32.86           | 8         |
| 2019-04-01 | 14          | 4            | 2019          | OCEANIA       | Shopify  | F3      | Retirees     | Families    | New           | 171          | 27539   | 161.05          | 9         |
| 2019-04-01 | 14          | 4            | 2019          | ASIA          | Shopify  | unknown | unknown      | unknown     | New           | 176          | 30921   | 175.69          | 10        |

