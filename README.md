# 🧹 SQL-Data-Cleaning-Project

##  This project showcases a comprehensive data cleaning process performed on a dataset of tech company layoffs using MySQL Workbench. The raw dataset (layoffs.csv) was cleaned, standardized, and transformed into a reliable format for further analysis.
---
# 📁 Dataset Overview

The dataset includes information about tech layoffs across various companies and countries, containing attributes like:

company

location

industry

total_laid_off

percentage_laid_off

date

stage

country

funds_raised_millions

---
# 🛠️ Cleaning Methodology
The data cleaning process was executed using SQL and followed these key steps:

1. ## 🗂️ Duplicates Removal
Created a staging table to preserve original data.

Used ROW_NUMBER() with PARTITION BY to identify duplicates.

Removed duplicate rows from the dataset.

2. ## ✨ Data Standardization
Trimmed whitespaces from company and country fields.

Standardized industry values (e.g., normalized all variations of "crypto" to Crypto).

Removed punctuation (e.g., full stops from United States.).

Converted date from TEXT to proper DATE datatype using STR_TO_DATE().

3. ## 🕳️ Handling Null and Missing Values
Identified null and blank fields in industry and total_laid_off.

Used self-join logic to fill in missing industry values based on existing company records.

Updated empty strings in fields to NULL.

4. ## 🧹 Column Cleanup
Dropped intermediate/temporary columns like row_num once cleaning was complete.

---
## 🧾 SQL Script
The entire data cleaning process is documented in the SQL data cleaning Project.sql file. It is well-commented and structured to guide readers step-by-step through the transformation process.

Data Cleaning Methodology
1. Creating a Staging Table
First, I created a staging table to preserve the original data:

```sql
CREATE TABLE layoffs_staging LIKE layoffs;

INSERT layoffs_staging SELECT * FROM layoffs;
```

2. Identifying and Removing Duplicates
Since the table lacked a primary key, I used window functions to identify duplicates:

```sql
-- Identify duplicates using ROW_NUMBER()
SELECT *,
ROW_NUMBER() OVER (
    PARTITION BY company, location, industry, total_laid_off, 
    percentage_laid_off, 'date', stage, country
) AS row_num
FROM layoffs_staging;

-- Create a new table with row numbers for duplicate removal
CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  row_num int
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- Insert data with row numbers
INSERT INTO layoffs_staging2
SELECT *,
ROW_NUMBER() OVER (
    PARTITION BY company, location, industry, total_laid_off, 
    percentage_laid_off, 'date', stage, country
) AS row_num
FROM layoffs_staging;

-- Delete duplicates
DELETE FROM layoffs_staging2 WHERE row_num > 1;
```


3. Standardizing Data
```sql
-- Trim company names
UPDATE layoffs_staging2 SET company = TRIM(company);

-- Standardize industry names (e.g., all crypto variations to 'Crypto')
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'crypto%';

-- Fix country names (remove trailing periods)
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'united states%';

-- Convert date text to proper DATE type
UPDATE layoffs_staging2
SET date = STR_TO_DATE(date, '%m/%d/%Y');

ALTER TABLE layoffs_staging2 MODIFY COLUMN date DATE;
```

4. Handling Null and Blank Values
```sql
-- Set blank industry values to NULL
UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';

-- Fill null industry values using non-null values from the same company
UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
    ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;
```

## Key Findings During Cleaning
1. Duplicate Records: Several companies had duplicate entries that needed to be removed
2. Inconsistent Naming:
3. Multiple variations of "Crypto" in the industry field
4. "United States" appeared with and without a trailing period
5. Data Type Issues:
6. Dates were stored as text needing conversion to DATE type
7. Missing Data: Some industry values were blank or null but could be filled using information from other records for the same company

---
## 📊 Tools Used
MySQL Workbench

SQL (CTEs, Joins, Window Functions, Data Type Casting, etc.)

## ✅ Outcome
A clean, analysis-ready dataset (layoffs_staging2) suitable for:

Descriptive analysis (e.g., layoffs trends by industry or country)

## 🚀 Next Steps
Exploratory Data Analysis (EDA)

Dashboarding in Power BI or Tableau

Predictive modeling using Python
