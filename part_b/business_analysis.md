# Business Case Analysis

## B1. Problem Formulation

### (a) Machine Learning Problem Formulation

The goal of this problem is to predict which promotion type will maximise the number of items sold in each store during a given month.

* **Target Variable:** `items_sold`
* **Candidate Input Features:**

  * Store attributes: store size, location type, footfall, competition density, customer demographics
  * Promotion details: promotion type
  * Calendar features: month, weekend indicator, festival indicator
  * Historical sales patterns: previous month sales, seasonal trends

This is a **supervised machine learning regression problem** because the target variable, `items_sold`, is a continuous numerical value. The model learns from historical data to predict the expected sales volume under different promotion strategies.

Once predicted sales for each promotion are obtained, the company can choose the promotion with the highest expected sales for each store-month combination.

---

### (b) Why Items Sold is Better Than Total Revenue

Using **items sold** is more reliable than **total sales revenue** because revenue can be influenced by factors unrelated to promotion effectiveness, such as:

* Price differences between products
* Discounts reducing revenue even if sales volume increases
* Premium products generating high revenue with fewer units sold

For example, a Flat Discount may increase the number of items sold but reduce average revenue per item. If the company uses revenue as the target, it may underestimate the success of that promotion.

This illustrates the broader ML principle that **the target variable should directly represent the business objective**. Since the objective is to maximise unit sales through promotions, `items_sold` is the most aligned target variable.

---

### (c) Alternative to One Global Model

Instead of one single global model, a better approach would be a **hierarchical or segmented modelling strategy**.

For example:

* Build separate models for **urban, semi-urban, and rural stores**
* Or include **store-specific effects** using store identifiers or mixed-effect models

This is important because customer behaviour differs by location. A promotion that performs well in urban stores may not perform similarly in rural stores.

Segmented models improve prediction accuracy by allowing the model to capture differences in customer response patterns across store groups.

---

## B2. Data and EDA Strategy

### (a) Joining the Data

The data comes from four tables:

1. **Transactions** – sales records
2. **Store attributes** – store size, location, competition
3. **Promotion details** – promotion type per store-month
4. **Calendar** – weekend and festival indicators

These tables would be joined using:

* `store_id` to merge transactions with store attributes
* `promotion_id` or store-month keys to merge promotions
* `transaction_date` to merge calendar data

The **grain** of the final dataset should be:

> **One row = one store for one month**

Before modelling, transaction-level data should be aggregated to monthly store-level metrics such as:

* Total items sold
* Total footfall
* Average basket size
* Number of promotion days
* Festival/weekend counts

This aggregation aligns the data with the monthly promotion decision-making process.

---

### (b) Exploratory Data Analysis

Before modelling, the following EDA should be performed:

#### 1. Promotion-wise sales comparison

Use boxplots or bar charts to compare `items_sold` across promotion types.

This helps identify which promotions generally perform better and whether some promotions show high variability.

---

#### 2. Time series sales trends

Plot monthly sales trends by store or promotion.

This helps detect seasonality, such as higher sales during festival periods, which can be used to engineer time-based features.

---

#### 3. Correlation heatmap

Check correlations between numerical variables such as footfall, competition density, and items sold.

This helps identify important predictors and detect multicollinearity.

---

#### 4. Sales by store segment

Compare sales distributions across urban, semi-urban, and rural stores.

This helps determine whether store segmentation is necessary.

---

The findings from these analyses guide:

* Feature creation
* Segmentation strategy
* Choice of model complexity

---

### (c) Handling Promotion Imbalance

If 80% of data has no promotion, the model may learn to favour the “no promotion” case and underestimate the effect of actual promotions.

This imbalance can lead to:

* Poor learning for minority promotion types
* Biased predictions toward non-promotion periods

To address this:

* Use balanced sampling or weighted loss functions
* Ensure sufficient examples of each promotion in training
* Evaluate performance separately by promotion type

This helps the model learn the impact of promotions more accurately.

---

## B3. Model Evaluation and Deployment

### (a) Train-Test Split and Metrics

Since the data is monthly and time-ordered, the split should be **chronological**:

* Train on the first 2.5 years
* Test on the last 6 months

A random split is inappropriate because it mixes future data into training, causing **data leakage** and unrealistic performance estimates.

Evaluation metrics:

#### RMSE

Measures average prediction error, giving larger penalties to large mistakes.

Useful for identifying whether the model makes large forecasting errors.

---

#### MAE

Measures average absolute error.

Easy to interpret in business terms—for example, “the prediction is off by 20 items on average.”

---

#### R²

Measures the proportion of variance explained.

Useful for understanding how well the model captures sales patterns overall.

Together, these metrics provide a balanced evaluation of accuracy and stability.

---

### (b) Explaining Different Recommendations with Feature Importance

Feature importance shows which factors influenced the recommendation most.

For Store 12:

* In **December**, the model may recommend Loyalty Points Bonus because:

  * festival season
  * higher customer footfall
  * loyal customers respond well to points

* In **March**, it may recommend Flat Discount because:

  * lower seasonal demand
  * price sensitivity is higher

Using feature importance tools such as:

* Random Forest feature importance
* SHAP values

we can show the marketing team which factors influenced the recommendation in each month.

This improves trust and helps explain why recommendations change over time.

---

### (c) Deployment Process

The deployment process would involve:

#### 1. Save the trained model

Use serialization tools such as `joblib` or `pickle` to save the trained preprocessing pipeline and model.

---

#### 2. Prepare monthly data

At the start of each month:

* collect store data
* merge promotion and calendar features
* apply the same preprocessing steps used during training

---

#### 3. Generate predictions

For each store, predict expected `items_sold` for each promotion type and select the best-performing promotion.

---

#### 4. Monitor performance

Track:

* prediction error (RMSE, MAE)
* changes in sales patterns
* drift in feature distributions

If prediction errors rise significantly or feature distributions shift, retraining should be triggered.

This ensures the model remains accurate as customer behaviour changes over time.

