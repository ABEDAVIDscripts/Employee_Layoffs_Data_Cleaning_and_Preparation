# Data Cleaning with MySQL

### Project Overview
The primary objective of this project is to clean and prepare a dataset named **Layoffs** for accurate analysis. This involves identifying and handling inconsistencies, null values, duplicates, and irrelevant data in the table to ensure the dataset is ready for further analysis or reporting.

<br>

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
 
  <br>

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

<br>

#### 2. Remove Duplicates

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

#### 3. Standardize the data

* *Whitespace Standardization*

*Remove whitespaces in the columns using Trim*

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
<br>

* *Standardizing Date*

```sql
select `date` from layoffs_view2;
```

*Covert to standard date format*

```sql
UPDATE layoffs_view2
SET `date` = str_to_date(`date`, '%m/%d/%Y');
```
<br>

* *Standardizing DataType*

```sql
describe layoffs_view2;
```

*Alter Date column, change from text to date*

```sql
Alter table layoffs_view2
modify `date` date;
```

*Alter Percentage_laid_off column, change from text to float*

```sql
ALTER TABLE layoffs_view2
MODIFY percentage_laid_off float;
```
<br>


#### 4. Handling Null values or Blank values

*check for NULL & Blank space using industry column*

```sql
select * from layoffs_view2
where industry is null 
OR industry = '';
```

*There are 4 different companies affected.*

*first: Airbnb company. check if company have repeated values in the Industry column*

```sql
select * from layoffs_view2
where company = 'Airbnb';
```

*The record can be populated. Update the blank space for company Airbnb*

```sql
UPDATE layoffs_view2
SET industry = 'Travel'
WHERE company = 'Airbnb' 
AND (industry is NULL OR industry = '');
```

*second: Bally's Interactive company. check if company have repeated values in the Industry column*

```sql
select * from layoffs_view2
where company like 'Bally%';
```
*The record for Bally's company is a single record, therefore it can't be populated*


*third: Carvana company. check if company have repeated values in the Industry column*

```sql
select * from layoffs_view2
where company = 'Carvana';
```

*The record can be populated. Update the blank space for company Carvana.*

```sql
UPDATE layoffs_view2
SET industry = 'Transportation'
WHERE company = 'Carvana'
AND (industry = '' OR industry is NULL);
```

*fourth. Juul company. check if company have repeated values in the Industry column*

```sql
select * from layoffs_view2
where company = 'Juul';
```

*The record can be populated. Update the blank space for company Juul.*

```sql
UPDATE layoffs_view2
SET industry = 'Consumer'
WHERE company = 'Juul'
AND (industry = '' OR industry is NULL);
```

<br>

#### 5. Remove irrelevant records & columns

*check for blank space & null values in the columns that cannot be populated, using total_laid_off & percentage_laid_off columns*

```sql
select * from layoffs_view2
where (total_laid_off is null or total_laid_off = '')
and (percentage_laid_off is null or percentage_laid_off = '')
and (funds_raised_millions is null or funds_raised_millions = '');
```

*delete for the blankspaces and null conditions*

```sql
DELETE
FROM layoffs_view2
where (total_laid_off is null or total_laid_off = '')
and (percentage_laid_off is null or percentage_laid_off = '')
and (funds_raised_millions is null or funds_raised_millions = '');
```

*Check for the any other nulls and blank spaces values*

```sql
select * from layoffs_view2
where (total_laid_off is null or total_laid_off = '')
and (percentage_laid_off is null or percentage_laid_off = '');
```

*Observation: I believe this null values will be irrelevant for analysis, Total & Percentage Laid Off are part of the core data for analysis. Hence, Data is not trusted.*

```sql
DELETE
FROM layoffs_view2
where (total_laid_off is null or total_laid_off = '')
and (percentage_laid_off is null or percentage_laid_off = '');
```
