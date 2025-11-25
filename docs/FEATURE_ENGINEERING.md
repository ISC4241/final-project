# Feature Engineering Report

This document details the feature engineering decisions made for the Knight Hacks Dues Prediction project. The goal was to transform raw behavioral data into high-signal features for predicting `y_paid_dues`.

## 1. Categorical Simplification

The raw dataset contained high-cardinality categorical variables that would lead to sparse feature matrices if one-hot encoded directly.

### School Grouping (`school_grouped`)

-   **Problem**: The `school` column had many unique values, but the vast majority were "University of Central Florida".
-   **Solution**: Grouped all non-UCF entries into `Other`.
-   **Rationale**: The primary population is UCF students. Splitting the long tail of other universities adds noise without sufficient sample size for the model to learn patterns.

### Major Grouping (`major_grouped`)

-   **Problem**: `major` had dozens of variations.
-   **Solution**: Grouped into 4 key buckets:
    -   **Computer Science**
    -   **Information Technology**
    -   **Computer Engineering**
    -   **Other** (All other majors)
-   **Rationale**: Tech majors behave differently. Separating CS, IT, and CE allows the model to capture specific sub-culture differences (e.g., IT students might be more social, CE more workshop-focused) rather than lumping them all into one "Tech" bucket.

_(Note: We retain the raw `class` features as distinctions within class standing are valuable signals.)_

---

## 2. Event Granularity & Interaction Terms

Initial analysis showed that raw event counts were good predictors, but the _type_ of engagement mattered more than raw volume.

### Engagement Breadth (`engagement_breadth`)

-   **Definition**: The count of distinct _types_ of events attended (GBM, Workshop, Social, etc.).
-   **Rationale**: Differentiates "generalists" (who explore everything) from "specialists" (who only come for one thing).
-   **Hypothesis**: Higher breadth indicates deeper integration into the community $\rightarrow$ higher likelihood of paying dues.

### Tech vs. Social Score

-   **Tech Score**: Sum of `n_workshop`, `n_collabs`, `n_hello_world`, `n_tech_exploration`, `n_class_support`.
-   **Social Score**: Sum of `n_social`, `n_gbm`.
-   **Rationale**:
    -   **Tech events** signal a desire for skill acquisition.
    -   **Social events** signal a desire for community.
    -   We created `tech_social_ratio` to measure the _balance_ of these intents.

### Sponsor Hunter Flag (`is_sponsor_hunter`)

-   **Definition**: Binary flag. `1` if `n_sponsorship_year > 0` AND sponsorship events make up $\ge 80\%$ of total attendance.
-   **Rationale**: The "Grifter Hypothesis". Some members only attend events with free swag/food (often sponsorship events) and have no intention of supporting the club financially. This flag isolates that behavior.

---

## 3. Feature Selection: What We Kept vs. Dropped

### Kept: Strong Signals

-   **`n_ops_year` & `ops_ratio`**:
    -   Correlation: **+0.27** and **+0.22**.
    -   **Insight**: Members who help run the club (Ops) are highly likely to pay dues.
-   **`attendance_streak_weeks`**:
    -   Correlation: **+0.35** (Highest).
    -   **Insight**: Consistency is the strongest predictor of payment.
-   **`class_support_ratio`**:
    -   Correlation: **-0.16**.
    -   **Insight**: Members who _only_ come for class support (tutoring/help) are _less_ likely to pay. They are transactional users.

### Dropped: Weak/Noisy Signals

-   **Raw Ratios (`social_ratio`, `workshop_ratio`, etc.)**:
    -   Correlations were near zero (e.g., `social_ratio` $\approx 0.03$, `workshop_ratio` $\approx 0.005$).
    -   **Reason**: A high ratio often just meant "attended 1 event and it was a workshop". This didn't correlate with payment. The _counts_ (`n_workshops`) carried the actual signal.
-   **Sensitive Demographics**:
    -   Dropped `gender` and `race_or_ethnicity` to prevent algorithmic bias, as they showed low correlation/importance anyway.

---

## 4. Summary of Engineered Features

| Feature              | Type   | Source  | Impact Hypothesis                          |
| :------------------- | :----- | :------ | :----------------------------------------- |
| `engagement_breadth` | Int    | Derived | High breadth $\rightarrow$ High commitment |
| `tech_score`         | Int    | Derived | Skill-seeking behavior                     |
| `social_score`       | Int    | Derived | Community-seeking behavior                 |
| `is_sponsor_hunter`  | Binary | Derived | Negative signal (transactional behavior)   |
| `is_discord_active`  | Binary | Derived | Digital engagement proxy                   |
