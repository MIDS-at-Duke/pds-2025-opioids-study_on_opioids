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

#### 02_mortality_cleaning.ipynb

This notebook cleans and processes **CDC mortality data (2003–2015)** to prepare it for analysis in the opioid study. It downloads a ZIP of annual mortality text files, reads and concatenates them into a single DataFrame, inspects the structure and missing values, and begins cleaning steps such as removing duplicates and handling unusual or placeholder values. The goal is to transform raw vital statistics into a structured dataset with county-level opioid-related death counts that can later be merged with population estimates and used for exploratory analysis and modeling in subsequent parts of the project.

#### 03_arcos_processing.ipynb

This notebook processes the **DEA ARCOS (Automation of Reports and Consolidated Orders System)** dataset to create a county-year aggregated view of opioid distribution for the study. It begins by defining and validating expected input files and required columns, loading a sample of the raw ARCOS transactional data to inspect structure and drug fields, and implementing helper functions for data quality checks (e.g., ensuring required files exist, checking for duplicates, and validating year ranges and positive values). The main pipeline then reads the full large ARCOS dataset in manageable chunks, **calculates Morphine Milligram Equivalents (MME) to standardize opioid quantity measures, aggregates the data by county and year,** and applies quality controls and validations, resulting in a cleaned, validated dataset suitable for merging with population and mortality data in subsequent analyses.

These generate cleaned files in `01_data/clean/`.

### 3. Run data quality checks (03_data_quality)

#### 01 Investigate extreme values

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

#### 03 Select population thresholds

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
