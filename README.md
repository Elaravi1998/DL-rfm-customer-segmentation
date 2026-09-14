# 📊 RFM Customer Segmentation

> **Customer segmentation using Recency, Frequency, and Monetary (RFM) analysis on Online Retail transaction data.**

![Python](https://img.shields.io/badge/Python-3.x-blue?logo=python)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458?logo=pandas)
![NumPy](https://img.shields.io/badge/NumPy-Numerical%20Computing-013243?logo=numpy)
![Matplotlib](https://img.shields.io/badge/Matplotlib-Visualization-orange?logo=matplotlib)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter)
![Google Colab](https://img.shields.io/badge/Google%20Colab-Ready-F9AB00?logo=googlecolab)

---

## 🚀 Project Overview

This project performs **RFM (Recency, Frequency, Monetary) customer segmentation** using an Online Retail transaction dataset.

The goal is to transform raw customer transaction information into meaningful customer-level metrics and classify customers into different value segments.

The analysis follows three fundamental customer dimensions:

* 🕐 **Recency** — How recently a customer made a purchase
* 🔄 **Frequency** — How frequently a customer made purchases
* 💰 **Monetary** — How much money a customer spent

These three metrics are combined into a weighted **RFM Score**, which is then used to categorize customers into different segments such as:

* 🏆 Top Customer
* 💎 High Value Customer
* 📈 Mid Value Customer
* 📉 Low Value Customer
* ⚠️ Lost Customer

---

## 🎯 Project Objectives

The main objectives of this project are:

1. 📥 Load and inspect the Online Retail dataset.
2. 🧹 Handle missing values.
3. 🔄 Convert columns into appropriate data types.
4. 🕐 Calculate customer Recency.
5. 🔁 Calculate customer Frequency.
6. 💰 Calculate customer Monetary value.
7. 📊 Combine the three RFM components.
8. 🏅 Rank and normalize RFM metrics.
9. 🧮 Calculate a weighted RFM score.
10. 👥 Segment customers based on their RFM scores.
11. 📈 Visualize the customer segment distribution.

---

## 📂 Dataset

The notebook uses an **Online Retail transaction dataset**.

The dataset contains the following fields:

| Column        | Description                      |
| ------------- | -------------------------------- |
| `Invoice`     | Invoice/transaction identifier   |
| `StockCode`   | Product stock code               |
| `Description` | Product description              |
| `Quantity`    | Number of items purchased        |
| `InvoiceDate` | Date and time of the transaction |
| `Price`       | Product price                    |
| `Customer ID` | Customer identifier              |
| `Country`     | Customer's country               |

The notebook initially loads the dataset using:

```python
df = pd.read_csv(
    "online_retail_listing.csv",
    encoding="unicode_escape",
    delimiter=";"
)
```

---

## 🧹 Data Cleaning

The notebook performs several preprocessing steps before calculating RFM metrics.

### 1. 🔍 Missing Value Inspection

Missing values are checked using:

```python
df.isna().sum()
```

The dataset contains missing values in columns including:

* `Description`
* `Price`
* `Customer ID`
* `Country`

### 2. 🗑️ Removing Missing Values

Rows containing missing values are removed:

```python
df.dropna(inplace=True)
```

After this step, the notebook verifies that the remaining dataset contains no missing values.

---

## 🔄 Data Type Conversion

### 📅 Invoice Date

The `InvoiceDate` column is converted into a datetime format:

```python
df["InvoiceDate"] = pd.to_datetime(df["InvoiceDate"])
```

### 💰 Price

The dataset contains comma-formatted price values such as:

```text
6,95
```

The comma is removed before numerical conversion:

```python
df["Price"] = df["Price"].replace(",", "", regex=True)
df["Price"] = pd.to_numeric(df["Price"])
```

This prepares the price information for numerical calculations.

---

# 🧠 RFM Analysis

## 🕐 1. Recency

**Recency** measures how recently each customer made a purchase.

The notebook first finds the latest invoice date for every customer:

```python
df_Recency = df.groupby(
    by="Customer ID",
    as_index=False
)["InvoiceDate"].max()
```

A reference date is then obtained from the maximum invoice date:

```python
Recent_data = df_Recency["InvoiceDate"].max()
```

The number of days since the customer's most recent purchase is calculated as:

```python
df_Recency["Recency"] = df_Recency["InvoiceDate"].apply(
    lambda x: (Recent_data - x).days
)
```

### 📌 Interpretation

A smaller Recency value means the customer purchased more recently.

For example:

```text
Recency = 5
```

means the customer's latest purchase was 5 days before the reference date.

---

# 🔁 2. Frequency

**Frequency** represents how often a customer appears in the transaction data.

The notebook removes duplicate rows and counts invoice dates for each customer:

```python
df_frequency = (
    df.drop_duplicates()
      .groupby(
          by="Customer ID",
          as_index=False
      )["InvoiceDate"]
      .count()
)

df_frequency.columns = ["Customer ID", "Frequency"]
```

The resulting dataset contains:

```text
Customer ID
Frequency
```

### 📌 Interpretation

A higher Frequency generally indicates a customer has made more transactions.

---

# 💰 3. Monetary Value

The notebook first calculates the total price for each transaction:

```python
df["Total_price"] = df["Price"] * df["Quantity"]
```

The total monetary value for each customer is then calculated:

```python
df_Monitory = df.groupby(
    by="Customer ID",
    as_index=False
)["Total_price"].sum()
```

The resulting customer-level metric represents the customer's total transaction value.

> **Note:** The notebook names this column `Monitory`. In standard RFM terminology, this is normally called **Monetary**.

---

# 🔗 Combining RFM Metrics

The Recency, Frequency, and Monetary datasets are merged using `Customer ID`:

```python
RFM_df = df_Recency.merge(
    df_frequency,
    on="Customer ID"
)

RFM_df = RFM_df.merge(
    df_Monitory,
    on="Customer ID"
)
```

The columns are then renamed:

```python
RFM_df.columns = [
    "Customer ID",
    "Last_date",
    "Recency",
    "Frequency",
    "Monitory"
]
```

The resulting dataset contains customer-level RFM information.

---

# 🏅 RFM Ranking

The notebook calculates rankings for each RFM component.

### 🕐 Recency Rank

```python
RFM_df["R_rank"] = RFM_df["Recency"].rank(
    ascending=False
)
```

### 🔁 Frequency Rank

```python
RFM_df["R_Frequency"] = RFM_df["Frequency"].rank(
    ascending=True
)
```

### 💰 Monetary Rank

```python
RFM_df["R_Monitory"] = RFM_df["Monitory"].rank(
    ascending=True
)
```

These rankings are used as the basis for the normalized RFM calculations.

---

# 📊 Normalization

The ranking values are converted to a 0–100 style scale using the maximum rank.

### Recency

```python
RFM_df["R_rank_Normal"] = (
    RFM_df["R_rank"] /
    RFM_df["R_rank"].max()
) * 100
```

### Frequency

```python
RFM_df["R_Frequency_Normal"] = (
    RFM_df["R_Frequency"] /
    RFM_df["R_Frequency"].max()
) * 100
```

### Monetary

```python
RFM_df["R_Monitory_Normal"] = (
    RFM_df["R_Monitory"] /
    RFM_df["R_Monitory"].max()
) * 100
```

---

# 🧮 RFM Score

The project combines the three normalized components using weighted contributions:

```python
RFM_df["RFM_Score"] = (
    0.15 * RFM_df["R_rank_Normal"]
    + 0.28 * RFM_df["R_Frequency_Normal"]
    + 0.57 * RFM_df["R_Monitory_Normal"]
)
```

The resulting score is then scaled:

```python
RFM_df["RFM_Score"] *= 0.05
```

Finally, the values are rounded:

```python
RFM_df = RFM_df.round(2)
```

### ⚖️ Weight Distribution

| RFM Component | Weight |
| ------------- | -----: |
| 🕐 Recency    |    15% |
| 🔁 Frequency  |    28% |
| 💰 Monetary   |    57% |

The largest contribution in this notebook comes from the **Monetary** component.

---

# 👥 Customer Segmentation

Customers are classified according to their calculated RFM score.

The notebook uses the following segmentation logic:

```python
RFM_df["Customer Segment"] = np.where(
    RFM_df["RFM_Score"] > 4.5,
    "Top Customer",
    np.where(
        RFM_df["RFM_Score"] > 4,
        "High value Customer",
        np.where(
            RFM_df["RFM_Score"] > 3,
            "Mid Value Customer",
            np.where(
                RFM_df["RFM_Score"] > 1.6,
                "Low value Customer",
                "Lost Customer"
            )
        )
    )
)
```

## 🏆 Segment Definitions

| Segment                | RFM Score Condition | General Interpretation        |
| ---------------------- | ------------------: | ----------------------------- |
| 🏆 Top Customer        |             `> 4.5` | Highest-value customers       |
| 💎 High Value Customer |             `> 4.0` | Strong customer value         |
| 📈 Mid Value Customer  |             `> 3.0` | Moderate customer value       |
| 📉 Low Value Customer  |             `> 1.6` | Lower customer value          |
| ⚠️ Lost Customer       |            `<= 1.6` | Customers requiring attention |

---

# 📈 Visualization

The notebook visualizes the distribution of customer segments using a pie chart:

```python
from matplotlib import pyplot as plt

plt.pie(
    RFM_df["Customer Segment"].value_counts(),
    labels=RFM_df["Customer Segment"].value_counts().index,
    autopct="%.0f%%"
)

plt.show()
```

This provides a quick overview of the proportion of customers belonging to each segment.

---

# 🛠️ Technologies Used

| Technology          | Purpose                        |
| ------------------- | ------------------------------ |
| 🐍 Python           | Programming language           |
| 🐼 Pandas           | Data manipulation and analysis |
| 🔢 NumPy            | Numerical operations           |
| 📊 Matplotlib       | Data visualization             |
| 📓 Jupyter Notebook | Interactive development        |
| ☁️ Google Colab     | Notebook execution environment |

---

# 📁 Project Structure

```text
rfm-customer-segmentation/
│
├── 📓 RFM_Customer_Segmentation.ipynb
├── 📄 README.md
├── 📊 online_retail_listing.csv
└── 📁 images/
    └── customer_segments.png
```

> If the dataset is not included in the repository, update the notebook's data-loading path accordingly.

---

# ▶️ How to Run

## Option 1 — Google Colab ☁️

1. Open the `.ipynb` notebook in Google Colab.
2. Upload the required dataset.
3. Update the dataset path if necessary.
4. Run the notebook cells sequentially.

The original notebook uses Google Drive mounting:

```python
from google.colab import drive

drive.mount("/content/drive")
```

---

## Option 2 — Jupyter Notebook 💻

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/rfm-customer-segmentation.git
```

### 2. Navigate into the project

```bash
cd rfm-customer-segmentation
```

### 3. Install dependencies

```bash
pip install pandas numpy matplotlib jupyter
```

### 4. Launch Jupyter Notebook

```bash
jupyter notebook
```

### 5. Open the notebook

Open:

```text
RFM_Customer_Segmentation.ipynb
```

Run the cells from top to bottom.

---

# 📌 Key Workflow

```text
📥 Load Dataset
       ↓
🔍 Inspect Data
       ↓
🧹 Handle Missing Values
       ↓
🔄 Convert Data Types
       ↓
🕐 Calculate Recency
       ↓
🔁 Calculate Frequency
       ↓
💰 Calculate Monetary Value
       ↓
🔗 Merge RFM Metrics
       ↓
🏅 Rank RFM Components
       ↓
📊 Normalize Rankings
       ↓
🧮 Calculate Weighted RFM Score
       ↓
👥 Create Customer Segments
       ↓
📈 Visualize Segments
```

---

# 💡 Business Applications

RFM analysis can help businesses understand customer behavior and support decisions such as:

### 🏆 Customer Retention

Identify valuable customers and create strategies to retain them.

### 🎁 Personalized Marketing

Develop targeted campaigns based on customer purchasing behavior.

### 💰 Revenue Optimization

Identify customers contributing significant monetary value.

### 🔄 Re-engagement Campaigns

Identify customers with low activity who may need targeted offers.

### 👥 Customer Segmentation

Divide customers into actionable groups based on purchasing behavior.

---

# 🔬 Project Learning Outcomes

This project demonstrates practical experience with:

* 🐍 Python data analysis
* 🐼 Pandas DataFrames
* 🧹 Data cleaning
* 🔍 Missing-value handling
* 📅 Datetime processing
* 💰 Numerical data conversion
* 📊 GroupBy operations
* 🔗 DataFrame merging
* 🏅 Ranking techniques
* 📏 Normalization
* 🧮 Weighted scoring
* 👥 Customer segmentation
* 📈 Data visualization

---

# ⚠️ Important Notes

### 1. RFM vs Deep Learning

Although this notebook may be part of a broader learning path involving Deep Learning, the uploaded notebook itself is primarily a **data analytics / customer segmentation project using RFM methodology**.

It does **not** implement a neural network, CNN, RNN, Transformer, or other deep-learning model.

### 2. Monetary Naming

The notebook uses:

```text
Monitory
```

for the monetary metric.

The conventional RFM terminology is:

```text
Monetary
```

### 3. Dataset Path

The notebook currently references a Google Drive path. When moving the project to GitHub, it is better to use a relative path such as:

```python
df = pd.read_csv(
    "data/online_retail_listing.csv",
    encoding="unicode_escape",
    delimiter=";"
)
```

This makes the project easier for other users to reproduce.

---

# 🚀 Possible Future Improvements

The current notebook can be extended into a stronger portfolio project.

### 🔥 Recommended Improvements

* 📊 Add descriptive statistics
* 📈 Add RFM distribution plots
* 📉 Add histograms for Recency, Frequency, and Monetary values
* 📦 Add box plots to detect outliers
* 🌍 Analyze customer segments by country
* 🛍️ Analyze the most purchased products
* 📅 Analyze sales trends over time
* 🤖 Compare RFM segmentation with K-Means clustering
* 🧠 Build a machine-learning customer segmentation model
* 📊 Create an interactive dashboard using Plotly or Power BI
* 🎯 Develop customer-specific marketing recommendations

---

# 🌟 Potential Advanced Version

A future version of this project could combine traditional RFM analysis with machine learning:

```text
Online Retail Data
        ↓
Data Cleaning
        ↓
RFM Feature Engineering
        ↓
RFM Score
        ↓
K-Means Clustering
        ↓
Customer Segments
        ↓
Segment Profiling
        ↓
Marketing Recommendations
```

This would make the project more suitable for a **Data Science / Machine Learning portfolio**.

---

# 📊 Example RFM Feature Table

The final customer-level dataset contains information conceptually similar to:

| Customer ID | Last Date | Recency | Frequency | Monetary | RFM Score | Customer Segment       |
| ----------- | --------- | ------: | --------: | -------: | --------: | ---------------------- |
| Customer A  | Date      |      10 |        25 |     High |      High | 🏆 Top Customer        |
| Customer B  | Date      |      30 |        15 |   Medium |    Medium | 💎 High Value Customer |
| Customer C  | Date      |     100 |         8 |   Medium |    Medium | 📈 Mid Value Customer  |
| Customer D  | Date      |     250 |         3 |      Low |       Low | 📉 Low Value Customer  |
| Customer E  | Date      |     600 |         1 |      Low |  Very Low | ⚠️ Lost Customer       |

> The table above illustrates the concept; it is not presented as a direct output of the uploaded notebook.

---

# 🤝 Contributing

Contributions are welcome! 🎉

If you would like to improve this project:

1. 🍴 Fork the repository.
2. 🌿 Create a new branch.
3. ✏️ Make your changes.
4. 🧪 Test the changes.
5. 📤 Submit a pull request.

Example:

```bash
git checkout -b feature/improve-rfm-analysis
```

---

# 📜 License

This project is intended for **educational and portfolio purposes**.

If you add a dataset from an external source, make sure to follow the dataset's original licensing and usage requirements.

---

# ⭐ If You Find This Project Useful

If this project helped you understand **RFM analysis, customer segmentation, or Python data analytics**, consider giving the repository a ⭐ on GitHub!

---

## 🧠 Final Summary

This project demonstrates how raw online retail transactions can be transformed into actionable customer insights through:

**Recency + Frequency + Monetary → RFM Score → Customer Segmentation**

🚀 **From raw transactions to meaningful customer intelligence!**

---
