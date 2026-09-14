# Capstone Report — Content Archetypes for Content Review Prioritization

* **Author:** Priyansh Srivastava
* **Lane:** Data Science / Machine Learning
* **Repo:** https://github.com/Priyansh-rath18/flyrank-internship-
* **Date:** 2026-09-14

## 0. Abstract

This study asks whether published content can be grouped into useful performance archetypes to support content-review prioritization. The analysis uses the public FlyRank ML Internship dataset, combining content attributes with search and website-performance aggregates. DuckDB and Pandas were used for data preparation and feature engineering, followed by standardized K-Means clustering. The six-cluster analysis produced descriptive groups named Star Performer, Authority Pillar, Visible Underconverter, Buried Thin Page, Hidden Engagement Gem, and Dormant, with proposed actions such as Protect, Improve, Rewrite, Merge, Monitor, and Prune. The output is a directional decision-support framework for helping editors prioritize content review, not a proven prediction of future rankings or traffic.

## 1. Problem framing

### Research question

Can published content be grouped into meaningful performance archetypes using observed search visibility, engagement, authority, and content-level signals?

### Decision supported

The analysis supports a content-review prioritization decision. It helps identify which pages may deserve attention first and what type of manual review may be appropriate.

### Unit of analysis

The unit of analysis is a published content item identified by `content_hash_id`.

The identifier is used to join content attributes with performance aggregates. It is not used as a machine-learning feature.

### Output

The analysis produces:

* A cluster assignment for each content item.
* A descriptive archetype label.
* A proposed editorial action.
* Cluster-level performance profiles.
* Charts showing archetype distribution and the relationship between visibility and ranking position.

### Human action

An editor can use the output to select pages for further investigation, such as reviewing search intent, content depth, internal links, engagement, or possible consolidation.

### Cost of a wrong decision

A wrong recommendation could lead to unnecessary rewrites, loss of useful content, poor consolidation decisions, or missed opportunities to improve valuable pages. For this reason, the output is intended to support human review rather than make automatic editorial decisions.

### Why data and ML help

A large content inventory can contain many pages with different performance patterns. A multi-feature clustering approach can summarize these patterns more broadly than a single metric such as impressions. However, the usefulness of the groups must be checked against a transparent baseline and future performance.

## 2. Data safety

### Dataset

The analysis uses the public FlyRank internship warehouse:

`FlyRank/internship-warehouse`

The dataset is accessed through Hugging Face using DuckDB and Parquet files.

### Tables used

| Table                                           | Purpose                                                                             |
| ----------------------------------------------- | ----------------------------------------------------------------------------------- |
| `dim_content.parquet`                           | Content attributes, search demand, authority-related fields, and publication status |
| `fact_content_query_90d.parquet`                | Query-level search-performance aggregates over a 90-day window                      |
| `fact_content_daily_performance_sample.parquet` | Daily website and search-performance sample                                         |

### Date windows

The query-performance table contains 90-day fields and comparisons between the last 30 days and the previous 30 days.

The exact calendar start and end dates of the dataset release have not yet been verified from the source metadata. They must be recorded from the actual release before submission.

The daily performance sample was aggregated across its available records. The precise calendar coverage and its alignment with the query-performance window should be confirmed.

### Filtering

The analysis retained records where:

* `is_published == True`
* `is_deleted == False`

Unpublished and deleted content was excluded because the research question concerns active published content and its review priorities.

### Features used

The analysis used:

* Log-transformed impressions and clicks.
* 90-day click-through rate.
* Weighted average search position.
* Click momentum between recent and previous 30-day periods.
* Query count.
* Rare-impression share.
* Engagement rate.
* Average engagement time.
* Log-transformed scroll events.
* Log-transformed backlinks.
* Log-transformed search volume.
* Word count.

### Deliberately excluded fields

The following fields were not used as clustering features:

* `content_hash_id`: used only for joining and identifying content records.
* `is_published`: used for filtering.
* `is_deleted`: used for filtering.
* `content_type`: retained as metadata but not included in the current feature matrix.
* `main_intent`: retained as metadata but not included in the current feature matrix.
* `competition_level`: retained as metadata but not included in the current feature matrix.
* `char_count`: available in the content table but not included in the current feature matrix.
* `trend_direction` and `trend_pct`: not used as features. If present in another assignment table, they must be excluded unless their timing and meaning are verified.
* Private queries, client-identifying information, and private URLs: not used or exposed in the report.

### Leakage considerations

The current analysis is descriptive and uses observed performance signals to form clusters. It does not claim to predict a future outcome.

A future predictive evaluation must ensure that features are calculated only from information available before the evaluation period. In particular, future clicks, impressions, trend labels, and other outcome-derived fields must not enter the feature matrix.

### Public-safety confirmation

The report is intended to contain only public dataset information, aggregate analysis, and pseudonymous content identifiers. A final repository scan is required to confirm that no client-identifying details, private queries, URLs, or access tokens appear anywhere under `work/`.

## 3. Baseline

### Baseline definition

The proposed baseline is a transparent ranking of content by 90-day impressions.

Pages are ordered from highest to lowest observed impressions. The highest-ranked items form the baseline priority group under a specified selection rule, such as the top 10% of the eligible content population.

### Why this is a useful baseline

Impressions are a simple and interpretable measure of search visibility. The baseline is easy to reproduce and gives a reference point for judging whether a more complex multi-feature approach provides additional prioritization value.

### Fair comparison

The baseline and the clustering-based approach must use:

* The same eligible content population.
* The same evaluation window.
* The same selection budget or number of items.
* The same outcome metric.
* The same evaluation split, if a future-period evaluation is used.

### Baseline result

The numerical baseline result has not yet been verified from a completed evaluation run.

**Required before submission:**

* Define the exact evaluation metric.
* Define the selection budget.
* Run the baseline.
* Run the clustering-based method under the same conditions.
* Record the resulting metrics in a committed results file.

No model improvement over the baseline is claimed in this report.

## 4. Model / analysis

### Method

The analysis uses K-Means clustering to group content items according to standardized performance and content-related features.

The workflow is:

1. Load the three Parquet tables with DuckDB.
2. Aggregate query-level and daily performance by `content_hash_id`.
3. Join the aggregates with content attributes.
4. Filter to published, non-deleted content.
5. Fill missing numeric performance fields with zero.
6. Engineer ratios, momentum, and log-transformed features.
7. Standardize the feature matrix with `StandardScaler`.
8. Evaluate candidate cluster counts from 4 through 7 using silhouette score.
9. Fit a six-cluster K-Means model.
10. Inspect cluster profiles and assign descriptive archetype names.
11. Map each archetype to a proposed action.

### Feature list

The current feature matrix contains:

* `log_impressions`
* `log_clicks`
* `ctr_90d`
* `avg_position_90d_w`
* `momentum`
* `query_count`
* `rare_impressions_share_avg`
* `engagement_rate`
* `avg_engagement_sec`
* `log_scroll_events`
* `log_backlinks`
* `log_search_volume`
* `word_count`

### Assumptions

* The content-level joins correctly represent the same content items across tables.
* Missing performance values after the join are treated as zero for the selected fields.
* Log transformations help reduce the influence of very large count values.
* Standardization is appropriate for comparing features with different scales.
* Six clusters provide a useful starting point for interpreting the content population.
* Cluster names are interpretations of observed profiles rather than ground-truth labels.

### Cluster count

The notebook evaluates K-Means solutions with cluster counts from 4 to 7 using silhouette scores.

The selected solution uses six clusters. The actual silhouette scores and the justification for selecting six clusters must be copied from a fresh notebook run.

### Archetype labels

| Cluster | Archetype              | Proposed action |
| ------- | ---------------------- | --------------- |
| 3       | Star Performer         | Protect         |
| 2       | Authority Pillar       | Improve         |
| 0       | Visible Underconverter | Rewrite         |
| 4       | Buried Thin Page       | Merge           |
| 5       | Hidden Engagement Gem  | Monitor         |
| 1       | Dormant                | Prune           |

These labels are analyst-defined interpretations based on cluster profiles.

### Target or proxy definition

There is no supervised target label in the current analysis. The cluster assignment is an unsupervised descriptive output, and the proposed action is a human-defined interpretation of the cluster profile.

## 5. Evaluation

### Evaluation design

The current notebook evaluates internal cluster structure using silhouette score and examines cluster profiles.

This is not a completed future-performance validation. The current feature construction uses observed performance fields from the available windows, so a random split of the same records would not establish that the archetypes predict future outcomes.

### Required comparison

A stronger evaluation should use a time-aware design:

1. Build features from an earlier performance window.
2. Fit the scaler and clustering model using the earlier window.
3. Assign content to archetypes.
4. Measure later clicks, impressions, or engagement in a separate future window.
5. Compare the clustering-based priority rule with the impressions baseline.
6. Use the same number of selected items and the same future outcome metric.

If a time-aware split is not possible with the available release, the report should clearly describe the evaluation as descriptive rather than predictive.

### Metrics

The following values must be produced by a fresh run:

| Metric                  |    Baseline | Clustering-based method |
| ----------------------- | ----------: | ----------------------: |
| Selection budget        | [TO VERIFY] |             [TO VERIFY] |
| Evaluation population   | [TO VERIFY] |             [TO VERIFY] |
| Mean future impressions | [TO VERIFY] |             [TO VERIFY] |
| Mean future clicks      | [TO VERIFY] |             [TO VERIFY] |
| Mean future engagement  | [TO VERIFY] |             [TO VERIFY] |
| Lift over baseline      | [TO VERIFY] |             [TO VERIFY] |

These results must not be filled with estimates.

### Internal clustering evaluation

The notebook reports silhouette scores for candidate values of K. The exact values from the fresh run should be recorded here.

* K = 4: [TO VERIFY]
* K = 5: [TO VERIFY]
* K = 6: [TO VERIFY]
* K = 7: [TO VERIFY]

### Error analysis

Because the current approach is unsupervised, it does not produce classification errors against a ground-truth action label.

A practical review should inspect:

* Pages assigned to unexpected archetypes.
* Pages with high impressions but low clicks.
* Pages with strong engagement but low search visibility.
* Pages with missing or zero performance after joining.
* Pages whose proposed action conflicts with business importance or content intent.

The final notebook should include a small manually inspected sample and document any surprising assignments.

## 6. Interpretation

### What the analysis found

The six-cluster solution separates content into groups with different observed combinations of visibility, ranking, engagement, authority, and content-related signals.

The current archetype interpretations are:

#### Star Performer — Protect

A group interpreted as having strong observed performance. These pages may deserve monitoring and protection from unnecessary changes.

#### Authority Pillar — Improve

A group interpreted as having strong authority or visibility signals with possible opportunities for further improvement.

#### Visible Underconverter — Rewrite

A group interpreted as having search visibility but weaker click performance relative to its exposure. These pages may deserve title, description, intent, or content-presentation review.

#### Buried Thin Page — Merge

A group interpreted as having limited visibility and relatively weak content-depth signals. These pages may be candidates for consolidation after manual review.

#### Hidden Engagement Gem — Monitor

A group interpreted as having useful engagement signals despite limited search visibility. These pages should not be removed based on search visibility alone.

#### Dormant — Prune

A group interpreted as having limited observed activity. These pages require additional checks before any decision to remove, redirect, or consolidate them.

### Visual evidence

The notebook produces two main figures:

1. A bar chart showing the number of content items in each archetype.
2. A scatter plot showing log-transformed 90-day impressions against weighted average search position, colored by cluster.

The figures show observed distributions and relationships. They do not establish causation.

### Negative or uncertain findings

The current work does not demonstrate that:

* The six clusters are the only useful grouping.
* The proposed actions improve future traffic.
* K-Means outperforms a simple impressions-based ranking.
* Any archetype predicts Google's ranking algorithm.
* A page should be deleted solely because it belongs to a particular cluster.

The absence of a completed future-period comparison is an important limitation, not evidence that the clustering method has no value.

## 7. Recommendation

### Ranked action playbook

The following order is a proposed operational review order. It is not a measured ranking of business impact.

### 1. Review Visible Underconverters — Rewrite

Check search intent, titles, descriptions, page openings, and whether the content matches the user's likely need.

**Confidence:** Directional.

### 2. Protect Star Performers

Monitor changes in clicks, impressions, ranking, and engagement. Avoid unnecessary changes to pages with strong observed performance.

**Confidence:** Directional.

### 3. Improve Authority Pillars

Review content completeness, internal linking, related-topic coverage, and intent alignment.

**Confidence:** Directional.

### 4. Review Buried Thin Pages — Merge candidates

Check whether the page has a unique purpose, valuable links, or content that should be retained. Consolidation should follow manual review.

**Confidence:** Directional.

### 5. Monitor Hidden Engagement Gems

Investigate other traffic sources, internal navigation, and the possibility that the page serves a narrow but valuable audience.

**Confidence:** Directional.

### 6. Review Dormant Pages — Prune candidates

Check business importance, backlinks, historical value, and whether refreshing or redirecting is more appropriate than removal.

**Confidence:** Directional.

### How an editor could use the output

An editor could:

1. Filter the action playbook by proposed action.
2. Review a small sample of pages from each archetype.
3. Check search intent, content quality, business importance, and technical context.
4. Record the final editorial decision.
5. Measure the result over a future period.
6. Update the prioritization approach using the observed outcomes.

### Honest framing

The recommendations are observed-pattern-based and directional. They are intended for decision-support and human review. They are not automatic decisions and have not been shown to cause improvements in rankings, traffic, or revenue.

## 8. Reproducibility

### Repository

Repository:

https://github.com/Priyansh-rath18/flyrank-internship-

The capstone notebook should be committed under:

`work/notebooks/`

The completed report should be saved as:

`work/capstone_report.md`

### Suggested repository structure

```text
flyrank-internship-/
├── work/
│   ├── notebooks/
│   │   └── content_archetypes_capstone.ipynb
│   └── capstone_report.md
├── submission/
│   └── paper_url.txt
└── README.md
```

### Environment

The analysis uses:

* Python
* DuckDB
* Pandas
* NumPy
* scikit-learn
* Matplotlib

Example installation command:

```bash
pip install duckdb pandas numpy scikit-learn matplotlib
```

### Reproduction steps

From a fresh clone:

```bash
git clone https://github.com/Priyansh-rath18/flyrank-internship-.git
cd flyrank-internship-
pip install duckdb pandas numpy scikit-learn matplotlib
```

Then open and run the capstone notebook from top to bottom.

The notebook should:

1. Load the dataset.
2. Build the analysis frame.
3. Engineer features.
4. Evaluate candidate cluster counts.
5. Fit K-Means.
6. Create cluster profiles.
7. Generate the charts.
8. Generate the action playbook.
9. Run the baseline and evaluation cells.
10. Export the results required by the report.

### Random seeds

The current clustering workflow uses:

* K-Means `random_state=42`.
* Sampling `random_state=42`.
* Silhouette evaluation `random_state=42`.

The exact random seed and parameters should remain consistent between the notebook and the reported results.

### Reproducibility status

The following items must be verified before submission:

* The notebook runs top to bottom without errors.
* The exact dataset release and date coverage are recorded.
* The baseline comparison is executed.
* The evaluation metrics are exported.
* The charts and cluster profiles are regenerated.
* The final results match a fresh notebook run.
* No credentials or private information are committed.

If a sealed or holdout evaluation is claimed, the code that creates the sealed frame and the resulting metrics file must also be committed.

## 9. Acknowledgments & data credit

Built on the **FlyRank ML Internship dataset**.

Data source: [FlyRank](https://flyrank.ai)

The analysis uses the public FlyRank internship warehouse and presents the resulting archetypes as descriptive, directional decision-support outputs.
