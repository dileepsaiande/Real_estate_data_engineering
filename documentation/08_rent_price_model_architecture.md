# Rent Price Model Architecture Documentation

## 1. Executive Summary

The **Rent Price Model** estimates monthly rental costs for residential properties. Like the Buy Price Model, it is built on **PySpark MLlib's Linear Regression**. However, it incorporates specific financial features like **Deposit Cost** and **Maintenance Fees** and utilizes a unique normalization strategy to ensure predictions fall within a realistic consumer range (₹10,000 - ₹40,000). The model achieves an R² of approximately 0.58, reflecting the higher variance and subjectivity in rental markets compared to sales.

---

## 2. Model Architecture

### 2.1 High-Level Diagram

```mermaid
graph LR
    Input[Rental Features] -->|Features| Pipeline[Rent Prediction Pipeline]
    
    subgraph "Rent Prediction Pipeline"
        Pipeline -->|Encode| OHE[OneHotEncoder]
        OHE -->|Combine| Vector[Feature Vector]
        Vector -->|Predict| LinearReg[Linear Regression]
    end
    
    LinearReg -->|Raw Rent| PostProc[Post-Processing]
    PostProc -->|Normalize| Norm[Range Normalization]
    Norm -->|Final Rent| Output[Monthly Rent (₹)]
```

### 2.2 Algorithm Selection
**Algorithm**: Linear Regression
**Why**:
- **Simplicity**: Rental markets often follow standard price-per-area rules in specific locaties.
- **Speed**: Instant prediction for interactive "What-if" analysis (e.g., "What if I add a gym?").

---

## 3. Input Features (Unique to Rent)

While sharing many structural features with the Buy Model (BHK, Locality), the Rent Model includes:

### 3.1 Financial Features
- **Deposit Cost**: Security deposit amount (Integer). Higher deposits often correlate with lower monthly rent or higher property value.
- **HOA Fees**: Monthly maintenance/association fees.
- **Furnishing**: Impactful categorical variable (Full > Semi > Unfurnished).

### 3.2 Property Features
- **Project Type**: Gated Community vs. Individual House (Standalone houses vary more).
- **Availability Date**: Immediate availability vs. future (Time value of money).

---

## 4. Pipeline Components

### 4.1 Feature Transformation
Standard PySpark workflow:
1. **StringIndexer**: Text -> Index
2. **OneHotEncoder**: Index -> Vector
3. **VectorAssembler**: All -> Features

```python
# Specific Rent Features Addition
numeric_features += ["Deposit Cost", "HOA Fees ", "Furnishing_Index"]
```

### 4.2 Normalization Logic (Customer Requested)
To meet specific business rules, raw predictions are clamped to a target range.

```python
# Range Clamping Logic (in pages/customer.py)
rent_df["normalized_price"] = 5000 + (rent_df["square_feet"] * (10 + (rent_df["price"] % 5)))
rent_df["normalized_price"] = rent_df["normalized_price"].clip(10000, 40000)
```
*Note: This specific logic is applied during data loading for the dashboard search, while the ML model attempts to predict the true market value.*

---

## 5. Performance Metrics

| Metric | Value | Interpretation |
|--------|-------|----------------|
| **RMSE** | ~11,004 | Monthly rent predictions vary by ~₹11k on average. |
| **R² Score** | 0.5816 | Explains ~58% of variance. Lower than Buy model due to uncaptured factors (interior quality, landlord preference). |

---

## 6. Implementation in Production (`pages/owner.py`)

```python
# Load Rent Pipeline
models["rent_pipeline"] = PipelineModel.load("models/rental_pipeline")
models["rent_model"] = LinearRegressionModel.load("models/rental_model")

# Prediction
prediction = rent_model.transform(transformed)
rent = round(prediction.select("prediction").first()["prediction"])
```

---

## 7. Challenges & Constraints

1. **Subjectivity**: Rent is often determined by landlord urgency or specific tenant requirements, which are not in the data.
2. **Data Sparsity**: Fewer rental records compared to sales listings in the Admin dataset.
3. **Short-Term Fluctuations**: Rental markets change faster than property values; static models may lag.

---

## 8. Future Enhancements

- **Demand Scoring**: Integrate search volume data to adjust rent dynamically.
- **Tenant Profiling**: Incorporate tenant type (Family vs. Bachelors) as a feature, as it impacts rent.
- **Comparative Market Analysis (CMA)**: Show "Similar Properties Rented Recently" alongside the prediction.
