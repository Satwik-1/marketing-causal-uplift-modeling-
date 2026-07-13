# marketing-causal-uplift-modeling-
Built a marketing causal uplift model using statistical analysis and machine learning to estimate individual treatment effects, optimize campaign targeting, and maximize marketing ROI.

# Marketing Campaign Effectiveness: Causal Inference and Uplift Modeling

Predicting who **will** convert is often less valuable than predicting **who will convert because of an intervention**.

In this project, I analyzed the effectiveness of a digital marketing campaign using randomized experimentation, causal inference, and uplift modeling. Beginning with an A/B test, I estimated the average treatment effect of the campaign before simulating a real-world observational setting and applying Propensity Score Matching (PSM) and Inverse Probability of Treatment Weighting (IPTW) to illustrate common adjustment methods for observational data. Finally, I built a T-Learner uplift model to estimate individualized treatment effects and identify which customers should actually receive advertising.

---

## Business Problem

A company wants to determine whether showing advertisements increases customer conversions.

There are two important business questions:

1. **Does advertising increase conversions overall?**
2. **Which customers should receive advertisements to maximize profit?**

Traditional predictive models estimate the probability that a customer converts. However, customers who are already likely to convert without intervention may not benefit from advertising. The objective of uplift modeling is to estimate the **incremental causal effect** of treatment for each individual, allowing marketing resources to be allocated where they are expected to generate the greatest additional value.

From a behavioral perspective, customers do not respond uniformly to marketing interventions. Differences in preferences, motivation, and decision-making lead to **treatment effect heterogeneity**, meaning the same advertisement may influence some customers while having little or no effect on others.

Sending advertisements to every customer can waste marketing budget, while failing to target customers who are likely to respond because of the advertisement leaves revenue unrealized.

This project demonstrates how causal inference and uplift modeling can support better marketing decisions.

---

## Dataset

The project uses the Criteo Uplift Modeling Dataset, which contains approximately 13 million customers. For computational efficiency while preserving a large-scale analysis, I randomly sampled 200,000 customers for this project.

Each observation contains:

- Customer features (`f0`–`f11`)
- Treatment indicator (advertisement shown)
- Conversion outcome
- Additional behavioral variables used for modeling

---

## Exploratory Data Analysis

The dataset was explored before modeling to understand treatment assignment, conversion behavior, and feature distributions.

Initial exploration focused on:

- Treatment vs. control sizes
- Conversion rates
- Feature distributions
- Missing values
- Class balance

---

## Randomized A/B Test

The dataset originates from a randomized experiment, allowing estimation of the **Average Treatment Effect (ATE)** without confounding bias.

The **Average Treatment Effect (ATE)** was estimated as the difference in conversion rates between the treatment and control groups. A confidence interval and hypothesis test were then used to determine whether the observed treatment effect was statistically distinguishable from zero rather than the result of random sampling variability.

<img width="395" height="200" alt="01_ab_test_effect" src="https://github.com/user-attachments/assets/6745b349-ad1e-4842-814a-dec89593c959" />

The experiment showed that the advertising campaign increased conversions on average.

---

## Business Impact

Beyond determining statistical significance, the treatment effect was translated into estimated business value.

Using assumed revenue per conversion and advertising cost, the overall financial impact of the campaign was estimated to determine whether the campaign was economically beneficial.

---

## Simulating an Observational Study

Randomized experiments are often unavailable in practice.

To demonstrate causal inference methods, I simulated an observational dataset by removing the randomized treatment assignment and introducing treatment-selection bias.

A naïve comparison of treatment and control groups would therefore produce biased estimates.

---

## Propensity Score Matching (PSM)

To reduce selection bias, I estimated each customer's probability of receiving treatment using logistic regression.

A logistic regression model estimated each customer's **propensity score**, or probability of receiving treatment conditional on the observed features. Treated customers were then matched to their nearest untreated neighbor in propensity-score space to create more comparable groups before estimating the treatment effect.

---

## Inverse Probability of Treatment Weighting (IPTW)

As an alternative adjustment method, I implemented IPTW.

Rather than matching individuals, IPTW assigns weights based on inverse treatment probabilities. Unlike matching, which discards some observations, IPTW retains the full dataset by weighting each customer according to the inverse of their probability of receiving the treatment they actually received. This creates a weighted pseudo-population in which observed covariates are more balanced between treatment groups.

### Common Support

Before interpreting causal estimates, overlap between treatment and control propensity scores was verified.

<img width="461" height="236" alt="02_propensity_overlap" src="https://github.com/user-attachments/assets/ce38b7f6-9cce-44c6-bbb4-b5e65b8df5f4" />

The observed overlap indicates that causal comparisons are reasonable for much of the population.

Although these methods improved comparability between groups, their estimates remained different from the randomized A/B result, illustrating the limitations of adjustment methods in observational settings.

---

## Uplift Modeling (T-Learner)

Average treatment effects do not answer which individual customers should receive advertisements.

To estimate individualized treatment effects, I implemented a **T-Learner**.

Two separate logistic regression models were trained:

- Treatment model
- Control model

For every customer:

- Treatment conversion probability was predicted.
- Control conversion probability was predicted.
- Predicted uplift was calculated as:

```
P(Convert | Treatment)
-
P(Convert | Control)
```

Customers with larger predicted uplift are expected to benefit more from receiving advertisements.

---

## Uplift Validation

Predicted uplift was validated by grouping customers into deciles according to predicted treatment effect.

Observed treatment effects were then calculated within each decile. To avoid optimistic evaluation, the T-Learner was trained on a training split and uplift validation was performed only on the held-out test set.

<img width="324" height="180" alt="Screenshot 2026-07-06 003043" src="https://github.com/user-attachments/assets/6b6b243a-8c68-4d39-8d6c-852c81cd4de8" />

Customers in the highest predicted uplift deciles experienced substantially larger observed treatment effects, suggesting that the model successfully ranked customers according to their responsiveness to advertising. Because individual treatment effects are not directly observable, uplift models cannot be evaluated using traditional supervised learning metrics alone. Instead, customers were ranked by their predicted uplift and grouped into deciles, allowing observed treatment effects to be compared across groups. A successful uplift model assigns larger observed treatment effects to higher predicted uplift deciles, demonstrating effective prioritization of customers rather than merely accurate conversion prediction.

---

## Business Strategy Evaluation

Three targeting strategies were compared:

- Advertise to everyone with positive predicted uplift
- Advertise only to the top 20% highest-uplift customers
- Advertise only to the top 10% highest-uplift customers

Expected revenue, advertising cost, and expected profit were estimated under simple business assumptions.

<img width="344" height="182" alt="Screenshot 2026-07-06 003052" src="https://github.com/user-attachments/assets/c6b357df-0501-463d-85c5-4d02e99d2e95" />

Targeting customers with the highest predicted uplift substantially improved expected profitability. Under the assumptions used in this analysis, targeting the top 20% of customers produced the highest estimated profit, illustrating how personalized treatment strategies can outperform uniform advertising.

---

## Key Results

- Estimated the Average Treatment Effect using a randomized A/B test.
- Demonstrated how observational bias can distort causal conclusions.
- Applied Propensity Score Matching (PSM) and Inverse Probability of Treatment Weighting (IPTW) to demonstrate adjustment methods for observational data.
- Built a T-Learner to estimate individualized treatment effects.
- Evaluated uplift predictions by comparing observed treatment effects across predicted uplift deciles.
- Compared alternative marketing strategies using expected business profit rather than predictive accuracy alone.

## Behavioral Interpretation & Future Work

This project demonstrates that customers do not respond uniformly to marketing interventions. Rather than asking whether a campaign works on average, uplift modeling identifies which customers are most likely to change their behavior because of treatment. Future work could investigate why these customers are more persuadable by incorporating additional behavioral features or experimentally testing alternative messaging strategies.

---

## Assumptions

### A/B Test

- Random treatment assignment
- Independent observations
- Stable Unit Treatment Value Assumption (SUTVA)

### Propensity Score Methods

- No important unmeasured confounders
- Positivity (overlap)
- Correct specification of the propensity score model

### T-Learner

- Representative training data
- Stable relationships between features and conversion probability
- Reasonably calibrated probability estimates

---

## Limitations

- The observational dataset was simulated for demonstration purposes.
- Unmeasured confounding cannot be completely eliminated in real observational studies.
- Business assumptions (revenue per conversion and advertising cost) were simplified.
- The T-Learner used logistic regression; more flexible uplift models may capture more complex treatment heterogeneity.

---

## Future Improvements

Possible extensions include:

- X-Learners and DR-Learners
- Causal Forests
- Budget-constrained targeting optimization
- Contextual bandits for adaptive marketing
- Probability calibration
- Production deployment and monitoring
- Evaluate uplift models using specialized metrics such as Qini curves and AUUC (Area Under the Uplift Curve).
  
---

## Technologies

- Python
- pandas
- NumPy
- matplotlib
- scikit-learn
- SciPy
- statsmodels

---

## How to Run

Install dependencies:

```bash
pip install -r requirements.txt
```

Open the notebook:

```text
marketing_uplift_modeling.ipynb
```

---

## Repository Structure

```
marketing-uplift-modeling/

├── marketing_uplift_modeling.ipynb
├── README.md
├── requirements.txt
└── images/
    ├── 01_ab_test_effect.png
    ├── 02_propensity_overlap.png
    ├── 03_uplift_validation.png
    └── 04_profit_comparison.png
```
