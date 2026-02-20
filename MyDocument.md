# Data Engineering Coding Challenge – Documentation

## 1. Introduction

This project implements an end-to-end PySpark data processing pipeline to analyze NYC job postings data. The solution covers:

- Data exploration  
- Data cleaning  
- Feature engineering  
- KPI computation  
- Visualization  
- Processed data storage  

All transformations and analytics were implemented using **PySpark**, as required by the assignment.

---

## 2. Dataset Overview

The dataset (`nyc-jobs.csv`) contains job postings from the official New York City government jobs portal.

### Dataset Characteristics:
- ~2900+ records  
- 29 columns  
- Mix of:
  - Categorical columns (agency, job_category, posting_type)
  - Numeric columns (salary range, number of positions)
  - Date columns (posting_date, process_date)
  - Unstructured text columns (job_description, preferred_skills)

### Important Note

The dataset contains **only NYC job postings (USA)**.

Therefore, the analysis represents a subset of the US public sector job market and does not reflect the entire US job market.

---

## 3. Data Cleaning

### 3.1 Column Standardization

- Converted all column names to lowercase  
- Replaced spaces and special characters with underscores  

This ensures consistent schema naming and avoids transformation issues.

---

### 3.2 Numeric Column Cleaning

Salary columns contained currency symbols and commas (e.g., `$75,000.00`).  
Direct casting would result in NULL values.

Steps performed:
- Removed non-numeric characters using `regexp_replace`
- Casted values to `double`
- Renamed `#_of_positions` to `number_of_positions`
- Casted positions column to integer

---

### 3.3 Date Conversion

Converted the following columns to proper date type:

- posting_date  
- posting_updated  
- process_date  

Date conversion was validated by checking min and max posting dates.  
Dataset date range: **2012–2019**.

---

### 3.4 Null Handling

- Removed rows where salary average could not be computed  
- Ensured KPIs are calculated using valid salary data  

---

## 4. Feature Engineering

Three feature engineering techniques were applied:

### 4.1 Salary Average

Created:

salary_avg = (salary_range_from + salary_range_to) / 2

Purpose:
- Represents midpoint salary
- Used consistently across all salary-based KPIs

---

### 4.2 Degree Encoding

Created a numeric `degree_score` based on qualification text:

- PhD → 5  
- Master → 4  
- Bachelor → 3  
- Others → 1  

Purpose:
- Convert unstructured education requirements into numeric form
- Enable correlation analysis with salary

---

### 4.3 Posting Age

Created:

posting_age_days = datediff(current_date(), posting_date)

Purpose:
- Measure job recency
- Enable time-based analytics

---

## 5. KPI Implementation

### KPI 1: Number of Job Postings per Category (Top 10)

- Grouped by job_category  
- Counted rows using `count(*)`  
- Sorted descending  
- Limited to top 10  

Each row represents one job posting.

---

### KPI 2: Salary Distribution per Job Category

- Grouped by job_category  
- Calculated average of salary_avg  

Identifies highest-paying job categories.

---

### KPI 3: Correlation Between Degree and Salary

- Calculated Pearson correlation between:
  - degree_score
  - salary_avg  

Measures relationship between education level and compensation.

---

### KPI 4: Highest Salary per Agency

- Grouped by agency  
- Calculated maximum salary_avg  

Identifies top-paying agencies.

---

### KPI 5: Average Salary per Agency (Last 2 Years)

 filtering was based on:
(max posting_date in dataset) - 24 months
instead of system current date.

This ensures meaningful results.

---

### KPI 6: Highest Paid Skills

Steps:

1. Cleaned preferred_skills text  
2. Converted to lowercase  
3. Removed special characters  
4. Split into skill phrases  
5. Removed noise words  
6. Grouped by skill  
7. Calculated:
   - Average salary  
   - Frequency threshold  

This identifies skills associated with higher compensation in NYC public sector roles.

---

## 6. Assumptions

- Each row represents a unique job posting.  
- Salary midpoint approximates actual offered salary.  
- Dataset represents NYC public sector subset of US market.  

---

## 7. Challenges Faced

- Date conversion initially resulted in null values due to format mismatch.  
- Salary columns contained currency symbols and commas.  
- Preferred skills contained encoding artifacts. 

All issues were resolved through validation and cleaning adjustments.

---

## 8. Data Storage

Processed dataset stored as:

dataset/output/nyc_jobs_processed.parquet

Parquet was chosen because:
- Columnar format
- Efficient for analytics
- Optimized for Spark workloads

