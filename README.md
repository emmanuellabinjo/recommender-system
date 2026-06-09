# 🛒 UK E-Commerce Recommender System

> A product recommender system built on 397,000 real UK retail transactions, implementing and comparing Item-Item Collaborative Filtering and SVD Matrix Factorisation approaches.

[![Python](https://img.shields.io/badge/Python-3.x-blue?logo=python)](https://python.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-green)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Active-brightgreen)]()

**[Notebook](recommender.ipynb)**

---

## Table of contents

- [Project overview](#project-overview)
- [Key results](#key-results)
- [Dataset](#dataset)
- [Project structure](#project-structure)
- [Methodology](#methodology)
- [Key findings](#key-findings)
- [How to run locally](#how-to-run-locally)
- [Limitations and future work](#limitations-and-future-work)
- [Acknowledgements](#acknowledgements)

---

## Project overview

Recommender systems power some of the most valuable products in the world — Netflix, Amazon, Spotify. This project builds and compares two classic recommender approaches on a real UK e-commerce dataset: Item-Item Collaborative Filtering using cosine similarity, and SVD Matrix Factorisation using the Surprise library. Both models generate personalised product recommendations for individual customers based on their purchase history.

---

## Key results

| Metric | Value |
|---|---|
| Transactions (after cleaning) | 397,884 |
| Unique customers | 4,339 |
| Unique products | 3,665 |
| User-item matrix sparsity | ~97% |
| SVD cross-validation RMSE | 4.39 |
| SVD cross-validation MAE | 2.18 |
| Latent factors | 50 |

---

## Dataset

### Source

| Dataset | Source | Rows (raw) | Rows (after cleaning) | Licence |
|---|---|---|---|---|
| Online Retail Dataset | [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/352/online+retail) | 541,909 | 397,884 | Public |

### Cleaning steps

- Removed 134,000 rows with missing CustomerID (guest checkouts)
- Removed cancelled orders (InvoiceNo starting with "C")
- Capped Quantity at 95th percentile (36 units) to remove extreme wholesale orders

### Key columns used

| Column | Description |
|---|---|
| `CustomerID` | Unique customer identifier |
| `StockCode` | Unique product identifier |
| `Description` | Product name |
| `Quantity` | Units purchased — used as implicit rating |

---

## Project structure

```
recommender-system/
│
├── recommender.ipynb       # Full pipeline: EDA, collaborative filtering, SVD, evaluation
├── .gitignore
└── README.md
```

> The raw Excel file is excluded from the repository. Download from UCI — see instructions below.

---

## Methodology

### 1. Data cleaning and preparation

541,909 raw transactions were cleaned by removing guest checkouts (missing CustomerID), cancelled orders, and extreme quantity outliers. The cleaned dataset of 397,884 transactions was aggregated into a user-item matrix where each cell represents the total quantity a customer purchased of a given product.

### 2. Item-Item Collaborative Filtering

Cosine similarity was computed between all 3,665 product vectors in the user-item matrix. For a given product, the most similar products are those purchased by the most overlapping set of customers.

For personalised recommendations, similarity scores are aggregated across all products a customer has purchased — products that are consistently similar to many of the customer's purchases rise to the top.

### 3. SVD Matrix Factorisation

SVD (Singular Value Decomposition) decomposes the user-item matrix into two lower-dimensional matrices:
- A **customer matrix** — describing each customer's latent preferences across 50 hidden factors
- An **item matrix** — describing each product's latent characteristics across the same 50 factors

To predict how much a customer will like an unbought product, the dot product of their latent vectors is computed. The model was trained using 3-fold cross-validation, achieving an RMSE of 4.39 on held-out data.

### 4. Comparison

Both models were run on the same customer to compare recommendation quality. The two approaches surface different products — Item-Item CF tends to find products frequently bought together, while SVD uncovers deeper latent preference patterns.

---

## Key findings

**The user-item matrix is 97% sparse** — most customers have only purchased a small fraction of the 3,665 available products. This sparsity is the core challenge of recommender systems and is why simple similarity approaches struggle without dimensionality reduction.

**Item-Item CF and SVD recommend different products** — this is expected and desirable. In production, combining both approaches (hybrid recommender) typically outperforms either alone.

**Quantity as an implicit rating works reasonably well** — without explicit star ratings, quantity purchased is a practical proxy for preference. However, bulk purchasing for business purposes can introduce noise.

**SVD RMSE of 4.39 is reasonable** — given quantities range from 1 to 36 after capping, an average error of 4.39 units is acceptable for a baseline model.

---

## How to run locally

### Prerequisites

- Python 3.10+
- Anaconda recommended

### 1. Clone the repo

```bash
git clone https://github.com/emmanuellabinjo/recommender-system.git
cd recommender-system
```

### 2. Install dependencies

```bash
pip install scikit-surprise implicit
pip install pandas numpy scikit-learn openpyxl
```

### 3. Download the data

Go to [UCI Online Retail Dataset](https://archive.ics.uci.edu/dataset/352/online+retail) and download `Online_Retail.xlsx`. Save it in the project folder.

### 4. Run the notebook

Open `recommender.ipynb` and run all cells in order.

---

## Limitations and future work

### Current limitations

- **Quantity as implicit feedback**: Quantity is an imperfect proxy for preference — a customer buying 20 units of a product for resale is not the same as genuinely preferring it.
- **Cold start problem**: New customers with no purchase history cannot receive personalised recommendations from either model.
- **No evaluation of recommendation quality**: RMSE measures quantity prediction accuracy, not whether the recommended products are genuinely useful. Metrics like Precision@K and NDCG@K would be more appropriate.
- **Static model**: The model does not update as new purchases occur.

### Planned improvements

- [ ] Implement Precision@K and NDCG@K evaluation metrics
- [ ] Build a hybrid model combining Item-Item CF and SVD scores
- [ ] Add ALS (Alternating Least Squares) via the implicit library for implicit feedback
- [ ] Handle cold start with a content-based fallback using product descriptions
- [ ] Deploy as a FastAPI endpoint returning recommendations as JSON
- [ ] Containerise with Docker for production deployment

---

## Acknowledgements

- [UCI Machine Learning Repository](https://archive.ics.uci.edu/) for the Online Retail dataset
- [Surprise](https://surpriselib.com/) library for SVD implementation
- Dr. Daqing Chen, School of Engineering, London South Bank University for the original dataset

---

## Licence

This project is licensed under the MIT Licence.

---

*Built as part of a data science portfolio. Questions or suggestions? Open an issue or connect on [LinkedIn](https://linkedin.com/in/emmanuel-labinjo).*