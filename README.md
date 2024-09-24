# Data Cleaning with MySQL

### Project Overview
The primary objective of this project is to clean and prepare a dataset named **Layoffs** for accurate analysis. This involves identifying and handling inconsistencies, null values, duplicates, and irrelevant data in the table to ensure the dataset is ready for further analysis or reporting.

### Dataset
The dataset used for this project is the "layoffs.csv".

The dataset contains records of layoffs from different companies across various locations. It includes information such as the company name, location, industry, total laidoff, percentage of laidoffs, date of layoffs, stage, country and funds raised.

#### Columns in the Dataset:
- company: Name of the company.
- location: The city where the layoffs occurred.
- industry: Industry the company belongs to.
- total_laid_off: The total number of employees laid off.
- percentage_laid_off: Percentage of employees laid off.
- date: Date of the layoffs.
- country: The country where the layoffs occurred.
- funds_raised_millions: The total amount of funds raised by the company.
- stage: Refers to the company’s stage in its financial or business lifecycle at the time of layoffs, below are the various stages:

  
  - Post-IPO: The company has gone public.
  - Series A-J: Various stages of private funding, with A being early-stage and J being later-stage.
  - Seed: The earliest funding stage for startups.
  - Private Equity: Funded by private equity firms.
  - Subsidiary: A part of a larger company.
  - Acquired: The company has been purchased.
  - Unknown: The stage is not disclosed.

### Objectives
1. Import the csv file into MySQL database.
2. Identify and handle missing values in critical columns.
3. Remove duplicate records that could cause errors in data analysis.
4. Standardize the data and ensure data format consistency.
5. Prepare the cleaned data for deeper analysis or reporting.

<br>

### Steps Taken in Layoffs Data Cleaning
#### 1. Data importing and inspection

*Table imported using Table-Data-Import-Wizard*

*create a new work table to protect the raw dataset.*

```sql
CREATE TABLE layoffs_view
LIKE layoffs;
```
*copy data into the new work table*

```sql
INSERT INTO layoffs_view 
SELECT * FROM layoffs;
```
*verify the New Work Table*

```sql
select * from layoffs_view
```

*there is no primary key, create a new unique column using row_number*

```sql
SELECT *, ROW_NUMBER()
OVER(PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, 
stage, country, funds_raised_millions) AS row_num
FROM layoffs_view;
```

*confirm if there are duplicate, if the unique value is greater than 1*

```sql
WITH duplicate_cte 
AS (SELECT *, ROW_NUMBER()
OVER(PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, 
stage, country, funds_raised_millions) AS row_num
FROM layoffs_view)
SELECT * FROM duplicate_cte
WHERE row_num > 1;
```

*duplicate found! Create another work table and move the query result of the row_number into it*

```sql
CREATE TABLE layoffs_view2
LIKE layoffs_view;
```

*Add a new column Row_Num, before moving the query result*

```sql
Alter Table layoffs_view2
Add Column row_num int;
```

*move the query result of the Row Number into layoffs_view2*

```sql
Insert into layoffs_view2
(SELECT *, ROW_NUMBER() OVER(PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, 
stage, country, funds_raised_millions) AS row_num
FROM layoffs_view);
```

*delete the duplicates*

```sql
DELETE FROM layoffs_view2
WHERE row_num > 1;
```

<br>

#### 2. Standardize the data

* *Whitespace Standardization, check for extra spacing in the columns using Trim*

```sql
select company, trim(company), location, trim(location), industry, trim(industry), stage, trim(stage), 
country, trim(country)
from layoffs_view2;
```

*whitespace found! Update and remove the whitespaces*

```sql
update layoffs_view2
set company = trim(company), location = trim(location), industry = trim(industry), 
stage = trim(stage), country = trim(country);
```
<br>

* *Standardizing Categorical Values*

*check with Location column*

```sql
select distinct location from layoffs_view2
order by location;
```

*correct & update {DÃ¼sseldorf, FlorianÃ³polis and MalmÃ¶} to {Dusseldorf, Florianopolis and Malmo} respectively*

```sql
UPDATE layoffs_view2
SET Location = CASE
WHEN Location = 'DÃ¼sseldorf' THEN 'Dusseldorf'
WHEN Location = 'FlorianÃ³polis' THEN 'Florianopolis'
WHEN Location = 'MalmÃ¶' THEN 'Malmo'
ELSE Location
End;
```

*check with Industry column*

```sql
SELECT distinct industry from layoffs_view2
order by industry;
```

*Inconsistency found. Update Crypto Currency & CryptoCurrency to Crypto*

```sql
UPDATE layoffs_view2
SET industry = 'Crypto'
WHERE industry LIKE 'crypto%';
```

*check with Country column*

```sql
select distinct country
from layoffs_view2
order by country;
```

*Inconsistency found. Update (United State.) to United State*

```sql
UPDATE layoffs_view2
SET Country = 'United State'
WHERE Country = 'United State.';
```
