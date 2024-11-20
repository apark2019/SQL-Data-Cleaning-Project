# SQL-Data-Cleaning-Project
In this project, we're going to walk through the steps I took to clean and organize a set of data using MySQL.





1. <b>Remove Duplicates</b>
2. <b>Standardize the Data</b>
3. <b>Null Values or blank values</b>
4. <b>Remove Any Columns</b>
5. <b>Final Dataset</b>
  <br />

<h2>Removing Duplicates</h2>
<h3>First I created a new table containing identical data, in case of a mistake while cleaning/organizing.</h3>

<p align="center">
  row_num duplicates: <br/>
  <img src="https://i.imgur.com/jBZS15C.png" height="30%" width="30%" alt="Disk Sanitization Steps"/>
  <br />
  <br />
  
CREATE TABLE layoffs_staging
LIKE layoffs;

SELECT *
FROM layoffs_staging;

INSERT layoffs_staging
SELECT *
FROM layoffs;

<h3>Here I created a new column as row_num to visuallize which rows had duplicates</h3>

<p align="center">
  row_num duplicates: <br/>
  <img src="https://i.imgur.com/aA3uw2Z.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
  <br />
  <br />

SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, industry, total_laid_off, percentage_laid_off, 'date') AS row_num
FROM layoffs_staging;

WITH duplicate_cte AS
(
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company,location, industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging
)
SELECT *
FROM duplicate_cte
WHERE row_num > 1;

SELECT *
FROM layoffs_staging
WHERE company = 'Casper';

WITH duplicate_cte AS
(
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company,location, industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging
)
DELETE
FROM duplicate_cte
WHERE row_num > 1;


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
  `row_num` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

SELECT *
FROM layoffs_staging2
WHERE row_num >1;

INSERT INTO layoffs_staging2
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company,location, industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;

<h3>This is the command I used to delete any rows that had a value of more than 1, which indicated duplicates</h3>

DELETE
FROM layoffs_staging2
WHERE row_num >1;

SELECT *
FROM layoffs_staging2;

<p align="center">
  Table With No Duplicates: <br/>
  <img src="https://i.imgur.com/QSL6ztu.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
  <br />
  <br />
  <br />
<h2>Standardizing data, finding any issues such as typos, extra spaces, or unnecessary characters.</h2>
<h3>Here I cleared up any unnecessary white spaces</h3>


<p align="center">
  White Spaces are Cleared: <br/>
  <img src="https://i.imgur.com/ZZnC4vO.png" height="35%" width="35%" alt="Disk Sanitization Steps"/>
  <br />
  <br />
  
SELECT company, TRIM(company)
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET company = TRIM(company);

<h3>We can look under the "industry" coloumn to find "Crypto" "CryptoCurrence" "Crypto Currency" and assume they are under the same industry.</h3>

<p align="center">
  Industry Error: <br/>
  <img src="https://i.imgur.com/3RgG2bA.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
  <br />
  <br />
  
SELECT DISTINCT industry
FROM layoffs_staging2
;

UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypt%';

<h3>Cleaning up typos cont.</h3>

<p align="center">
  Here we see that "United States." has an unnecessary "." at the end: <br/>
  <img src="https://i.imgur.com/oEHyMUD.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
  <br />
  <br />
  
SELECT DISTINCT country, TRIM(TRAILING '.' FROM country)
FROM layoffs_staging2
ORDER BY 1;

UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';

<h3>Here I Change the date coloumn to be in "Date" format.</h3>

<p align="center">
  Before and After I change the date from a "text" format to "date" format: <br/>
  <img src="https://i.imgur.com/x7og158.png" height="30%" width="30%" alt="Disk Sanitization Steps"/>
  <br />
  <br />
  
SELECT `date`
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;

<h2>Null Values or blank values</h2>

<h3>Here I am looking for any "industry" coloumns that are either blank or null values</h3>

<p align="center">
  Missing/null "industry" coloumn value: <br/>
  <img src="https://i.imgur.com/kmjpvjQ.png" height="30%" width="30%" alt="Disk Sanitization Steps"/>
  <br />
  <br />
  
SELECT *
FROM layoffs_staging2
WHERE industry IS NULL
OR industry = '';

<h3>We can see here that the company Airbnb has two rows, however one is missing the industry coloumn value.</h3>

<p align="center">
  Missing industry value: <br/>
  <img src="https://i.imgur.com/9y6MJdW.png" height="100%" width="100%" alt="Disk Sanitization Steps"/>
  <br />
  <br />
  
  SELECT *
FROM layoffs_staging2
WHERE company LIKE 'Airbnb%';

<h3> I use the join function to combine two of the same tables with the same company where the industry coloumn is missing values for comparison.</h3>
<p align="center">
  1st table: <br/>
  <img src="https://i.imgur.com/dJlkBhI.png" height="100%" width="100%" alt="Disk Sanitization Steps"/>
  <br />
  <br />
  <p align="center">
  2nd table: <br/>
  <img src="https://i.imgur.com/ukC8BGR.png" height="100%" width="100%" alt="Disk Sanitization Steps"/>
  <br />
  <br />
    
  SELECT *
FROM layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON t1.company = t2.company
	AND t1.location = t2.location
WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;

<h3>We can see here that I have succesfully filled out the missing values for industry.</h3>
  <p align="center">
  : <br/>
  <img src="https://i.imgur.com/pUY0WoN.png" height="100%" width="100%" alt="Disk Sanitization Steps"/>
  <br />
  <br />
    
UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON t1.company = t2.company
SET t1.industry = t2.industry 
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;

<h3>This is to see that I did not miss any industry coloumn null values</h3>

  <p align="center">
  <br/>
  <img src="https://i.imgur.com/HuJAGxE.png" height="100%" width="100%" alt="Disk Sanitization Steps"/>
  <br />
  <br />
    
SELECT *
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

<h2>Removing Coloumns</h2>
<h3>The first step I take regarding null and blank values is to see if they are necessary to keep or not.</h3>
<h4>I assume here that these are not necessary to keep and can be removed.</h4>

<p align="center">
  Total_laid_off and percentage_laid_off are null: <br/>
  <img src="https://i.imgur.com/p9rj09B.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
  <br />
  <br />
  
SELECT *
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';

DELETE
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

<h3>Removing row_num column.</h3>

ALTER TABLE layoffs_staging2
DROP COLUMN row_num;

<h2>Final Dataset</h2>
<h3>Here is the final dataset with all the steps taken to clean the data.</h3>
<p align="center">
  Final dataset: <br/>
  <img src="https://i.imgur.com/YeQNjI8.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
  <br />
  <br />
  
