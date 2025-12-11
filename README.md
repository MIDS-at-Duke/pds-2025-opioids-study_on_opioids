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

01_population_cleaning.ipynb

02_mortality_cleaning.ipynb

03_arcos_processing.ipynb

These generate cleaned files in 01_data/clean/.

### 3. Run data quality checks (03_data_quality)

Investigate extreme values

Merge datasets

Select population thresholds

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