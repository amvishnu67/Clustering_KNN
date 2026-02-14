# FIFA 22 Player Segmentation with K-Means

This project builds a portfolio-grade clustering workflow on FIFA 22 player data with strong emphasis on validation, outlier handling, and interpretable insights.

## Project Goal
Cluster football players into meaningful groups using K-Means so scouting, salary benchmarking, and talent segmentation can be done in a data-driven way.

## Datasets
- `data/players_22.csv`
- Optional historical files are available in `data/` for future temporal analysis.

## Main Notebook
- `K-means_portfolio.ipynb`

The notebook includes:
1. Data loading and schema checks
2. Missing-value and duplicate validation
3. Outlier analysis using IQR
4. Feature engineering (`log1p` for `wage_eur` and `value_eur`)
5. Robust scaling to reduce outlier influence
6. K selection with multiple metrics:
   - Inertia (elbow)
   - Silhouette score
   - Davies-Bouldin index
7. Final K-Means training and scoring
8. Cluster profiling and top-player interpretation
9. PCA visualization of cluster separation
10. Export of report artifacts

## Why This Is Portfolio-Ready
- Reproducible workflow with fixed random seed
- Transparent preprocessing and model-selection logic
- Business-friendly interpretation (not just algorithm output)
- Saved artifacts for presentation and downstream use

## Outputs
Generated into `reports/`:
- `kmeans_k_selection_metrics.csv`
- `feature_outlier_report.csv`
- `players22_clustered.csv`

## How To Run
1. Create/activate a Python environment.
2. Install dependencies:
   ```bash
   pip install pandas numpy matplotlib seaborn scikit-learn jupyter
   ```
3. Open and run:
   - `K-means_portfolio.ipynb`

## Suggested Next Improvements
1. Compare K-Means with GMM and HDBSCAN.
2. Add temporal drift analysis across FIFA 15-22.
3. Move reusable logic into `src/` as functions/classes.
4. Add automated tests for preprocessing and metric outputs.

## Author Notes
The notebook is heavily commented by segment so each modeling decision is easy to explain in interviews and project walkthroughs.
