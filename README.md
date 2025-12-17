# Movie Analysis Notebook

## Overview

This repository contains a Jupyter notebook for exploratory data analysis (EDA) and simple analytics on a movies dataset. The notebook covers data cleaning, feature engineering, and analyses such as total revenue by genre and average rating by genre. It is written for reproducible, project-style analysis and is suitable for presentation or further extension.

## Files in this project

* `notebook.ipynb` — Main analysis notebook (primary).
* `IPNYB Instructions.ipynb` — Supporting instructions / helper notebook.
* `README.md` — This file (project overview and instructions).


## Dataset (expected schema)

The notebook uses a DataFrame with the following columns (example):

* `movie_id` (object)
* `title` (object)
* `original_title` (object)
* `year` (int)
* `runtime_minutes` (float)
* `genres` (object) — comma-separated genre strings (e.g. `Action,Drama`)
* `studio` (object)
* `domestic_gross` (float)
* `foreign_gross` (float)
* `averagerating` (float)
* `numvotes` (int)
* `Total_revenue` (float) — `domestic_gross + foreign_gross` (may be precomputed in the notebook)

## Setup / Requirements

Recommended Python version: **3.8+**

Install required packages (create virtualenv first if desired):

```bash
pip install -r requirements.txt
```

Example `requirements.txt`:

```
pandas>=1.3
matplotlib
jupyterlab
openpyxl    # if reading/writing Excel files
seaborn      # optional, for nicer plots
```

## How to run

1. Open the notebook in Jupyter or JupyterLab:

```bash
jupyter lab notebook.ipynb
```

2. Run cells top-to-bottom.
3. If data is external, update the file path in the notebook (`pd.read_csv(...)` or similar).

## Key preprocessing steps (performed in the notebook)

1. **Inspect columns**

```python
df1.columns
```

2. **Handle missing genres** — fill with mode or `Unknown`:

```python
genre_mode = df1['genres'].mode()[0]
df1['genres'].fillna(genre_mode, inplace=True)
# or
# df1['genres'].fillna('Unknown', inplace=True)
```

3. **Compute total revenue** (if not present):

```python
df1['Total_revenue'] = df1['domestic_gross'].fillna(0) + df1['foreign_gross'].fillna(0)
```

4. **Split multi-genre strings and explode** — this ensures each row maps to one genre:

```python
# split (only if genres are comma-separated strings)
df1['genres'] = df1['genres'].str.split(',')
# explode
df1_exploded = df1.explode('genres')
# clean whitespace
df1_exploded['genres'] = df1_exploded['genres'].str.strip()
```

## Example analyses included

* **Total revenue per genre** (handles multi-genre movies):

```python
revenue_by_genre = df1_exploded.groupby('genres')['Total_revenue'].sum().sort_values(ascending=False)
```

* **Average rating per genre** (with movie counts):

```python
ratings_by_genre = (
    df1_exploded.groupby('genres')['averagerating']
    .agg(mean_rating='mean', movie_count='count')
    .sort_values('mean_rating', ascending=False)
)
# Optionally filter to genres with enough samples
ratings_by_genre_filtered = ratings_by_genre.query('movie_count >= 10')
```

* **Combined table**: mean rating, sum revenue, and movie count together

```python
summary = df1_exploded.groupby('genres').agg(
    total_revenue=('Total_revenue', 'sum'),
    mean_rating=('averagerating', 'mean'),
    movie_count=('movie_id', 'count')
).sort_values('total_revenue', ascending=False)
```

## Visualization examples (matplotlib)

Bar chart of total revenue by genre:

```python
import matplotlib.pyplot as plt
revenue_by_genre.head(15).plot(kind='bar', figsize=(10,6))
plt.title('Top 15 Genres by Total Revenue')
plt.ylabel('Total revenue')
plt.tight_layout()
plt.show()
```

Scatter plot of mean rating vs total revenue (bubbles size = movie_count):

```python
plt.figure(figsize=(8,6))
plt.scatter(summary['mean_rating'], summary['total_revenue'], s=summary['movie_count']*2, alpha=0.6)
plt.xlabel('Mean Rating')
plt.ylabel('Total Revenue')
plt.title('Genre: Rating vs Revenue (bubble size = movie count)')
plt.grid(True)
plt.show()
```

## Tips, caveats and next steps

* **Multi-genre handling**: exploding genres duplicates rows for multi-genre movies — this is intentional and required to attribute movies to each genre. If you want to avoid double-counting revenue, consider allocating a movie's revenue proportionally across its genres.
* **Missing values**: check `domestic_gross` and `foreign_gross` for missingness — filling with 0 affects totals.
* **Outliers**: very large blockbusters can dominate sums — consider median or trimmed means for robust comparison.
* **Sample size**: always check `movie_count` before trusting means for rare genres.

## Reproducibility & export

* Save final cleaned table:

```python
df1_exploded.to_csv('movies_cleaned_exploded.csv', index=False)
```

* Save summary table for reports:

```python
summary.to_csv('genre_summary.csv')
```

## Authors

Created by: claire Njeri, Yvonne Rajula, Abdullahi Hassan, Lauren Kuria, Issac Macharia and Dahir Ahmed
If you want changes to this README (add badges, license, or export as README.md), tell me and I'll update it.

---

*End of README*
