## World-Layoffs
This is an SQL project on World Layoffs analyzing data across three years from March of 2020 assessing the impact of COVID-19 on major world organizations/corporations.

### Tool 
MYSQL

### Skills
- Data manipulation
- Conditional logic
- Joins
- Subqueries
- CTEs

  ### Data Cleaning
  The following tasks were performed during data cleaning and transformation
  - Removed Duplicates
  - Standardized the data
  - Checked and worked on null values
  - Removed unncessary rows and columns

 ### Duplicates
  ```sql
  SELECT *, 
     ROW_NUMBER() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off,
                                    `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_standard;

WITH duplicates_cte AS(
SELECT *, 
     ROW_NUMBER() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off,
                                    `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_standard)
SELECT * 
FROM duplicates_cte
WHERE row_num > 1;
  ```
### Data Standardization
```sql
UPDATE layoffs_standard2
SET company = TRIM(company);

SELECT DISTINCT industry
FROM layoffs_standard2;

SELECT *
FROM layoffs_standard2
WHERE industrY LIKE 'Crypto%';

UPDATE layoffs_standard2
SET industry = 'Crypto'
WHERE industrY LIKE 'Crypto%';

SELECT DISTINCT country, TRIM(TRAILING '.' FROM country)
FROM layoffs_standard2
ORDER BY 1;

UPDATE layoffs_standard2
SET country =  TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';

SELECT date, STR_TO_DATE(date, '%m/%d/%Y') AS date_formatted
FROM layoffs_standard2;

UPDATE layoffs_standard2
SET date = STR_TO_DATE(date, '%m/%d/%Y');

ALTER TABLE layoffs_standard2
MODIFY COLUMN date DATE;
```

### Nulls and Blanks
```sql
 SELECT t1.industry, t2.industry
 FROM layoffs_standard2 t1
 JOIN layoffs_standard2 t2
 ON t1.company = t2.company
 AND t1.location = t2.location
 WHERE (t1.industry IS NULL OR t1.industry = '')
 AND t2.industry IS NOT NULL;
 
 UPDATE layoffs_standard2
 SET industry = NULL
 WHERE industry = '';
 
 UPDATE layoffs_standard2 t1
 JOIN layoffs_standard2 t2
   ON t1.company = t2.company
 SET t1.industry = t2.industry
 WHERE t1.industry IS NULL 
 AND t2.industry IS NOT NULL;
```
### Delete Unnecessary Rows and Columns
```sql
 DELETE
FROM layoffs_standard2
WHERE total_laid_off IS NULL 
 AND percentage_laid_off IS NULL;
 
 ALTER TABLE layoffs_standard2
 DROP COLUMN row_num;
```

### Exploratory Data Analysis
```sql
-- date range
SELECT MIN(date), MAX(date)
FROM layoffs_standard2;

SELECT *
FROM layoffs_standard2
LIMIT 10;

SELECT MAX(total_laid_off), MAX(percentage_laid_off)
FROM layoffs_standard2;

SELECT *
FROM layoffs_standard2
WHERE percentage_laid_off = 1
ORDER BY funds_raised_millions DESC;

-- layoffs by company
SELECT company, SUM(total_laid_off) AS total_layoffs
FROM layoffs_standard2
WHERE total_laid_off IS NOT NULL
GROUP BY 1
ORDER BY total_layoffs DESC;

-- layoffs by industry
SELECT industry, SUM(total_laid_off) AS total_layoffs
FROM layoffs_standard2
WHERE total_laid_off IS NOT NULL
GROUP BY 1
ORDER BY total_layoffs DESC;

-- -- layoffs by country
SELECT country, SUM(total_laid_off) AS total_layoffs
FROM layoffs_standard2
WHERE total_laid_off IS NOT NULL
GROUP BY 1
ORDER BY total_layoffs DESC;

-- layoffs by stage
SELECT stage, SUM(total_laid_off) AS total_layoffs
FROM layoffs_standard2
WHERE total_laid_off IS NOT NULL
GROUP BY 1
ORDER BY total_layoffs DESC;

-- layoffs by year
SELECT YEAR(date) AS Year, SUM(total_laid_off) AS total_layoffs
FROM layoffs_standard2
WHERE total_laid_off IS NOT NULL
GROUP BY 1
ORDER BY 1;

-- monthly running totals 
WITH Running_Total AS(
SELECT SUBSTRING(date, 1, 7) AS Month, 
       SUM(total_laid_off) AS monthly_layoffs
FROM layoffs_standard2
WHERE SUBSTRING(date, 1, 7)  IS NOT NULL
GROUP BY SUBSTRING(date, 1, 7) 
ORDER BY Month)
SELECT Month, 
       monthly_layoffs,
       SUM(monthly_layoffs) OVER(ORDER BY Month) AS running_total
FROM Running_Total;

-- company yearly laidoffs
SELECT company, YEAR(date) AS year, SUM(total_laid_off) AS yearly_layoffs
FROM layoffs_standard2
WHERE YEAR(date) IS NOT NULL
GROUP BY 1,2
ORDER BY  2;

WITH company_years AS(
SELECT company, YEAR(date) AS year, SUM(total_laid_off) AS yearly_layoffs
FROM layoffs_standard2
WHERE YEAR(date) IS NOT NULL
GROUP BY 1,2
ORDER BY  2
),
company_year_rank AS(
SELECT *, DENSE_RANK() OVER(PARTITION BY year ORDER BY yearly_layoffs DESC) AS ranking 
FROM company_years
ORDER BY ranking ASC)
SELECT * 
FROM company_year_rank
WHERE ranking <= 5
ORDER BY year;

-- industry yearly laidoffs
SELECT industry, YEAR(date) AS year, SUM(total_laid_off) AS yearly_layoffs
FROM layoffs_standard2
WHERE YEAR(date) IS NOT NULL
GROUP BY 1,2
ORDER BY  2, 3 DESC;

WITH industry_years AS(
SELECT industry, YEAR(date) AS year, SUM(total_laid_off) AS yearly_layoffs
FROM layoffs_standard2
WHERE YEAR(date) IS NOT NULL
GROUP BY 1,2
ORDER BY  2, 3 DESC
),
industry_year_rank AS(
SELECT *, DENSE_RANK() OVER(PARTITION BY year ORDER BY yearly_layoffs DESC) AS ranking 
FROM industry_years
ORDER BY ranking ASC)
SELECT * 
FROM industry_year_rank
WHERE ranking <= 5
ORDER BY year;
```
### Insights
- The date range was between March 2020 and March 2023, signifying the impact of COVID-19.
- Amazon had the highest number of laid off (18150), followed by Google(12000) and Meta(11000).
- The top five industries hit the most were consumer, retail, transportation, finance, and healthcare.
- The top five countries with the most laid off employees were United States, India, Netherlands, Sweden, and Brazil.
