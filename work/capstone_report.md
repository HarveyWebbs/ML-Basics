Capstone Report —


**Author:** Abdullah Zia
**Lane:** Lane 1: Ranking Signal Analysis
**Repo:** https://github.com/HarveyWebbs/ML-Basics
**Date:** August 28, 2026


0. Abstract
This research investigates which historical content signals—such as content age, update staleness, and CMS template type—are directionally associated with organic search visibility. The analysis was conducted on a pseudonymized 79-million-row cloud data warehouse, measuring content performance over a fixed 31-day window (March 2026). To isolate these signals, I compared a fixed-rule business heuristic against a Random Forest Regressor, utilizing a grouped-client data split to prevent domain memorization. The baseline heuristic actually outperformed the machine learning model, proving that complex algorithms can severely overfit to domain authority when generalizing to unseen clients. Despite this, feature importances successfully quantified that staleness and age account for over 96% of content performance variance compared to CMS templates. Ultimately, the model's logic was translated into a cost-evaluated, ranked action playbook that provides decision-support for editorial teams allocating content refresh resources.


**1. Problem framing**
This analysis provides decision-support for editorial resource allocation. The unit of analysis is a single pseudonymized content item (a webpage) evaluated over a fixed 1-month period. The final output is a ranked queue of content URLs mapped to specific action labels (e.g., COMPREHENSIVE_REFRESH) and reason codes.
A human editor uses this queue to prioritize which articles receive writing budget. The cost of a wrong call is high: updating content requires expensive human writer hours ($150+ per article). If we recommend refreshing content that has no underlying potential, we waste editorial budget for zero SEO gain. Machine Learning helps here because fixed rules ignore the contextual, non-linear decay curves of different content types.


**2. Data safety**
This analysis utilized the dim_content and fact_content_daily_performance tables from the FlyRank warehouse, specifically partitioned to month=2026-03.
Exclusions and Leakage Prevention:
gsc_impressions was deliberately excluded from the feature set. Predicting March clicks using March impressions is target leakage.
The final month (_sample, June 2026) was strictly excluded to serve as an untouched, sealed holdout.
client_hash_id was restricted to grouping logic for the data split and was never used as a predictive feature to prevent the model from memorizing domain authority.
A time-leakage audit verified that no content with a "negative age" relative to March 1, 2026, was included in the dataset.
I confirm that no client-identifying details, URLs, or private queries appear anywhere in the work/ directory.


**3. Baseline**
Before modeling, I built a transparent business rule: the "Stale Evergreen Refresh" heuristic. It flagged evergreen content where age_days > 180 and days_since_updated > 180.
To ensure a fair mathematical comparison, this baseline was translated into a Linear Regression model utilizing only the single days_since_updated feature. It was evaluated on the exact same metric (log-transformed clicks) and the exact same unseen test split as the ML model. The baseline established our foundational R-Squared and RMSE, representing standard SEO industry logic.


**4. Model / analysis**
I implemented a Random Forest Regressor. This method perfectly fits Lane 1 because SEO signals are highly interactive; the impact of staleness changes drastically depending on whether a page is an "Article" or a "Glossary".
Features used: content_type (One-Hot Encoded), age_days (continuous), and days_since_updated (continuous).
Features excluded on purpose: Domain hashes and engagement metrics.
Target Proxy: The natural logarithm of Google Search Console clicks log1p(total_gsc_clicks). This log transformation normalizes the severe power-law skew inherent to web traffic, preventing the model from over-indexing on the top 1% of viral pages.


**5. Evaluation**
I utilized a Grouped Split by Client (GroupShuffleSplit) to prevent the model from artificially inflating its score by memorizing client domain authority.
Results: The Linear Baseline achieved an RMSE of 0.8154 (R2: 0.0362). The Random Forest model achieved a worse RMSE of 1.0226 (R2: -0.5158).
Error Analysis: A negative R-Squared on an honest grouped split is a highly valid finding. It proves the Random Forest overfit to the historical traffic baselines of the training clients and failed to generalize to completely unseen domains. The simple, fixed baseline rule generalized better.
Furthermore, reviewing the top residuals revealed future data leakage from the dimension table. Several of the highest errors had negative days_since_updated values (e.g., -103 days). This indicates the content was published before March, but updated after our March evaluation window, confusing the historical prediction.


**6. Interpretation**
Despite the model's failure to generalize traffic volume across unknown domains, extracting the Feature Importances provided vital directional insights for the SEO team.
Observations:
The model mathematically confirmed that days_since_updated (67.8% importance) and age_days (28.2% importance) are the overwhelming drivers of content visibility signals.
Template format was remarkably insignificant. content_type_feedly article and keyword article accounted for less than 4% importance combined, and comparison article had near-zero impact.
Conclusion: Editorial teams should not worry about which CMS template they use; they should focus entirely on the maturity and freshness of the text itself.


**7. Recommendation**
The output of this analysis is the Content Action Playbook, exported to work/outputs/w07_action_playbook_queue.csv.
How to use: An SEO editor should pull the highest-ranking rows tagged COMPREHENSIVE_REFRESH, manually verify the brand intent of the URL, and assign the update to a writer.
Confidence and Limits: These recommendations are strictly directional decision-support tools. An algorithm cannot read the room or know if a product was discontinued. Human-in-the-loop review is mandatory. Furthermore, restricted content (Press Releases, Legal documents) is hard-coded into a "No-Go List" and excluded from automation entirely.


**8. Reproducibility**
Anyone with repository access can fully reproduce this research:
Clone the repository and install dependencies (pip install pandas numpy scikit-learn duckdb matplotlib).
Supply a standard Hugging Face Read Token as a Colab Secret (HF_TOKEN).
Execute notebooks w02 through w07 sequentially.
All data splits and algorithm initializations rely on random_state=42.
The baseline monitoring metrics generated in Week 7 are preserved in work/outputs/w07_monitoring_baselines.json.


**9. Acknowledgments & data credit**
This research was built on the FlyRank ML Internship dataset. Access to the warehouse and schema documentation is provided by https://flyrank.ai.
