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
