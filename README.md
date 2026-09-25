# Customer Segmentation using RFM Analysis

## Overview

This project segments customers of a UK-based online retailer into behavioral groups using RFM (Recency, Frequency, Monetary) analysis combined with K-means clustering. The goal is to move beyond treating every customer the same, and instead identify which customers are most valuable so marketing efforts can be targeted accordingly.

## Dataset

- **Source**: [Kaggle — ecommerce-data (carrie1)](https://www.kaggle.com/datasets/carrie1/ecommerce-data)
- **Description**: Real transaction-level data from a UK online retailer, covering ~1 year of sales
- **Raw size**: 541,909 transaction rows
- **After cleaning**: 406,829 rows (dropped rows with missing CustomerID and unneeded columns)
- **After filtering to UK market**: 361,878 rows (~89% of the cleaned dataset)

## Project Structure

- **Data Preparation** — cleaning, handling missing values, filtering to the UK market
- **RFM Calculation** — computing Recency, Frequency, and Monetary values per customer
- **Outlier Removal** — IQR-based filtering, applied separately to each metric
- **Normalization** — min-max scaling to bring all three metrics to a 0–1 range
- **Clustering** — K-means (K=4, chosen via the elbow method)
- **Cluster Analysis** — profiling and naming each segment
- **Visualization** — distribution plots, elbow plot, and a 3D PCA-based cluster plot

## Methodology

### Data Preparation
- Dropped `InvoiceNo`, `StockCode`, and `Description` (not needed for RFM)
- Dropped rows with missing `CustomerID` (an unidentified customer can't be segmented)
- Filtered to `Country == "United Kingdom"` since it accounts for the large majority of transactions, keeping the customer comparison fair

### RFM Calculation
- **Recency**: days since each customer's most recent purchase (inverted after normalization so higher = more recently active)
- **Frequency**: count of transaction line items per customer
- **Monetary**: total spend per customer (`Quantity × UnitPrice`, summed)

### Outlier Removal
Applied the IQR method independently to Recency, Frequency, and Monetary — any value outside `Q1 − 1.5×IQR` to `Q3 + 1.5×IQR` was removed. IQR was used over z-score because the underlying distributions (especially Monetary) are right-skewed rather than normal.

### Normalization
Min-max scaling brought all three metrics onto a comparable 0–1 range, since K-means relies on Euclidean distance and unscaled metrics (e.g. Monetary in currency vs. Recency in days) would otherwise dominate the clustering unevenly.

### Clustering
K-means was run with K values from 1–10, and the elbow method (plotting inertia vs. K) identified K=4 as the point where additional clusters stopped meaningfully improving cluster tightness.

## Results

Average normalized RFM scores per cluster:

| Segment | Recency | Frequency | Monetary |
|---|---|---|---|
| **True Friends** | 0.907 | 0.671 | 0.690 |
| **Butterflies** | 0.890 | 0.326 | 0.528 |
| **Barnacles** | 0.848 | 0.102 | 0.353 |
| **Strangers** | 0.312 | 0.098 | 0.336 |

- **True Friends** — recent, frequent, high spend → loyalty programs, exclusive offers, personalized communication
- **Butterflies** — recent and high spend, but infrequent → remarketing, limited-time promotions, complementary product suggestions
- **Barnacles** — engaged but low spend → upselling, product bundles, spending incentives
- **Strangers** — disengaged across all three metrics → brand awareness campaigns, reactivation offers

## Visualization

- Distribution plots (with KDE) for each normalized metric, used to check the effect of outlier removal and normalization
- Elbow plot to justify the choice of K=4
- 3D scatter plot of the four clusters, using PCA to project the RFM space for visualization

## Tech Stack

`Python`, `Pandas`, `NumPy`, `Matplotlib`, `Seaborn`, `Scikit-learn` (`KMeans`, `MinMaxScaler`, `PCA`)

## Usage

Or run it in Google Colab with no local install — upload `RFM_Analysis.ipynb` and `data.csv`, then run all cells in order.

## Key Design Decisions

- **IQR over z-score** for outlier removal, since RFM metrics are skewed rather than normally distributed
- **Min-max scaling over standardization**, for an interpretable 0–1 range across cluster averages
- **K-means over other clustering methods**, for efficiency and because RFM clusters tend to be roughly spherical in shape
- **Elbow method** used to select K, balancing cluster tightness against model simplicity

## License

MIT
