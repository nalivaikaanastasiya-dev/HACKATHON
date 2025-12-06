# Germany: Social, Economic, Migration and Crime Indicators (2003–2020)

This project builds and analyzes an integrated panel dataset for **Germany** covering the years **2003–2020**.  
The goal is to combine data from multiple open sources into a single, clean DataFrame and use it to explore how
crime dynamics relate to social, economic, and demographic factors.

The final preprocessed dataset is published on Kaggle as:

> `paulbruk/germany-social-economic-indicators-20032020`

the whole project can be run in Google Colab under the following link:

> `https://colab.research.google.com/drive/1TYZZwHtko9AFnkAkdwHnX4nn7MmVdTzS?usp=sharing`

---

## 1. Data Sources and Scope

The project integrates the following types of indicators for Germany (2003–2020):

- **World Bank indicators**
  - Birth and death rates
  - GDP and GDP per capita (constant and current USD)
  - Internet usage
  - Infant mortality
  - Life expectancy
  - Population density
- **Macroeconomic indicators**
  - Inflation rate
  - Unemployment rate
  - Economic growth (GDP growth, %)
- **Migration data**
  - Inflows, outflows, and net migration
  - International migration indicators from Kaggle
- **Crime and homicide statistics**
  - Violent offences, sexual violence, kidnapping and related categories
  - Intentional homicides per 100,000 people
  - Disaggregation by category, dimension and sex where available

The unified dataset covers the period **2003–2020**, providing one row per year for Germany with all available indicators.

---

## 2. Preprocessing and Data Engineering

Preprocessing is mostly implemented in `Preprocessing.ipynb` and consists of:

1. **Base time–country backbone**

   - Create a Year–Country grid for Germany: all years from 2003 to 2020.
   - Use this as the main “skeleton” for subsequent merges.

2. **Cleaning and harmonizing inputs**

   - Loading World Bank data, economic series, migration, crime and homicide tables.
   - Standardizing column names and making sure `Year` is stored as integer.
   - Fixing inconsistent units or scales (e.g. GDP per capita magnitude issues).

3. **Reshaping homicide data**

   - Original homicide data is in wide format (years as columns).
   - Use `melt` to transform it into long format with a `Year` column and a single indicator:
     `Intentional homicides (per 100,000 people)`.

4. **Reshaping crime statistics**

   - Crime tables are pivoted so that each combination  
     _(Indicator, Dimension, Category, Sex)_  
     becomes a separate feature column named:
     `Crimes_<Indicator>_<Dimension>_<Category>_<Sex>`.
   - Result: a wide, feature-rich table with one row per (`Country`, `Year`).

5. **Merging all sources**

   - Merge World Bank indicators, economic indicators, migration data, homicides and pivoted crime data
     into a single DataFrame using left joins on `Country` and `Year`.
   - Reorder columns so that `Country` and `Year` appear first.
   - Drop redundant or unused variables such as:
     - `Country Code`
     - `Region`
     - `IncomeGroup`
     - intermediate technical columns not needed in the final dataset.

6. **Fallback and API dependency**
   - Some preprocessing steps rely on external statistical APIs (World Bank, etc.).
   - If those APIs are not reachable, a preprocessed CSV (`germany_data.csv`) is used as a fallback.
   - The final version is exported as `germany_data.csv` and published on Kaggle.

---

## 3. Analytical Workflow

The main analysis is implemented in `Project.ipynb` and is structured in several blocks:

### Block 1. Data Imputation and Basic Statistics

- Inspect missing values across demographic, economic, migration and crime variables.
- Apply simple imputation where reasonable (e.g. short gaps inside a continuous series).
- Compute basic descriptive statistics for crime indicators:
  - mean, median, variance, standard deviation
  - min, max and range
  - distribution shape and the presence of outliers.

### Block 2. Crime Time Trend Description (2003–2020)

- Plot time series for key violent and sexual crime categories.
- Identify:
  - A **sharp structural break** in some categories around **2005–2007**, where recorded incidents drop by more than 90%.
    - Interpretation: highly likely due to a change in definitions/reporting rather than a real sudden drop in crime.
  - **Post-2007 dynamics**:
    - Most violent and sexual crime indicators stabilize at a relatively low level.
    - Some categories show a slight upward trend over time.

### Block 3–4. Correlations Between Crime and Socio-Economic Indicators

- Compute and visualize correlation matrices between:
  - Crime indicators (e.g. violent offences, sexual violence)
  - Economic and demographic factors (GDP, GDP per capita, inflation, unemployment, life expectancy, population density, etc.).
- Key patterns identified:
  - **Strong negative correlations**:
    - Higher **life expectancy** and **GDP / GDP per capita** tend to be associated with **lower levels of recorded violence**.
  - **Strong positive correlations**:
    - **Unemployment rate** and **population density** tend to move in the same direction as several categories of violent crime.
  - **Weak or neutral correlations**:
    - Short-term indicators like **inflation** and **economic growth (%)** show weaker or unstable relationships with crime.

### Block 5–6. Dispersion and Group Differences (ANOVA / T-tests)

- Apply **ANOVA** and related tests to compare mean incident levels across different crime categories.
- Use t-tests / F-statistics to check if:
  - Crime categories significantly differ from each other in terms of average counts.
  - Particular economic or demographic strata show different average crime levels.
- Results:
  - Many differences between crime categories are **statistically significant**.
  - The p-values are typically far below conventional thresholds (0.05), but:
    - The small sample size (only 18 years) limits the robustness of these findings.

### Block 7. Linear Regression: Crime vs. Macro Indicators

- Fit basic linear regression models linking selected crime indicators to macro-variables.
- Examples:
  - Violent offences as a function of unemployment rate and population density.
  - Sexual violence as a function of economic and demographic indicators.
- These models highlight **direction of influence** (positive/negative) but are treated cautiously
  due to multicollinearity and the short time span.

### Block 8. Poisson and Binomial Models for Rare Crimes

- Some crime categories show very low yearly counts (rare events).
- Poisson and binomial distributions are used to:
  - Check whether counts are consistent with a Poisson process.
  - Explore the probability of observing a given number of incidents.
- The conclusion is that:
  - For very rare crime categories, Poisson/Binomial models are more appropriate than normal approximations.
  - However, data limitations prevent building highly reliable predictive models.

### Block 9. Time Series Modeling (ARIMA)

- Apply ARIMA models (e.g. ARIMA(1,1,1)) to selected aggregate crime indicators.
- Goals:
  - Capture the underlying temporal structure (trend + autoregression).
  - Produce baseline forecasts for the near future.
- Overall result:
  - After accounting for the early structural break, the model suggests a **flat or slightly declining**
    trajectory for certain crime categories, indicating **stability rather than explosive growth**.

### Block 10–12. Multiple Linear Regression and Final Forecast (2021–2025)

- Build a **Multiple Linear Regression (OLS)** model for a key crime indicator
  (e.g. sexual violence) using predictors like:
  - Unemployment rate
  - Life expectancy
  - Possibly other socio-economic variables.
- Evaluate:
  - Goodness of fit (R²) – in-sample explanatory power appears high.
  - Statistical significance of coefficients – several predictors have **non-significant p-values**,
    which, combined with multicollinearity and only 13–18 observations, makes causal interpretation fragile.
- Generate forecasts for **2021–2025** using projected or assumed paths for the predictors.
- Validate forecasts visually by comparing:
  - Historical series vs. model predictions.
  - Predicted future path vs. recent trends.

---

## 4. Key Findings and Conclusions

From the combined descriptive, correlation and modeling analysis, several high-level conclusions emerge:

1. **Data quality and structural breaks matter**

   - The sharp drop in some crime categories around 2005–2007 is almost certainly driven by
     changes in reporting or classification, not by a real-world overnight collapse in crime.
   - Any serious modeling must take this into account and avoid treating the entire period as homogeneous.

2. **Socio-economic context is strongly linked to crime levels**

   - Higher **life expectancy** and **GDP / GDP per capita** correlate with **lower violent crime**.
   - Higher **unemployment** and **population density** correlate with **higher crime levels**.
   - Short-term macro indicators like inflation or one-year GDP growth play a secondary role.

3. **Within-crime group differences are statistically significant**

   - Different crime categories (e.g. sexual violence vs. serious assault) have clearly distinct
     average levels and variance profiles.
   - ANOVA results confirm that not all crime types can be treated as a single homogeneous group.

4. **Forecasts show stability rather than explosive growth**

   - Both ARIMA-based time series forecasts and regression-based projections suggest that,
     after the early 2000s adjustment, overall violent crime remains **relatively stable** with
     no strong evidence of exponential growth in the near future.
   - Forecasts are still uncertain due to the small sample and structural changes in the data.

5. **Model limitations**
   - Small number of yearly observations (18 years) and strong multicollinearity between
     socio-economic indicators restrict reliable causal inference.
   - Some coefficients in OLS models are not statistically significant despite high R².
   - Crime reporting and classification changes introduce additional noise that models cannot fully absorb.

Overall, the project demonstrates how integrating socio-economic, demographic and crime data for Germany can provide
a richer picture of crime dynamics and their potential drivers, while also highlighting the limitations of forecasting
with short, noisy time series.
