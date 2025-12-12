# Evaluating the Effectiveness of Opioid Prescription Regulations
*A Comparative Analysis of Florida (2010) and Washington (2012)*

## Overview

This repository contains all data, code, figures, and documentation for a policy analysis evaluating the impacts of two statewide opioid-prescribing interventions:

- Florida’s 2010 pain-clinic enforcement crackdown  
- Washington’s 2012 clinical guideline–based policy  

Using county-level data on opioid shipments, overdose mortality, and population, the study compares pre/post trends and applies a difference-in-differences framework to assess whether each policy meaningfully changed prescriber behavior and opioid-related harm. The analysis supports a policy memo targeted at state PDMP Directors and public health decision-makers.

---

## Research Questions

1. Did Florida’s 2010 enforcement policy reduce opioid shipments per capita?  
2. Did Florida’s policy reduce opioid overdose mortality?  
3. Did Washington’s 2012 guideline-based policy reduce opioid shipments per capita?  
4. Did Washington’s policy reduce opioid overdose mortality?  

---

## Key Findings

- **Florida’s enforcement strategy worked.**  
  Shipments declined sharply after 2010 and mortality trends improved modestly.

- **Washington’s guideline-only approach did not.**  
  Shipments and mortality trends remained parallel to control states.

- **Implication:**  
  Enforcement-backed oversight is far more effective than advisory guidance alone in reducing opioid availability and harm.

---

## Repository Structure


---

## How to Reproduce the Analysis

### 1. Install dependencies
```bash
pip install -r requirements.txt
```
### 2. Run data cleaning notebooks (02_data_preparation)

#### 01_population_cleaning.ipynb

This notebook processes **U.S. Census county population estimates** from multiple raw datasets spanning 2000–2024 to produce a unified, analysis-ready table for the opioid study. It loads three separate Census files covering 2000–2010, 2010–2019, and 2020–2024, filters for county-level records, reshapes each from wide to long format, generates standardized FIPS codes, and then concatenates them into a single dataset. Finally, it filters the combined data to include only the nine states relevant to the opioid policy analysis and exports the cleaned population file (`population_2000_2024.csv`).

- **Purpose:** Clean and harmonize U.S. Census county population estimates (2000–2024) from three raw files, standardize county identifiers.
- **Input:**
  - `01_data/raw/population/co-est00int-tot.csv`
  - `01_data/raw/population/co-est2019-alldata.csv`
  - `01_data/raw/population/co-est2024-alldata.csv`
- **Output:**
  - `01_data/clean/population_2000_2024.csv`
- **Key Steps:** Filter to county level, reshape to long format, merge years, create FIPS codes.

#### 02_mortality_cleaning.ipynb

This notebook cleans and processes **CDC mortality data (2003–2015)** to prepare it for analysis in the opioid study. It downloads a ZIP of annual mortality text files, reads and concatenates them into a single DataFrame, inspects the structure and missing values, and begins cleaning steps such as removing duplicates and handling unusual or placeholder values. The goal is to transform raw vital statistics into a structured dataset with county-level opioid-related death counts that can later be merged with population estimates and used for exploratory analysis and modeling in subsequent parts of the project.

- **Purpose:** Clean CDC WONDER mortality data (2003–2015), merge with population estimates.
- **Input:**
  - CDC WONDER mortality data (downloaded zip)
  - `01_data/clean/population_2000_2024.csv`
- **Output:**
  - `01_data/clean/merged_mortality_population.csv`
- **Key Steps:** Download and extract mortality data, clean and merge with population, standardize columns.

#### 03_arcos_processing.ipynb

This notebook processes the **DEA ARCOS (Automation of Reports and Consolidated Orders System)** dataset to create a county-year aggregated view of opioid distribution for the study. It begins by defining and validating expected input files and required columns, loading a sample of the raw ARCOS transactional data to inspect structure and drug fields, and implementing helper functions for data quality checks (e.g., ensuring required files exist, checking for duplicates, and validating year ranges and positive values). The main pipeline then reads the full large ARCOS dataset in manageable chunks, **calculates Morphine Milligram Equivalents (MME) to standardize opioid quantity measures, aggregates the data by county and year,** and applies quality controls and validations, resulting in a cleaned, validated dataset suitable for merging with population and mortality data in subsequent analyses.

- **Purpose:** Process DEA ARCOS opioid shipment data, aggregate to county-year, calculate MME, filter outliers.
- **Input:**
  - `01_data/raw/arcos/` (raw ARCOS transaction files)
- **Output:**
  - `01_data/clean/arcos_by_county_year.csv`
  - `01_data/clean/arcos_end_use_only.csv`
- **Key Steps:** Aggregate transactions, calculate MME, filter non-patient transactions, and validate data.

These generate cleaned files in `01_data/clean/`.

### 3. Run data quality checks (03_data_quality)

#### 01 Investigate extreme values

- **Purpose:** Identify and filter extreme values and outliers in ARCOS data.
- **Input:** `01_data/clean/arcos_by_county_year.csv`, population data
- **Output:** Cleaned ARCOS data for merging
- **Key Steps:** Visualize distributions, flag outliers, document filtering criteria.

This notebook investigates and identifies extreme outliers in the processed ARCOS opioid distribution data that likely reflect **non‑patient‑level transactions** (e.g., shipments involving manufacturers or distributors) rather than typical county opioid use. It loads the aggregated county‑year ARCOS dataset along with population estimates, merges them to compute per‑capita MME (morphine milligram equivalents) for each county and year, and then examines the distribution of these per‑capita values. **Counties with exceptionally high per‑capita MME** are flagged as “extreme,” and the notebook explores how these extremes relate to buyer business activity types to inform filtering and data‑quality decisions before further analysis.

- Before filtering:
  - County-years: 8,386
  - Extreme values: 89 (1.06%)
  - Mean MME/capita: 1,453,856
  - Max MME/capita: 2,534,741,636

- After filtering:
  - County-years: 7,369
  - Extreme values: 2
  - Mean MME/capita: 4,221
  - Max MME/capita: 11,861,450

#### 02 Merge datasets

This notebook merges the three primary cleaned data sources—ARCOS opioid distribution data, county population estimates, and CDC mortality counts—into a single county‑year panel dataset for analysis. It starts by loading the cleaned ARCOS distribution totals (in morphine milligram equivalents), population data filtered to overlapping years (2006–2015), and aggregated mortality data for drug overdose deaths. It then constructs consistent merge keys across datasets, merges population with mortality, and finally merges in ARCOS totals, producing a unified table that preserves all county observations and includes per‑county population, mortality, and opioid distribution values. The merged dataset is saved as `final_merged.csv` for use in later exploratory and modeling steps.

- **Purpose:** Merge ARCOS, population, and mortality data into a single panel for analysis.
- **Input:**
  - `01_data/clean/arcos_end_use_only.csv`
  - `01_data/clean/merged_mortality_population.csv`
- **Output:**
  - `01_data/clean/final_merged.csv`
  - `01_data/clean/final_merged_150k.csv` (filtered for counties >150k population)
- **Key Steps:** Join datasets, harmonize columns, filter for analysis sample.

#### 03 Select population thresholds

- **Purpose:** Select optimal population threshold to balance missingness and sample size.
- **Input:** `01_data/clean/final_merged.csv`
- **Output:** Documentation of threshold selection, filtered dataset
- **Key Steps:** Analyze missingness vs. population, plot elbow curve, select threshold.

The notebook is about choosing a population threshold (a minimum county population) that balances:
- Data completeness: avoiding too many missing values from suppressed CDC data
- Sample size: keeping enough data for solid analysis
- Geographic coverage: retaining a sufficient number of counties
The purpose is to find an “elbow point” in the trade-off curve between missingness and population size.

Drug-related mortality counts are suppressed in counties with fewer than 10 deaths, which creates substantial missingness in small-population counties. To determine an appropriate population threshold for inclusion, we evaluated how missingness, the number of retained observations, and the number of retained counties
changed across thresholds.

<img width="1789" height="489" alt="image" src="https://github.com/user-attachments/assets/30d2bf6f-a0a3-421b-9be2-89b15ee7d130" />


This notebook includes charts on (1) percentage of missing mortality values, (2) total county-year observations retained, and (3) the number of unique counties retained at each threshold. A threshold of **150,000** residents strikes a balance between data completeness and sample coverage: at this cutoff, missingness is approximately 33.6%, the dataset retains 1,594 county-year observations, and 146 unique counties remain.

### 4. Run main analysis (04_main_analysis)

Florida pre/post analysis

#### Parallel trends and Difference-in-Difference (DiD) analysis for the states of Washington and Florida

**Overview**
This project analyzes the impact of opioid policy interventions in Florida (2010) and Washington (2012) using Difference-in-Differences (DiD) methods. We examine both opioid shipments (MME per capita) and overdose mortality rates, comparing treated states to matched control states.

**Data**
- **Source:** DEA ARCOS, CDC WONDER, US Census
- **Files:**
	- `01_data/clean/final_merged_150k.csv`: Main analysis dataset (county-year level)
	- `01_data/clean/merged_mortality_population.csv`: Mortality and population data

**Hypotheses Tested**
1. **Florida Policy Effect on Opioid Volume**
2. **Florida Policy Effect on Overdose Mortality**
3. **Washington Policy Effect on Opioid Volume**
4. **Washington Policy Effect on Overdose Mortality**

## Methods
- **Difference-in-Differences (DiD):**
	- Treated states: Florida, Washington
	- Control states: Five matched states per treated state
	- Pre/post periods: Florida (pre-2010/post-2010), Washington (pre-2012/post-2012)
- **Regression Specification:**
	- Outcome ~ treated + post + treated_post
	- OLS regression using `statsmodels`



#### Main Analysis
- **04_main_analysis/01_prepost_analysis.ipynb**
	- **Purpose:** Visualize pre/post policy trends in opioid shipments and mortality for Florida and Washington.
	- **Input:** `01_data/clean/final_merged_150k.csv`
	- **Output:** Trend plots
	- **Key Steps:** Aggregate by state and period, plot time series for treated states.

- **04_main_analysis/02_parallel_trends_analysis.ipynb**
	- **Purpose:** Assess parallel trends assumption for DiD analysis.
	- **Input:** `01_data/clean/final_merged_150k.csv`
	- **Output:** Parallel trends plots
	- **Key Steps:** Compare pre-policy trends between treated and control states, fit trendlines.

- **04_main_analysis/03_DiD_Analysis_Visuals.ipynb**
	- **Purpose:** Create DiD visualizations for all four hypotheses (opioid volume and mortality, FL and WA).
	- **Input:** `01_data/clean/final_merged_150k.csv`
	- **Output:** DiD plots with confidence bands
	- **Key Steps:** Group by treatment/control and period, plot means and CIs, use concise plotting function.

- **04_main_analysis/04_DiD_Hypothesis_Tests.ipynb**
	- **Purpose:** Formally test four DiD hypotheses using OLS regression, output summary tables.
	- **Input:** `01_data/clean/final_merged_150k.csv`
	- **Output:** Regression summary tables (markdown)
	- **Key Steps:** Specify DiD regression, run OLS, summarize results for each hypothesis.

---

### Data Cleaning Summary

- **Raw Data Sources:**
	- DEA ARCOS opioid shipment data (`01_data/raw/arcos/`)
	- CDC WONDER mortality data (downloaded zip)
	- U.S. Census county population estimates (`01_data/raw/population/*.csv`)

- **Cleaning Steps:**
	1. Clean and harmonize population data (2000–2024) from three Census files.
	2. Clean and merge CDC mortality data with population estimates.
	3. Process ARCOS data: aggregate, calculate MME, filter outliers.
	4. Merge all datasets to create a county-year panel.
	5. Filter for analysis sample (counties >150k population).

- **Cleaned Datasets Produced:**
	- `01_data/clean/population_2000_2024.csv`: Cleaned population data
	- `01_data/clean/merged_mortality_population.csv`: Mortality + population
	- `01_data/clean/arcos_by_county_year.csv`: Aggregated ARCOS
	- `01_data/clean/arcos_end_use_only.csv`: ARCOS, end-use only
	- `01_data/clean/final_merged.csv`: Merged analysis panel
	- `01_data/clean/final_merged_150k.csv`: Analysis sample (counties >150k)

## Key Results
- **Florida:**
	- Significant reduction in opioid shipments after policy (DiD estimate negative, p > 0.05)
	- Significant reduction in overdose deaths (p < 0.05)
- **Washington:**
	- No significant change in opioid shipments or deaths (p > 0.05)

See `04_DiD_Hypothesis_Tests.ipynb` for full regression tables and summary.

## How to Reproduce
1. Install requirements: `pip install -r requirements.txt`
2. Run the analysis notebooks in order:
	 - 01_prepost_analysis.ipynb
	 - 02_parallel_trends_analysis.ipynb
	 - 03_DiD_Analysis_Visuals.ipynb
	 - 04_DiD_Hypothesis_Tests.ipynb

## Citation
If you use this code or data, please cite the original data sources and this repository.

---

*For questions, contact the project maintainer.*


### 5. View outputs

Figures are stored in 05_outputs/figures/ and used in the final memo.

## Data Sources

This study integrates three public datasets:

DEA ARCOS: Opioid shipment records

CDC Vital Statistics: County-level overdose mortality

U.S. Census: County population estimates

Raw data files are stored in 01_data/raw/.
