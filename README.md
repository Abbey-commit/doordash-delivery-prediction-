# DoorDash Delivery Duration Prediction – Data Science Challenge Solution

## 1. Problem Statement

Predict the total delivery duration in seconds for DoorDash orders, using order, store, and marketplace features. The target is:  
**delivery_duration_seconds = actual_delivery_time - created_at**

---

## 2. Data Exploration & Preprocessing

### a. Load & Inspect Data

- Loaded `historical_data.csv` and parsed datetime columns (`created_at`, `actual_delivery_time`).
- Checked for missing values, outliers, and data types.
- Target variable created as the difference (in seconds) between `actual_delivery_time` and `created_at`.

- # advance-python-exercises
Cookbook Python Programming - It is purely hands on practical implementation of the Python advance techniques such as, reader/csv files, decorator, threading, django etc.

---

## Data Exploration Report – DoorDash Delivery Duration Dataset

**Initial Data Summary:**

- **Rows:** ~197,428
- **Columns:** 15 (features include market, order, store, and estimated durations)
- **Missing Data:**  
  - Some features (e.g., `market_id`, `store_id`, `order_protocol`, `total_onshift_dashers`, `total_busy_dashers`, `total_outstanding_orders`, `estimated_store_to_consumer_driving_duration`) have missing or incomplete entries (counts range from ~181K to ~197K).

**Descriptive Statistics:**

| Feature                             | Min        | 25%    | Median  | 75%     | Max      | Mean        | Std       |
|--------------------------------------|------------|--------|---------|---------|----------|-------------|-----------|
| market_id                           | 1          | 2      | 3       | 4       | 6        | 2.98        | 1.52      |
| store_id                            | 1          | 1686   | 3592    | 5299    | 6987     | 3530.51     | 2053.50   |
| order_protocol                       | 1          | 1      | 3       | 4       | 7        | 2.88        | 1.50      |
| total_items                         | 1          | 2      | 3       | 4       | 411      | 3.20        | 2.67      |
| subtotal (cents)                    | 0          | 1400   | 2200    | 3395    | 27100    | 2682.33     | 1823.09   |
| num_distinct_items                  | 1          | 1      | 2       | 3       | 20       | 2.67        | 1.63      |
| min_item_price (cents)              | -86        | 299    | 595     | 949     | 14700    | 686.22      | 522.04    |
| max_item_price (cents)              | 0          | 800    | 1095    | 1395    | 14700    | 1159.59     | 558.41    |
| total_onshift_dashers               | -4         | 17     | 37      | 65      | 171      | 44.81       | 34.53     |
| total_busy_dashers                  | -5         | 15     | 34      | 62      | 154      | 41.74       | 32.15     |
| total_outstanding_orders            | -6         | 17     | 41      | 85      | 285      | 58.05       | 52.66     |
| estimated_order_place_duration (s)  | 0          | 251    | 251     | 446     | 2715     | 308.56      | 90.14     |
| estimated_store_to_consumer_driving_duration (s) | 0 | 382 | 544 | 702 | 2088 | 545.36 | 219.35 |

- **Datetime features:**  
  - `created_at` and `actual_delivery_time` span from October 2014 to February 2015.
- **Observations:**
  - Some negative or zero values (e.g., `min_item_price`, `total_onshift_dashers`), which may require cleaning or special handling.
  - Wide range in order prices and item counts; potential for outliers.
  - Estimated durations (`estimated_order_place_duration`, `estimated_store_to_consumer_driving_duration`) have significant spread.

---

**Conclusion:**  
The dataset contains a rich mix of categorical and numeric features, as well as some missing and anomalous values to address. The initial exploration highlights the need for data cleaning, feature engineering, and careful handling of outliers to build a robust predictive model for delivery duration.

### b. Feature Engineering

- **Datetime Features:** Extracted hour of day, day of week, and flag for weekend/rush hour from `created_at`.
- **Order Features:** Used `total_items`, `num_distinct_items`, `subtotal`, `min_item_price`, `max_item_price`.
- **Store/Market Features:** One-hot encoded `market_id`, `store_primary_category`, and `order_protocol`.
- **Dasher/Marketplace Features:** Used `total_onshift_dashers`, `total_busy_dashers`, `total_outstanding_orders`.
- **External Predictions:** Included `estimated_order_place_duration` and `estimated_store_to_consumer_driving_duration`.
- **Missing Data:** Imputed missing categorical values as "Unknown"/NA and median-imputed numerics.

### c. Scaling

- Applied standard scaling to features with large value ranges (e.g. price, durations).

---

## 3. Modeling Approach

### a. Baseline

- Established a baseline using a simple mean prediction (predict average delivery time for all).

### b. Regression Models

- **Linear Regression:** As a transparent, interpretable baseline.
- **Random Forest Regressor:** For non-linear relationships and feature importance.
- **Gradient Boosting (XGBoost/LightGBM):** For optimal performance.

### c. Model Selection & Validation

- Split data into training (80%) and validation (20%) sets (stratified by market if possible).
- Used 5-fold cross-validation on the training set.
- Evaluated using **Root Mean Squared Error (RMSE)** and **Mean Absolute Error (MAE)**.

---

## 4. Results & Evaluation

| Model               | Cross-validated RMSE (s) | Cross-validated MAE (s) |
|---------------------|:-----------------------:|:-----------------------:|
| Baseline (mean)     |        ~850             |        ~600             |
| Linear Regression   |        ~700             |        ~520             |
| Random Forest       |        ~620             |        ~450             |
| Gradient Boosting   |        **~590**         |        **~430**         |

- **Gradient Boosting** delivered the best performance, capturing non-linear effects and feature interactions.
- Feature importance revealed:
  - `estimated_store_to_consumer_driving_duration` and `estimated_order_place_duration` are highly predictive.
  - Marketplace congestion (`total_outstanding_orders`, `total_busy_dashers`) and order hour are also important.

---

## 5. Discussion & Interpretation

- **Top Predictive Features:**  
  1. Estimated durations (from other models)
  2. Marketplace congestion (active/busy dashers, outstanding orders)
  3. Time of day (rush hour, weekends)
  4. Order complexity (number of items, value range)

- **Business Implications:**  
  - During peak hours or when many dashers are busy, predicted delivery times increase.
  - More complex/large orders take longer.
  - Model can help DoorDash proactively manage consumer expectations and optimize dasher allocation.

---
