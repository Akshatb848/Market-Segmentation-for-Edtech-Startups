# EdTech Market Segmentation (District-level, India)

An exploratory notebook that groups Indian districts using public school-education and census indicators, as a first step toward deciding where an EdTech product might be marketed. It loads the 2015-16 district-wise education dataset from Kaggle, explores it with summary statistics and plots, and applies K-Means clustering (k = 3) to a 92-district subset, with an elbow plot and a PCA projection for inspection. It is a learning-stage exploratory analysis, not a validated segmentation model; see [Limitations](#limitations).

## Business problem

An EdTech startup entering the Indian market has to decide which regions to target first. Districts differ widely in population, urbanisation, literacy and school infrastructure. The idea behind this project is to group districts with similar profiles so that each group can be approached with a different go-to-market strategy.

## Dataset

- **Source:** [Education in India (Kaggle, rajanand)](https://www.kaggle.com/datasets/rajanand/education-in-india), district-wise school education statistics for academic year 2015-16.
- **Shape (from notebook output):** 680 rows (one per district) x 819 columns (`float64`: 13, `int64`: 803, `object`: 3). `df.isnull().sum()` shows no missing values in the columns displayed.
- **Columns used:** census indicators (`TOTPOPULAT`, `P_URB_POP`, `POPULATION_0_6`, `GROWTHRATE`, `SEXRATIO`, `OVERALL_LI`, `FEMALE_LIT`, `MALE_LIT`, `P_SC_POP`, `P_ST_POP`) and school counts by category for government (`SCH1G`-`SCH9G`) and private (`SCH1P`-`SCH9P`) schools, plus `SCHTOT` and `SCHTOTG`.
- **Not included in this repo.** Download the 2015-16 district-wise CSV from the Kaggle page above (it appears to be the file published there as `2015_16_Districtwise.csv`), rename it to `Edtech.csv`, and place it next to the notebook. The notebook reads `pd.read_csv('Edtech.csv')`.

## Approach

1. **Exploration:** `describe()`, `corr()`, `info()`, null counts, box plot of `CLUSTERS`, population vs. growth-rate scatter plots, a pair plot, and bar charts of schools, population and literacy by state.
2. **Subset selection:** the first 100 rows of the file (state codes 1-7, from Jammu & Kashmir through Delhi, in file order) and 29 numeric/categorical columns are kept. After `dropna()` the subset has 96 rows x 30 columns.
3. **Feature preparation:**
   - `OVERALL_LI` (overall literacy) is min-max scaled to [0, 1].
   - An `income` column is added from `tf.random.normal((10, 10))`, reshaped to 100 values and min-max scaled. **This is random noise, not real income data**, and no random seed is set.
   - After a further `dropna()`, 92 districts remain.
4. **Clustering:** `KMeans(n_clusters=3, random_state=0)` is fit on all remaining numeric columns (including the row `index` column) **without scaling**, apart from `OVERALL_LI` and `income`.
5. **Elbow method:** inertia is computed for k = 1-19, using only `income` and `OVERALL_LI`, after k = 3 had already been chosen.
6. **PCA:** the 92 x 31 feature matrix (including the cluster label) is min-max scaled and projected to 3 principal components; components 1 and 2 are plotted, coloured by literacy. PCA is used for visualisation only. It is not an input to the clustering.

## Results

What the executed notebook actually shows:

- **Data:** 680 districts x 819 columns loaded. 92 districts were clustered.
- **Clusters:** 3 K-Means clusters, shown as scatter plots against literacy, growth rate, total population and urban population. The notebook does not print cluster sizes, centroids or per-cluster profiles.
- **What drives the clusters:** in the "Total Population vs Overall literacy" plot, the three clusters split almost entirely along total population (roughly below 1 million, about 1-2 million, and above 2 million), with literacy spread across every cluster. This is expected, because the features were not scaled and `TOTPOPULAT` has by far the largest magnitude. The segments are therefore effectively district-population bands.
- **Elbow plot:** inertia drops steeply from k = 1 to about k = 3-4 and then flattens. No inertia values are printed, and the plot uses a different feature set from the clustering above.
- **No quantitative evaluation:** no silhouette score, Davies-Bouldin index or cluster stability check is computed.

## Limitations

- **Synthetic feature:** `income` is unseeded random noise generated with TensorFlow. Any pattern involving it, including the elbow plot, has no real-world meaning and changes on every run.
- **Scale dominance:** K-Means ran on unscaled features, so population counts dominate the distance metric. The clusters reflect district size, not a balanced mix of education and demographic indicators.
- **Partial coverage:** only 92 of 680 districts (7 northern states/UTs) are clustered. The results do not represent India as a whole.
- **Leakage into PCA:** the cluster label and the row index are included in the matrix passed to PCA.
- **Data quality:** some rows show values that look misaligned. For example, New Delhi's `P_URB_POP` equals its `GROWTHRATE` (-25.35). In addition, one `TOTPOPULAT` value is divided by 10 by hand (row 18) after the clustering frame had already been created, so this edit does not affect the clusters.
- **No business interpretation:** clusters are not profiled or turned into named personas or recommendations.
- **Reproducibility:** cells were run partly out of order (execution count 36 is missing), and the notebook was last executed in 2022 under Python 3.8.10. With current library versions, some cells may fail or behave differently. For example, `df.corr()` on a frame with text columns raises an error in pandas 2.x, `sns.barplot` called with positional x/y arguments raises an error in seaborn 0.12 and later, and `sns.distplot` is deprecated.

## How to run

The notebook was originally executed with Python 3.8.10. The dependency install has been checked on Python 3.11.

```bash
git clone https://github.com/Akshatb848/Market-Segmentation-for-Edtech-Startups.git
cd Market-Segmentation-for-Edtech-Startups

python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# Download the dataset from Kaggle and save it as Edtech.csv in this folder (see Dataset)
jupyter notebook EdTechMarketSegmentation.ipynb
```

`requirements.txt` sets only minimum versions. If cells fail on newer pandas or seaborn releases (see Limitations), install older versions, for example `pip install "pandas<2" "seaborn<0.12"` on Python 3.8-3.10.

## Project structure

```
.
├── EdTechMarketSegmentation.ipynb   # EDA, preprocessing, K-Means, elbow plot, PCA (with saved outputs)
├── requirements.txt                 # Third-party packages imported by the notebook, plus Jupyter
└── README.md
```

## Author

Akshat Banga - [LinkedIn](https://www.linkedin.com/in/akshat-banga-6574aa170/) - [GitHub](https://github.com/Akshatb848)
