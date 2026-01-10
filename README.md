# EUR/SEK Serverless Forecasting System

Group 2223 - Yuxin Jin and Yanling Lu



## Project Overview
https://lilianayl.github.io/ID2223_Project/

This project involves the design and implementation of a serverless machine learning system to predict the daily exchange rate between the Euro (EUR) and the Swedish Krona (SEK). The architecture follows a modular, four-stage pipeline pattern, utilizing a Feature Store to maintain separation between data engineering and model execution.

The system is designed for complete autonomy; it manages daily data ingestion, feature updates, and batch inference through scheduled workflows. By leveraging a serverless approach, the infrastructure scales dynamically to handle time-series data without the need for manual maintenance or dedicated server hosting.



## The Prediction Problem

The objective of this project is to build a robust, serverless forecasting system to predict the daily exchange rate between the Euro (EUR) and the Swedish Krona (SEK).

Predicting currency fluctuations is a complex time-series task influenced by macroeconomic stability and historical trends. To address this, our system utilizes three distinct categories of features:

- Macroeconomic Indicators: Real-time data including the EU Inflation Rate, Sweden's Inflation Rate, and Sweden’s Central Bank Interest Rates. These provide the fundamental economic context.

- Historical Trends (Lagged Features): To capture market momentum and autocorrelation, we engineered lagged features for the exchange rate over windows of 1, 3, 5, 10, and 15 days.

- Target Variable: The closing daily price of the EUR/SEK currency pair.



## System Architecture

### 1. Feature Backfill
This stage initializes the historical state of the system.
* **Data Ingestion**: Fetches historical data from the Frankfurter API (Exchange Rates) and Eurostat-linked sources (Inflation and Interest Rates).
* **Feature Group Creation**: Defines the schema and primary keys for three core Feature Groups in the **Hopsworks Feature Store**: `exchange_rate`, `inflation`, and `interest_rate`.
* **Historical Upload**: Populates the Feature Store with several years of data to provide a baseline for model training.

### 2. Feature Pipeline
This pipeline ensures the model always has access to the most recent macroeconomic data.
* **Automation**: Triggered daily via **GitHub Actions**.
* **Dynamic Upsert**: Fetches the latest daily exchange rates and indicators, performs light cleaning, and "upserts" (updates/inserts) them into the Feature Store.

### 3. Training Pipeline 
The "Brain" of the project where predictive logic is developed and versioned.
* **Unified Feature View**: Joins the three disparate feature groups on a common `date` key to create a training dataset.
* **Model Implementation**: Trains an **XGBoost Regressor** to predict the EUR/SEK rate.
* **Temporal Engineering**: Implements a "Lagged Model" that incorporates historical windows (1, 3, 5, 10, and 15-day shifts) to capture momentum and market trends.
* **Model Registry**: Stores the resulting artifacts (`model.json` and `model_lagged.json`) in the **Hopsworks Model Registry** with performance metrics (MSE, R²).

### 4. Batch Inference & Dashboard
The final stage that generates daily value and updates the user interface.
* **Daily Forecast**: Pulls the latest features and the current "best" model from the registry to predict the next day's exchange rate.
* **Hindcast Logic**: Compares the previous day’s prediction with the actual market data to calculate real-world error.
* **Automated Publishing**: Generates visualization plots and updates the **GitHub Pages Dashboard** automatically.



## Dynamic Data Sources

- **European Central Bank**: historical data can be found at: https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/html/index.en.html
  
  API is available at: https://api.frankfurter.app/
- **EU Inflation rate**: historical data can be found at: https://ec.europa.eu/eurostat/databrowser/view/PRC_HICP_MANR__custom_3761882/bookmark/table?lang=en&bookmarkId=4ad27e6f-358a-4a3d-82a0-587d69a833eb&c=1667558907980



## Dashboard

The dashboard is hosted via GitHub Pages and built using the Jekyll static site generator. 
- The site's behavior and theme are defined in _config.yml. It handles the base URL settings and metadata required to render the markdown files into a cohesive website.
- The primary interface is index.md. This file acts as a dynamic template that includes the latest prediction values, embeds the static plots which are updated daily by the automation workflow, and provides context for the model's performance, allowing stakeholders to see both future predictions and how well the model performed on yesterday's "hindcast."

The UI that shows the value of the predictions is available at: https://lilianayl.github.io/ID2223_Project/ 



## Technologies

### Machine Learning & Data Science
- Model Selection: The project utilizes the XGBoost Regressor (XGBRegressor) for currency prediction.
- Baseline & Improved Models: Two models are implemented: a baseline xgboost_model and an improved xgboost_model_lagged that incorporates time-lagged features (1, 3, 5, 10, and 15 days) to capture historical trends.
- Evaluation Metrics: Model performance is assessed using Mean Squared Error (MSE) and R-squared (R²) scores.
- Preprocessing: Data handling and feature engineering are performed using Pandas and NumPy.

### Data Sources & APIs
- Hopsworks Feature Store: Centralized infrastructure used for managing and serving features.
- Feature Groups: Three primary feature groups are maintained:
  - exchange_rate: Historical EUR/SEK rates.
  - inflation: EU and Swedish inflation rates.
  - interest_rate: Swedish interest rates.
- Feature Views: A combined feature view (macro_all_countries_fv) joins these groups on a time-series date key to create the final training dataset.
  
### Deployment & Automation

- Hopsworks Model Registry: Used to version and store trained model artifacts (model.json and model_lagged.json).
- Batch Inference Pipeline: An automated inference script downloads the latest models from the registry and generates daily forecasts.
- Visualization: Plots for forecasts and 1-day hindcasts are generated using Matplotlib.
- Environment Management: The project supports both Google Colab and local environments, with dependencies managed via uv and pip.
- GitHub Actions Workflow: The project is fully automated using GitHub Actions (currency-prediction.yml) using schedule - so that the pipeline is triggered automatically every day at 06:00 UTC, ensuring the dashboard always displays the most recent market data.





