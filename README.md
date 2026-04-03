# electricity-welfare-analysis

  code/
   └── electricity-welfare-analysis.do

   /********************************************************************

PROJECT: Electricity Prices and Household Welfare in the Philippines
AUTHOR: Althea M. Jonos
DATE: December 03, 2025

DESCRIPTION:
This script analyzes the impact of electricity prices and reliability
on household welfare using Philippine household survey data.

Welfare is measured through expenditure on:
- Food
- Health
- Education
- Communication

The analysis uses OLS regression with log transformations to estimate
elasticities and examines heterogeneous effects across income groups.

********************************************************************/


*====================================================*
* 0. INITIAL SETUP
*====================================================*

clear all
set more off

* Set working directory (UPDATE THIS PATH)
* cd "your/path/to/project"


*====================================================*
* 1. LOAD DATA
*====================================================*

* Replace with actual dataset file
use "data/your_dataset.dta", clear


*====================================================*
* 2. VARIABLE CONSTRUCTION
*====================================================*

*-------------------------------
* 2.1 Electricity Variables
*-------------------------------

* Log electricity price
gen ln_elec_price = log(averagepowerratephpkwh)


*-------------------------------
* 2.2 Welfare Variables
*-------------------------------

* Log expenditures (elasticity interpretation)
gen ln_food   = log(real_food)
gen ln_health = log(real_health)
gen ln_educ   = log(real_educ)
gen ln_comm   = log(real_comm)
gen ln_totex  = log(real_totex)


*-------------------------------
* 2.3 Household Controls
*-------------------------------

* Demographics
gen hh_age_sq = hh_age^2

* Income transformation
gen ln_pcinc = log(pcinc)


*-------------------------------
* 2.4 Regional Controls
*-------------------------------

gen ln_regional_gdp = log(regional_gdp_pc)


*====================================================*
* 3. INTERACTION TERMS
*====================================================*

* Urban heterogeneity
gen elecprice_urban = ln_elec_price * urb


*====================================================*
* 4. BASELINE REGRESSION MODELS
*====================================================*

/*
Model:
ln(Welfare) = β0 + β1 ln(Electricity Price)
            + β2 (Price × Urban)
            + Controls + ε

β1 = baseline elasticity (rural households)
β2 = additional effect for urban households
*/


*-------------------------------
* 4.1 Food Expenditure
*-------------------------------

reg ln_food ///
    ln_elec_price elecprice_urban ///
    hh_age hh_age_sq fsize ///
    hh_mstat hh_educ_level hh_sex ///
    hh_emp hh_informal urb ///
    ln_pcinc ///
    ln_regional_gdp regional_poverty regional_unemp regn_elec_access

estat ovtest


*-------------------------------
* 4.2 Health Expenditure
*-------------------------------

reg ln_health ///
    ln_elec_price elecprice_urban ///
    hh_age hh_age_sq fsize ///
    hh_mstat hh_educ_level hh_sex ///
    hh_emp hh_informal urb ///
    ln_pcinc ///
    ln_regional_gdp regional_poverty regional_unemp regn_elec_access


*-------------------------------
* 4.3 Education Expenditure
*-------------------------------

reg ln_educ ///
    ln_elec_price elecprice_urban ///
    hh_age hh_age_sq fsize ///
    hh_mstat hh_educ_level hh_sex ///
    hh_emp hh_informal urb ///
    ln_pcinc ///
    ln_regional_gdp regional_poverty regional_unemp regn_elec_access


*-------------------------------
* 4.4 Communication Expenditure
*-------------------------------

reg ln_comm ///
    ln_elec_price elecprice_urban ///
    hh_age hh_age_sq fsize ///
    hh_mstat hh_educ_level hh_sex ///
    hh_emp hh_informal urb ///
    ln_pcinc ///
    ln_regional_gdp regional_poverty regional_unemp regn_elec_access


*-------------------------------
* 4.5 Total Expenditure
*-------------------------------

reg ln_totex ///
    ln_elec_price elecprice_urban ///
    hh_age hh_age_sq fsize ///
    hh_mstat hh_educ_level hh_sex ///
    hh_emp hh_informal urb ///
    ln_pcinc ///
    ln_regional_gdp regional_poverty regional_unemp regn_elec_access


*====================================================*
* 5. EXPENDITURE SHARE ANALYSIS
*====================================================*/

* Budget shares
gen food_share   = real_food / real_totex
gen health_share = real_health / real_totex
gen educ_share   = real_educ / real_totex
gen comm_share   = real_comm / real_totex

foreach var in food_share health_share educ_share comm_share {
    reg `var' ///
        ln_elec_price elecprice_urban ///
        hh_age hh_age_sq fsize ///
        hh_mstat hh_educ_level hh_sex ///
        hh_emp hh_informal urb ///
        ln_pcinc ///
        ln_regional_gdp regional_poverty regional_unemp regn_elec_access
}


*====================================================*
* 6. INCOME HETEROGENEITY ANALYSIS
*====================================================*/

* Create income deciles
xtile income_decile = pcinc, n(10)

* Group into categories
gen income_group = .
replace income_group = 1 if income_decile <= 3
replace income_group = 2 if income_decile <= 7
replace income_group = 3 if income_decile > 7

label define income_grp 1 "Low" 2 "Middle" 3 "High"
label values income_group income_grp

* Run regressions by income group
forvalues i = 1/3 {
    reg ln_food ///
        ln_elec_price elecprice_urban ///
        hh_age hh_age_sq fsize ///
        hh_mstat hh_educ_level hh_sex ///
        hh_emp hh_informal urb ///
        ln_pcinc ///
        ln_regional_gdp regional_poverty regional_unemp regn_elec_access ///
        if income_group == `i'
}


*====================================================*
* 7. OUTPUT EXPORT (OPTIONAL)
*====================================================*/

* Install if needed:
* ssc install estout

* Export regression tables
* esttab using "output/results.rtf", replace ///
*     se star(* 0.10 ** 0.05 *** 0.01)


*====================================================*
* END OF FILE
*====================================================*/

  README.md

📊 Electricity Prices, Unreliability, and Household Welfare in the Philippines
 
  📌 Overview

  Electricity access in the Philippines is nearly universal—but household welfare has not improved proportionally.
  This project investigates why.

  Key question:
  
  How do electricity prices and unreliable supply affect how households allocate their budgets?
 
  Using household-level data, this study shows that electricity constraints reshape spending on essential goods such     as food, health, education, and communication.

  💡 Key Insight

  Electricity is not just a cost—it reshapes household decision-making. It affects welfare through two mechanisms:

  Price constraint → reduces disposable income and compresses consumption
  Reliability constraint → forces households to spend more on coping strategies
  
  ⚙️ Approach

  Dataset: 163,095 households from the 2023 FIES-LFS
  Merged with: Provincial electricity prices (DOE), Reliability indicators (SAIDI/SAIFI, ERC)
  Methodology: OLS regression (log-log specification)

  Coefficients interpreted as elasticities
  Interaction terms to capture urban/rural differences
  Income grouping for heterogeneous effects
  
  ⚙️ Data Workflow

  - Merged household survey with provincial electricity data
  - Constructed an electricity unreliability index using SAIDI and SAIFI
  - Applied log transformations for elasticity interpretation
  - Estimated regression models across multiple welfare dimensions
    
  📊 Key Results
  
  Electricity prices reduce household welfare
    - Higher prices → lower spending on food and total consumption

  Unreliability increases expenditure—but not welfare
    - ↑ Health, education, and communication spending

  📦 Data

  The dataset is not publicly included due to access restrictions. To replicate the analysis, you will need:

  - Household survey data (FIES-LFS)
  - Electricity price data (DOE)
  - Reliability indicators (SAIDI/SAIFI)
  - Regional socioeconomic variables
