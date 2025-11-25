# Knight Hacks Dues Payment Prediction Dataset

This document describes the dataset, features, and preprocessing pipeline for the **Knight Hacks Dues Payment Prediction** project.

## Objective

Predict whether a Knight Hacks member will pay dues in a given year (`y_paid_dues`) using demographic, engagement, event, and Discord-based behavioral data.

Each observation represents a **(member, year)** pair.
The goal is to understand what patterns of engagement and involvement lead to paid dues and to identify at-risk members.

---

## Target Variable

| Feature         | Type         | Description                                                                  |
| --------------- | ------------ | ---------------------------------------------------------------------------- |
| **y_paid_dues** | Binary (0/1) | Whether the member has a dues payment record for that year in `DuesPayment`. |

---

## Raw Data Features

### 1. Tenure

| Feature                    | Type    | Description                                                                                                                   |
| -------------------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------- |
| **member_days_since_join** | Integer | Days between today and the member’s `dateCreated` in the database. Represents how long they’ve been part of the organization. |

---

### 2. Event Attendance (per year)

| Feature                      | Type    | Description                                                                                       |
| ---------------------------- | ------- | ------------------------------------------------------------------------------------------------- |
| **events_attended_year**     | Integer | Total number of events attended (via `EventAttendee`).                                            |
| **distinct_event_days_year** | Integer | Number of unique calendar days with attendance.                                                   |
| **attendance_streak_weeks**  | Integer | Longest run of consecutive weeks where the member attended ≥1 event.                              |
| **n_gbm_year**               | Integer | Number of General Body Meeting (GBM) events attended.                                             |
| **gbm_ratio**                | Float   | `n_gbm_year / events_attended_year` (fraction of events that were GBMs).                          |
| **n_social_year**            | Integer | Number of social-tagged events attended.                                                          |
| **social_ratio**             | Float   | `n_social_year / events_attended_year` (fraction of events that were socials).                    |
| **n_hello_world_year**       | Integer | Number of Hello World events attended.                                                            |
| **hello_world_ratio**        | Float   | `n_hello_world_year / events_attended_year` (fraction of events that were Hello World).           |
| **n_sponsorship_year**       | Integer | Number of sponsorship-related events attended.                                                    |
| **sponsorship_ratio**        | Float   | `n_sponsorship_year / events_attended_year` (fraction of events that were sponsorship).           |
| **n_tech_exploration_year**  | Integer | Number of tech exploration events attended.                                                       |
| **tech_exploration_ratio**   | Float   | `n_tech_exploration_year / events_attended_year` (fraction of events that were tech exploration). |
| **n_class_support_year**     | Integer | Number of class support events attended.                                                          |
| **class_support_ratio**      | Float   | `n_class_support_year / events_attended_year` (fraction of events that were class support).       |
| **n_workshop_year**          | Integer | Number of workshop-tagged events attended.                                                        |
| **workshop_ratio**           | Float   | `n_workshop_year / events_attended_year` (fraction of events that were workshops).                |
| **n_ops_year**               | Integer | Number of Ops (operations) events attended.                                                       |
| **ops_ratio**                | Float   | `n_ops_year / events_attended_year` (fraction of events that were Ops).                           |
| **n_collabs_year**           | Integer | Number of collaboration events attended.                                                          |
| **collabs_ratio**            | Float   | `n_collabs_year / events_attended_year` (fraction of events that were collaborations).            |

---

### 3. Feedback and Involvement (per year)

| Feature                    | Type         | Description                                                        |
| -------------------------- | ------------ | ------------------------------------------------------------------ |
| **feedback_count_year**    | Integer      | Number of feedback forms submitted (from `EventFeedback`).         |
| **avg_event_rating_given** | Float        | Mean of `overallEventRating` for feedback given by the member.     |
| **left_any_feedback**      | Binary (0/1) | Whether the member submitted at least one feedback form that year. |

---

### 4. Hackathon Involvement (all-time)

| Feature                   | Type         | Description                                                                                           |
| ------------------------- | ------------ | ----------------------------------------------------------------------------------------------------- |
| **has_gone_to_hackathon** | Binary (0/1) | Whether the linked hacker account has **ever** had a `status='checkedin'` record in `HackerAttendee`. |

---

### 5. Discord Engagement

| Feature                          | Type         | Description                                                          |
| -------------------------------- | ------------ | -------------------------------------------------------------------- |
| **discord_member**               | Binary (0/1) | Whether the user currently exists in the Knight Hacks Discord guild. |
| **discord_days_since_join**      | Integer      | Days between today and the Discord `joined_at` timestamp.            |
| **has_role_ops**                 | Binary (0/1) | Whether the member has the Ops role in the Discord guild.            |
| **discord_msgs_year**            | Integer      | Total messages authored by the user in the guild during that year.   |
| **discord_active_days_year**     | Integer      | Number of unique days the user sent at least one message that year.  |
| **discord_channels_posted_year** | Integer      | Number of distinct channels the user posted in that year.            |
| **discord_num_roles**            | Integer      | Total number of roles assigned to the member in the Discord guild.   |

---

### 6. Demographics

| Feature               | Type        | Description                                                 |
| --------------------- | ----------- | ----------------------------------------------------------- |
| **class**             | Categorical | Member's class standing (e.g., Freshman, Senior, Graduate). |
| **major**             | Categorical | Member’s declared major.                                    |
| **school**            | Categorical | University or college the member is associated with.        |
| **gender**            | Categorical | Member’s gender (self-reported).                            |
| **race_or_ethnicity** | Categorical | Member’s race or ethnicity (self-reported).                 |

---

## Engineered Features

These features are derived from the raw data during the `feature-engineering` pipeline to capture non-linear signals and simplify noisy categories.

### 1. Event Refinement / Interaction Terms

| Feature                | Type         | Description                                                                                                          |
| ---------------------- | ------------ | -------------------------------------------------------------------------------------------------------------------- |
| **engagement_breadth** | Integer      | Count of distinct event _types_ (GBM, Workshop, etc.) attended. Measures if a member is a "generalist".              |
| **is_discord_active**  | Binary (0/1) | 1 if `discord_msgs_year > 0`, else 0. Captures whether the user is active on Discord regardless of volume.           |
| **tech_score**         | Integer      | Sum of `n_workshop_year`, `n_collabs_year`, `n_hello_world_year`, `n_tech_exploration_year`, `n_class_support_year`. |
| **social_score**       | Integer      | Sum of `n_social_year`, `n_gbm_year`.                                                                                |
| **tech_social_ratio**  | Float        | `tech_score / (social_score + 1)`.                                                                                   |
| **is_sponsor_hunter**  | Binary (0/1) | 1 if `n_sponsorship_year / events_attended_year >= 0.8`, else 0.                                                     |

### 2. Demographic Grouping

| Feature            | Type        | Description                                                                           |
| ------------------ | ----------- | ------------------------------------------------------------------------------------- |
| **school_grouped** | Categorical | **UCF**: Includes 'University of Central Florida'. **Other**: All other institutions. |

---

**Missing data handling**

-   Null counts → 0.
-   Ratios computed only if denominator > 0, else `NULL`.
-   Missing ratings → average or left blank.

**Imputation strategy for training:**

-   All event ratio fields (`gbm_ratio`, `social_ratio`, `hello_world_ratio`, `sponsorship_ratio`, `tech_exploration_ratio`, `class_support_ratio`, `workshop_ratio`, `ops_ratio`, `collabs_ratio`): Impute with `0` when `events_attended_year = 0` (semantically correct: no events = no specific types).
-   `avg_event_rating_given`: Impute with -1 for all non-null ratings (preserves distribution).
-   `discord_days_since_join`: Impute with `-1` when `discord_member = 0` (clear sentinel for "not a Discord member").
