# E-Commerce Shipping Analysis (R)

R-based exploratory analysis of an e-commerce shipping dataset to understand which operational factors are associated with the dataset’s delivery-timeliness flag.

## What’s in this repo
- `E-Commerce-Shipping-Data.csv` — the dataset (10,999 rows × 12 columns)
- `README.md` — this document

## Dataset columns (as provided)
- `ID`
- `Warehouse_block`
- `Mode_of_Shipment`
- `Customer_care_calls`
- `Customer_rating`
- `Cost_of_the_Product`
- `Prior_purchases`
- `Product_importance`
- `Gender`
- `Discount_offered`
- `Weight_in_gms`
- `Reached.on.Time_Y.N` (binary delivery-timeliness flag from the source dataset)

## Basic dataset facts (computed from the CSV)
- Rows: **10,999**
- Columns: **12**
- Label distribution (`Reached.on.Time_Y.N`):
  - Value **1**: **6,563** rows (**59.67%**)
  - Value **0**: **4,436** rows (**40.33%**)

## Analysis focus (what this project is for)
- Understand how the timeliness flag varies by:
  - shipment mode (`Mode_of_Shipment`)
  - warehouse block (`Warehouse_block`)
  - product importance (`Product_importance`)
  - discount level (`Discount_offered`)
  - package weight (`Weight_in_gms`)
  - customer interaction signals (`Customer_care_calls`, `Customer_rating`, `Prior_purchases`)
- Produce clean plots and a short driver summary that a non-technical reviewer can scan quickly

## How I want feedback (so people actually help)
Open a GitHub Issue titled: **Review: ecommerce-shipping-analysis-r** and tell me:
1) Which 2–3 visuals you’d keep if this was shown to a hiring manager  
2) What feature relationships look real vs misleading  
3) How you’d rewrite the project story in one paragraph for a recruiter
