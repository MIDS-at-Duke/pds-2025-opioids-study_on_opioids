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

This notebook processes U.S. Census county population estimates from multiple raw datasets spanning 2000–2024 to produce a unified, analysis-ready table for the opioid study. It loads three separate Census files covering 2000–2010, 2010–2019, and 2020–2024, filters for county-level records, reshapes each from wide to long format, generates standardized FIPS codes, and then concatenates them into a single dataset. Finally, it filters the combined data to include only the nine states relevant to the opioid policy analysis and exports the cleaned population file (`population_2000_2024.csv`).

#### 02_mortality_cleaning.ipynb

#### 03_arcos_processing.ipynb

These generate cleaned files in 01_data/clean/.

### 3. Run data quality checks (03_data_quality)

#### Investigate extreme values

#### Merge datasets

#### Select population thresholds

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

Parallel trends and DID analysis

### 5. View outputs

Figures are stored in 05_outputs/figures/ and used in the final memo.

## Data Sources

This study integrates three public datasets:

DEA ARCOS: Opioid shipment records

CDC Vital Statistics: County-level overdose mortality

U.S. Census: County population estimates

Raw data files are stored in 01_data/raw/.
