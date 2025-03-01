# Layoffs Data Cleaning SQL Project

## Project Overview

**Project Title**: Layoffs Data Cleaning in SQL  
**Database**: `world_layoffs`  

This project focuses on **cleaning and preparing layoff data** for analysis. The dataset comes from [Kaggle](https://www.kaggle.com/datasets/swaptr/layoffs-2022) and includes layoffs across various industries, countries, and companies. The goal is to ensure **data integrity, accuracy, and consistency** before further analysis.

---

## Objectives

1. **Create a staging table** to work with cleaned data while keeping the raw data intact.  
2. **Remove duplicate records** to maintain data accuracy.  
3. **Standardize and correct inconsistent data formats** (industry names, country names, and dates).  
4. **Handle missing and null values** effectively.  
5. **Remove unnecessary columns and rows** that don't contribute to the analysis.  

---

## Project Structure

### **1. Database Setup**

#### **Create a Staging Table**
```sql
CREATE TABLE world_layoffs.layoffs_staging
LIKE world_layoffs.layoffs;

INSERT INTO world_layoffs.layoffs_staging
SELECT * FROM world_layoffs.layoffs;
```

---

### **2. Remove Duplicate Records**

#### **Check for Duplicates**
```sql
SELECT company, industry, total_laid_off, `date`,
       ROW_NUMBER() OVER (
           PARTITION BY company, industry, total_laid_off, `date`
       ) AS row_num
FROM world_layoffs.layoffs_staging;
```

#### **Identify True Duplicates**
```sql
SELECT * FROM (
    SELECT company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions,
           ROW_NUMBER() OVER (
               PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
           ) AS row_num
    FROM world_layoffs.layoffs_staging
) duplicates
WHERE row_num > 1;
```

#### **Delete Duplicate Records**
```sql
DELETE FROM world_layoffs.layoffs_staging
WHERE (company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) IN (
    SELECT company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
    FROM (
        SELECT company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions,
               ROW_NUMBER() OVER (
                   PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
               ) AS row_num
        FROM world_layoffs.layoffs_staging
    ) DELETE_CTE
    WHERE row_num > 1
);
```

---

### **3. Standardize Data**

#### **Handle Industry Nulls & Standardization**
```sql
UPDATE world_layoffs.layoffs_staging2
SET industry = NULL
WHERE industry = '';

UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;
```

#### **Fix Industry Naming Inconsistencies**
```sql
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry IN ('Crypto Currency', 'CryptoCurrency');
```

#### **Standardize Country Names**
```sql
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country);
```

#### **Convert Date Format to Standard Date Type**
```sql
UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;
```

---

### **4. Handle Null Values**

#### **Check for Null Values**
```sql
SELECT * FROM world_layoffs.layoffs_staging2
WHERE industry IS NULL
ORDER BY industry;
```

#### **Remove Useless Data (Unusable Null Rows)**
```sql
DELETE FROM world_layoffs.layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

---

### **5. Final Cleanup**

#### **Remove Unnecessary Columns**
```sql
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```

---

## **Findings & Insights**

- **Duplicates removed**, ensuring data integrity.
- **Industry and country names standardized** for consistency.
- **Missing industry values filled using company reference data.**
- **Date format corrected** for better time-based analysis.
- **Unusable null records removed** to improve data quality.

---

## **How to Use This Project**

1. **Clone the Repository**: If this is on GitHub, clone or download the SQL file.
2. **Set Up the Database**: Run the queries in your SQL environment.
3. **Import Data**: Load the layoffs dataset into `layoffs_staging`.
4. **Run Cleaning Queries**: Execute the provided SQL scripts to clean and standardize the data.

---

## **Author & Portfolio**

This project is part of my portfolio to demonstrate SQL data cleaning skills. If you have questions or feedback, feel free to connect with me!

### 📌 Stay Connected:
- **LinkedIn**: [www.linkedin.com/in/joshua-n-a28005216](#)
- **Email**: [joshuajos999@gmail.com](#)

🚀 **Thank you for exploring my SQL project!** 🚀

