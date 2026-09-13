# E-Commerce Sales & Customer Analysis

An exploratory e-commerce analytics project using Python and Pandas to analyze customer behavior, product interactions, purchase patterns, and product-category performance across multiple datasets.

The project combines product interaction data with product metadata and separately analyzes customer purchasing behavior to build a broader view of how customers interact with an e-commerce environment.

## Project Overview

E-commerce analysis is not limited to completed purchases. Customer views, likes, purchase activity, demographic characteristics, location, previous purchases, and product categories can all provide useful information about customer behavior.

This project analyzes three complementary datasets:

1. **Customer interaction data** containing user-product interactions such as views, likes, and purchases.
2. **Customer purchase data** containing demographic and shopping characteristics.
3. **Product data** containing product information, selling prices, seller information, and categories.

The analysis follows two main tracks:

**Product Interaction Data + Product Metadata → Interaction & Category Analysis**

and

**Customer Purchase Data → Customer Segmentation**

This separation avoids incorrectly treating unrelated datasets as a single transactional table while still allowing each dataset to answer useful analytical questions.

---

## Objectives

The project explores questions such as:

- How are users interacting with products?
- What proportion of recorded interactions are views, likes, or purchases?
- Which product categories receive the most interactions?
- How does customer distribution vary across age groups?
- Which locations contain the largest customer groups?
- How do average purchase amounts differ across customer segments?
- How do previous purchase patterns vary between customer groups?
- How can product interaction data be enriched using product metadata?

---

## Tools & Technologies

- Python
- Pandas
- NumPy
- Matplotlib
- Jupyter Notebook

---

# Dataset

## 1. Product Interaction Data

The interaction dataset records user activity with individual products.

Important fields include:

| Column | Description |
|---|---|
| `user id` | Identifier for the user |
| `product id` | Identifier for the product |
| `Interaction type` | User action such as view, like, or purchase |
| `Time stamp` | Date and time of the interaction |

Example interaction types include:

- `view`
- `like`
- `purchase`

This dataset represents behavioral events rather than only completed sales.

---

## 2. Customer Details

The customer dataset contains demographic and purchasing information.

Important fields include:

| Column | Description |
|---|---|
| `Customer ID` | Customer identifier |
| `Age` | Customer age |
| `Gender` | Customer gender |
| `Item Purchased` | Purchased item |
| `Category` | Product category |
| `Purchase Amount (USD)` | Purchase value |
| `Location` | Customer location |
| `Season` | Purchase season |
| `Review Rating` | Customer review score |
| `Subscription Status` | Subscription status |
| `Discount Applied` | Whether a discount was applied |
| `Previous Purchases` | Number of previous purchases |
| `Payment Method` | Payment method used |
| `Frequency of Purchases` | Purchase frequency |

The dataset contains **3,900 customer records**.

---

## 3. Product Details

The product dataset contains product-level information used to enrich the interaction data.

Relevant fields used in the analysis include:

- Product ID
- Product Name
- Selling Price
- Product Category
- Amazon Seller status

The original product dataset contains more than **10,000 product records**, although several fields contain missing values and are not required for the analysis.

---

# Data Preparation

## Loading the Data

The three datasets are loaded separately using Pandas:

```python
dataset = pd.read_csv('E-commerece sales data 2024.csv')
df_customer = pd.read_csv('customer_details.csv')
df_product = pd.read_csv('product_details.csv')
```

---

## Product Data Integration

The product interaction dataset is enriched by joining it with product metadata using the product identifier.

```python
p = pd.merge(
    dataset,
    df_product,
    left_on='product id',
    right_on='Uniqe Id'
)

p = p.drop(['Uniqe Id'], axis=1)
df_product1 = p
```

This makes it possible to connect user interactions with attributes such as:

- product name
- selling price
- seller information
- product category

After the merge, the resulting interaction-product dataset contains approximately **2,600 matched records**.

---

## Selling Price Cleaning

Selling prices were originally stored as text values containing currency symbols.

The values were cleaned before numerical analysis:

```python
df_product1['Selling Price'] = (
    df_product1['Selling Price']
    .str.replace('$', '')
)
```

The cleaned values were then converted into numeric format so they could be used in aggregation and analysis.

---

# Customer Segmentation

Customer segmentation was performed to compare different demographic and geographic groups.

## Age Groups

Customers were divided into the following age segments:

```python
df_customer['Age Group'] = pd.cut(
    df_customer['Age'],
    bins=[0, 30, 40, 50, 60, 100],
    labels=['<30', '30-40', '40-50', '50-60', '60+']
)
```

For each segment, the analysis considers:

- number of customers
- average purchase amount
- total previous purchases

This provides more context than simply counting customers in each age group.

---

## Reusable Segmentation Function

A reusable function was created to analyze customer segments:

```python
def segmentation(df, column):
    result = df.groupby(column).agg({
        'Customer ID': 'count',
        'Purchase Amount (USD)': 'mean',
        'Previous Purchases': 'sum'
    }).reset_index()

    return result
```

This approach allows the same analytical logic to be reused across multiple customer dimensions.

---

## Customer Segmentation by Age

The first segmentation analyzes customer distribution across age groups.

<!--
Add the exported visualization to:

assets/customer-age-segmentation.png

Then replace this comment with:

![Customer Age Segmentation](assets/customer-age-segmentation.png)
-->

This analysis helps compare the relative size of each customer age segment.

---

## Customer Segmentation by Location

The analysis also groups customers by geographic location and ranks the locations by customer count.

The top locations are then visualized to identify geographic concentrations within the customer dataset.

<!--
Add the exported visualization to:

assets/customer-location-segmentation.png

Then replace this comment with:

![Customer Location Segmentation](assets/customer-location-segmentation.png)
-->

---

# Product & Interaction Analysis

## Product Category Analysis

After product interaction data is joined with product metadata, interactions can be analyzed by product category.

The analysis aggregates:

- number of user interactions
- selling-price values associated with those interactions

```python
segmentation_Category = df_product1.groupby('Category').agg({
    'Selling Price': 'sum',
    'user id': 'count'
}).reset_index()

segmentation_Category = segmentation_Category.sort_values(
    by='user id',
    ascending=False
)

Top_Category = segmentation_Category.head(10)
```

This produces a ranking of the most frequently interacted-with product categories.

> The aggregated selling-price value should be interpreted as the sum of product prices associated with interaction records, not automatically as company revenue, because the dataset also contains non-purchase interactions such as views and likes.

### Top Product Categories

<!--
Add the exported visualization to:

assets/top-product-categories.png

Then replace this comment with:

![Top Product Categories](assets/top-product-categories.png)
-->

---

## Interaction Analysis

User activity is grouped by interaction type:

```python
segmentation_Interaction = df_product1.groupby(
    'Interaction type'
).agg({
    'Selling Price': 'sum',
    'Category': 'count'
}).reset_index()
```

This allows the project to compare how frequently users:

- view products
- like products
- purchase products

### Interaction Distribution

<!--
Add the exported visualization to:

assets/interaction-distribution.png

Then replace this comment with:

![Interaction Distribution](assets/interaction-distribution.png)
-->

Understanding the distribution of interaction types provides a simple view of how users move beyond passive product exposure toward stronger engagement signals.

---

# Analytical Workflow

```text
                         E-Commerce Data
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
      Customer Dataset                 Interaction Dataset
              │                                 │
              │                                 ▼
              │                          Product Dataset
              │                                 │
              │                                 ▼
              │                           Merge on Product ID
              │                                 │
              ▼                                 ▼
     Customer Segmentation            Product Enrichment
              │                                 │
       ┌──────┴──────┐                 ┌────────┴────────┐
       │             │                 │                 │
       ▼             ▼                 ▼                 ▼
   Age Groups    Locations         Categories      Interaction Types
       │             │                 │                 │
       └──────┬──────┘                 └────────┬────────┘
              │                                 │
              ▼                                 ▼
       Customer Insights               Product Insights
              │                                 │
              └──────────────┬──────────────────┘
                             ▼
                    Analytical Findings
```

---

# Analysis Outputs

The project produces four main analytical views.

### 1. Customer Age Segmentation

Examines how customers are distributed across age groups and provides a foundation for comparing purchasing characteristics between demographic segments.

### 2. Geographic Customer Segmentation

Ranks locations according to customer representation and identifies the largest geographic customer groups in the dataset.

### 3. Product Category Performance

Identifies categories receiving the highest number of recorded user interactions.

### 4. User Interaction Distribution

Compares views, likes, and purchases to understand the composition of recorded customer-product activity.

---

# Skills Demonstrated

## Data Preparation

- CSV data loading
- data inspection
- missing-value assessment
- data-type conversion
- string cleaning
- numerical conversion

## Data Integration

- relational data thinking
- Pandas `merge()`
- joining datasets using product identifiers
- selecting relevant attributes after a join

## Data Analysis

- exploratory data analysis
- `groupby()`
- aggregation
- sorting and ranking
- demographic segmentation
- geographic segmentation
- product-category analysis
- customer behavior analysis

## Python

- Pandas
- NumPy
- reusable functions
- `pd.cut()`
- data transformation
- data aggregation

## Data Visualization

- Matplotlib
- pie charts
- horizontal bar charts
- categorical comparison
- customer segmentation visualization

---

# Repository Structure

Current repository structure:

```text
E-commerce-Sales-Analysis/
│
├── 004.ipynb
├── E-commerece sales data 2024.csv
├── customer_details.csv
└── product_details.csv
```

A cleaner portfolio structure could be:

```text
E-commerce-Sales-Analysis/
│
├── README.md
│
├── data/
│   ├── ecommerce_interactions.csv
│   ├── customer_details.csv
│   └── product_details.csv
│
├── notebooks/
│   └── ecommerce_sales_analysis.ipynb
│
└── assets/
    ├── customer-age-segmentation.png
    ├── customer-location-segmentation.png
    ├── top-product-categories.png
    └── interaction-distribution.png
```

---

# Limitations

Several limitations should be considered when interpreting this project.

### Different Analytical Sources

The customer dataset and product-interaction dataset represent complementary analytical sources, but they should not automatically be treated as one unified transactional database.

Customer segmentation is therefore analyzed independently from the product-interaction analysis.

### Interaction Value Is Not Revenue

The interaction dataset includes:

- views
- likes
- purchases

Therefore, summing product selling prices across every interaction does **not** represent actual company revenue.

A revenue calculation should only use confirmed purchase transactions and would require validation that each purchase event represents a completed transaction.

### Product Data Quality

Some fields in the source product dataset contain substantial missing data.

The analysis therefore focuses on fields with sufficient information for the selected analytical questions.

### Business Context

The datasets do not provide all information required for a complete commercial analysis, such as:

- cost of goods sold
- profit or margin
- inventory levels
- returns and refunds
- marketing acquisition cost
- complete order-level transaction information

The results should therefore be interpreted as exploratory customer and product interaction analysis rather than a complete financial performance analysis.

---

# Future Improvements

This project could be expanded by:

- filtering confirmed purchases before calculating sales value
- building a purchase conversion funnel from view → like → purchase
- calculating conversion rates by product category
- analyzing interaction trends over time
- comparing customer purchase frequency across segments
- analyzing payment-method preferences
- evaluating subscription and discount behavior
- adding repeat-customer analysis
- implementing the analysis in SQL
- developing an interactive Power BI dashboard
- reorganizing the repository into data, notebooks, and assets folders

A particularly valuable next step would be to calculate **category-level conversion rates** rather than relying only on interaction counts.

For example:

```text
Category
   ↓
Views
   ↓
Likes
   ↓
Purchases
   ↓
Purchase Conversion Rate
```

This would make the project more relevant to real e-commerce business analysis.

---

# Author

**Tsani Fauzan Alhakim**

Aspiring Data Analyst focused on Python, SQL, Power BI, Excel, data visualization, reporting, and business analytics.

- Portfolio: [tsanialhkm.github.io](https://tsanialhkm.github.io)
- GitHub: [github.com/TsaniAlhkm](https://github.com/TsaniAlhkm)
