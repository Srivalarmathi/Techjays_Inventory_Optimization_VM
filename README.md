# Techjays Inventory Optimization – Demand Forecasting & Stock Recommendations

Author: **Valarmathi Ganessin VM**  
Environment: **Python (Google Colab)**  

---

## 1. Project overview

This project implements a predictive inventory management system for Techjays, using historical sales and inventory data to:

- Forecast future demand at the **product level** (and optionally location level).
- Recommend **optimal inventory policies** such as:
  - Reorder Point (ROP)
  - Safety Stock
  - Economic Order Quantity (EOQ)
- Identify:
  - Products at risk of **stockout**
  - **Overstocked** items
  - **Slow-moving** and **dead stock** items

The goal is to support better purchasing and stocking decisions, reduce stockouts, and minimize excess inventory.

---

## 2. Business objectives

1. **Predict future sales**  
   - Use historical sales patterns to forecast demand.
   - Capture trends and seasonality where possible.

2. **Recommend optimal stock levels**  
   - Prevent stockouts for active, high-velocity products.
   - Reduce overstock and carrying costs.

3. **Improve inventory health metrics**  
   - Higher service level (95–99%).  
   - Lower stockout rate (<2–5%).  
   - Improved inventory turnover and controlled Days of Supply (DOS).

---

## 3. Data description

The analysis is based on the `Techjays Inventory.xlsx` workbook, which contains 7 sheets:

- **INVENTORY** – Current stock levels and inventory transactions.  
- **SALES** – Outbound customer sales and quantities shipped.  
- **RECEIVING TRANSACTIONS** – Inbound receipts from vendors.  
- **PRODUCTS** – Product master data including lead time and min levels.  
- **COMP** – Customer and vendor master information.  
- **PRODUCT GROUP** – Product grouping and category information.  
- **LOCATIONS** – Warehouse and consignment locations.

Key identifiers and relationships:

- `ProdCode` links SALES, INVENTORY, RECEIVING, and PRODUCTS.  
- `CompID` links SALES with COMP (customers).  
- `VendorID` links INVENTORY/RECEIVING with COMP (vendors).  
- `lngProductGroupID` links PRODUCTS with PRODUCT GROUP.  
- `Loc` / `Location` links transactional data with LOCATIONS.

---

## 4. Data preprocessing steps

The following business rules and cleaning steps are applied:

- Use data from **2019 onwards** only.
- Exclude locations where the delivery location code starts with **"C"** (consignment) or **"Q"** (quarantine).
- Remove products with **no sales in the last 2 years** (2024–2025).
- Remove invalid **negative inventory quantities**.
- Exclude non-product service codes such as:  
  `CANCEL, CERT, CREDIT ONLY, CREDIT CARD, CUT, DRAW, FREIGHT, GRIND, HEAT TREAT, MISC, PACKAGING, PROCESSING, STRAIGHT CUT, TEST, STRAIGHT/CUT PROC, STRAIGHT/CUT, CREDITCARD, FW, FWR`.
- Aggregate quantities for the same **ProdCode + Date** (e.g., sum `QtyShipped` per product per day).
- Ensure at least **5 time-series data points per product** for reliable forecasting.

Cleaned datasets are then aggregated (e.g., to daily or monthly level) and merged with product attributes (lead time, product group, etc.).

---

## 5. Methodology

### 5.1 Exploratory Data Analysis (EDA)

- Analyze sales trends over time (daily/weekly/monthly).  
- Identify high-volume and fast-moving products.  
- Analyze sales by:
  - Product group
  - Location
  - Customer
- Compare **receiving vs sales** patterns where relevant.
- Compute basic inventory KPIs such as inventory turnover and Days of Supply (when data is available).

### 5.2 Forecasting

- Prepare time series per product (and optionally per location).  
- Start with robust, interpretable baseline models (e.g., moving average / exponential smoothing).  
- Optionally evaluate advanced models on selected SKUs, such as:
  - ARIMA / SARIMA
  - Prophet
  - Tree-based models (XGBoost / Random Forest) with feature engineering
- Evaluate models using:
  - RMSE (Root Mean Squared Error)
  - MAE (Mean Absolute Error)
  - MAPE (Mean Absolute Percentage Error)

### 5.3 Inventory policy calculations

Using historical demand and product lead time:

- **Average Daily Demand (D)** per product.  
- **Lead Time (L)** from the `Products` sheet (`intLeadTime`).  
- **Safety Stock (SS):**  

  

\[
  SS = Z \times \sigma_D \times \sqrt{L}
  \]



  Where:
  - \( Z \) = service level factor (e.g., 1.65 for 95% service level)
  - \( \sigma_D \) = standard deviation of demand
  - \( L \) = lead time (in days)

- **Reorder Point (ROP):**

  

\[
  ROP = (D \times L) + SS
  \]



- **Economic Order Quantity (EOQ):**

  Classical EOQ formula using assumed or provided:
  - Annual demand
  - Ordering cost per order
  - Holding cost per unit per year

---

## 6. Outputs & deliverables

The project produces:

- **Forecasted demand** per product (and optionally per location).  
- A consolidated **inventory recommendation table** including:
  - ProdCode  
  - Product group  
  - Average daily demand  
  - Lead time  
  - Safety stock  
  - Reorder point (ROP)  
  - EOQ (if applicable)  
  - Velocity classification (fast / slow mover)
- Identification of:
  - Products at risk of **stockout**  
  - **Overstocked** products  
  - **Slow-moving** or **dead stock** items
- Visualizations:
  - Historical vs forecast demand  
  - Current vs recommended inventory levels  
  - Inventory risk segments by product and/or location

---

## 7. How to run this project

1. Open the notebook in **Google Colab** (click "Open in Colab" from GitHub or upload manually).  
2. Upload `Techjays Inventory.xlsx` to a known path (e.g., Google Drive).  
3. Update the file path in the notebook where required.  
4. Run all cells in order:
   - Data loading and cleaning  
   - EDA  
   - Forecasting  
   - Inventory policy calculations  
   - Reporting

Python version: **3.x**  
Key libraries: `pandas`, `numpy`, `matplotlib`, `seaborn`, `statsmodels`, `prophet` (optional), `scikit-learn`.

---

## 8. Assumptions & limitations

- Forecasts are based solely on historical sales and the structure of the provided dataset.  
- Lead time variability and detailed cost parameters may be limited or approximated.  
- EOQ calculations may rely on assumed holding and ordering cost values if not explicitly available.  
- Model performance and recommendations may need refinement when integrated into a live production environment or combined with additional business constraints.

---

## 9. Next steps (potential enhancements)

- Integrate the solution into a scheduled pipeline (e.g., daily/weekly auto-refresh).  
- Build an interactive dashboard (Power BI / Tableau / Streamlit) for ongoing monitoring.  
- Enhance models with:
  - Price effects
  - Promotions
  - Customer segments
  - External factors (seasonality, holidays, macroeconomic data)
- Extend to multi-echelon inventory optimization if multiple network layers exist.

---
