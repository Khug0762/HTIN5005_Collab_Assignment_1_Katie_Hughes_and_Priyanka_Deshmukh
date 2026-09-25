Data Science Reproduction Part B
===================================

PURPOSE
-------
Methodological reproduction of Khmaissia et al. (2020) using Greater
Sydney Postal Areas (POAs), ABS 2021 Census data and NSW Health
COVID-19 case data.

PUBLIC INPUT DATA
-----------------
1. NSW Health COVID-19 cases by location
   Landing page:
   https://data.nsw.gov.au/data/dataset/covid-19-cases-by-location

   File used:
   confirmed_cases_table1_location.csv

2. ABS 2021 Census General Community Profile
   Geography: Postal Areas (POA)
   State: New South Wales
   Header: short-header

   Landing page:
   https://www.abs.gov.au/census/find-census-data/datapacks

   File used:
   2021_GCP_POA_for_NSW_short-header.zip

3. ABS 2021 Postal Area boundaries
   GDA2020 Shapefile

   Landing page:
   https://www.abs.gov.au/statistics/standards/
   australian-statistical-geography-standard-asgs/
   edition-3-july-2021-june-2026/access-and-downloads/
   digital-boundary-files

   File used:
   POA_2021_AUST_GDA2020_SHP.zip

HOW TO RUN
----------
1. Open Data_Science_Reproduction_Part_B.ipynb in Google Colab.
2. Run all cells sequentially from Step 1 to the end.
3. Step 2 attempts to download all three source datasets automatically.
4. If an automatic download is blocked, upload the three files when
   prompted by Step 2.
5. Continue running all cells in order.
6. Results, figures, trained objects and this README are written under:
   /content/Data_Science_Reproduction_Part_B/

STUDY DESIGN
------------
Study area: Greater Sydney
Final cohort: 247 POAs
Study window: 2021-06-16 to 2021-10-10 inclusive (117 days)

ABS Postal Areas (POAs) are used as an approximation of postcode
boundaries to link Census and NSW Health postcode-level information.

CORE MODELLING PIPELINE
-----------------------
247 Greater Sydney POAs
-> 28 predefined candidate predictors
-> z-score standardisation using StandardScaler
-> LASSO + RReliefF feature selection
-> union of selected features
-> K-means on the selected standardised features
-> primary k = 3
-> rank cluster labels by mean COVID-19 Increase Rate (IR)
-> cluster feature-profile heatmap using mean standardised values
-> category-level 1-D t-SNE after clustering for interpretation
-> temporal and geographic analyses
-> k = 6 sensitivity analysis

FEATURE SELECTION
-----------------
LASSO:
- 5-fold cross-validation
- alpha selected by LassoCV: 0.0010911375457
- features retained when coefficient is non-zero
- number retained: 6

RReliefF:
- source repository: https://github.com/AmritSe/RReliefF
- source file:
  https://raw.githubusercontent.com/AmritSe/RReliefF/master/relieff.py
- updates = "all"
- k = 10
- sigma = 30
- selection rule: weight > 0
- number retained: 16

Overlap between LASSO and RReliefF: 4
Final union used for K-means: 18

CLUSTER SELECTION
-----------------
Candidate k values: 2 to 10
Highest silhouette k: 2
Highest silhouette score: 0.2819
Primary k: 3
Primary-k silhouette score: 0.2695

The silhouette coefficient is highest at k = 2. Inspection of the
elbow curve indicates diminishing improvements in inertia after
approximately k = 3. Because the silhouette score for k = 3 remains
close to the maximum and the elbow supports a three-cluster solution,
k = 3 is retained for the primary analysis.

PRIMARY CLUSTER SUMMARY
-----------------------
         n_postcodes  mean_covid_IR  median_covid_IR  mean_cases  median_cases  mean_cases_per_1000  median_cases_per_1000
cluster                                                                                                                   
0                 75         0.0033           0.0000    100.7600          66.0               5.0024                 4.4194
1                116         0.0046           0.0000     74.9397          27.5               4.1066                 2.9234
2                 56         0.0299           0.0085    699.6071         480.5              23.3938                18.1018

EXPLORATORY IR COMPARISONS
--------------------------
One-way ANOVA:
F = 35.273825
p = 3.49794e-14

Kruskal-Wallis:
H = 48.879070
p = 2.43246e-11

These comparisons are exploratory rather than independent validation
because COVID-19 IR is used during supervised feature selection and
later to rank the cluster labels.

IMPORTANT IMPLEMENTATION NOTES
------------------------------
- COVID-19 IR is not an input to K-means.
- K-means is fitted directly to the selected standardised predictors.
- IR is used for supervised feature selection and post-clustering
  ranking of cluster labels.
- The cluster profile heatmap displays mean z-scores for the 18 selected
  features within each primary cluster and is used to interpret the
  characteristics that distinguish the clusters.
- Category-level t-SNE is performed after clustering and is used only
  for interpretation.
- t-SNE coordinate signs and magnitudes should not be interpreted as
  original feature weights or effect directions.
- Raw population is not a K-means predictor. It is used to calculate
  cases per 1,000 population and population density.
- The external RReliefF implementation must be cited because borrowed
  source code is used.

OUTPUTS
-------
Results:
/content/Data_Science_Reproduction_Part_B/results

Figures:
/content/Data_Science_Reproduction_Part_B/submission_outputs/figures

Trained objects:
/content/Data_Science_Reproduction_Part_B/submission_outputs/models
