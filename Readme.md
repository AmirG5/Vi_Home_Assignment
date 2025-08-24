# Vi Home Assignment — Churn Uplift Solution

**Submitted by Amir Gabay, Phone number: 050-2440885**
---

This repository contains my solution to the VI Data Science home assignment.  
The goal of the task is to identify members at risk of churn and generate a ranked list of those who should be prioritized for outreach.

Link to presentation: [Click here](https://docs.google.com/presentation/d/1tvzlTxqdAsnBo4Q8cLCtwVKjJUPIoqvhQuccT6yGMQ4/edit?usp=sharing)

## 💡 General explanation

I chose uplift modeling to address this challenge. Uplift modeling is a causal inference method that helps predict a member’s outcome if they receive outreach (treatment) versus if they do not (control). To achieve this, we estimate the counterfactual — what would have happened had the member received or not received the treatment.

### Features used

In this challenge I created various different features:

1. **Claims features:** The number of claims a member had in each icd-code in the observation window (ten features).
2. **App usage features:** Number of times the member used the app in the observation window (one feature).
3. **Web visits features:** Number of times the member visited any web page in the observation window (one feature) as well as the number of times the member visited any of the 26 different page titles in the observation window (another 26 features, so total of 27).
4. **Temporal features:** The observation window ends at 2025-07-15 00:00:00 (and the churn label is a near-term churn after the observation window). We calculate the number of days from the member's last interaction in the app, website and from their last claim up to the end of the observation window. We also calculate the number of days from the member's signup date (total of four features).

So a total of 42 features for each member.

### Train/test split

The data represents 10k members that were observed in an observation window and we want to predict whether or not they churned right after that observation window. Thus we split the data into 75% train set (7500 members) and 25% test set (2500 members). In order to keep the distributions of the churn and treatment labels, we stratify the split on the churn and treatment labels. This way our train set can represent the test set's distribution.

We consider the train set as a set of members we could learn from in the past and that we can't outreach anymore. The test set is considered as a set of members that we can outreach and we want to find out who we should outreach most.

### Approach and models training

For this challenge I used the **T-learner** approach, which trains two separate models:

1. $P(\text{Churn} \mid X_i, T_i = 0)$ — The probability of churn if a member is in the **control group** (no outreach). This is done by training a model solely on non-outreached members.

2. $P(\text{Churn} \mid X_i, T_i = 1)$ — The probability of churn if a member is in the **treatment group** (receives outreach). This is done by training a model solely on outreached members.

Where $X_i$ is member $i$'s features and $T_i$ is whether member $i$ received a treatment or not.

By estimating both probabilities for each member, we can calculate the **Individual Treatment Effect (ITE)**, which represents the uplift:

$$
ITE = P(\text{Churn} \mid X_i, T_i = 0) - P(\text{Churn} \mid X_i, T_i = 1)
$$

Since churn is an undesirable outcome, we define uplift as the reduction in churn risk due to outreach.  
This is why the treatment probability is subtracted from the control probability: a positive uplift means that outreach decreases the chance of churn.

### Evaluation

In order to evaluate the models I used both ROC AUC and the Qini curve.

- **ROC AUC-** Measures how well a model distinguishes between two classes (e.g., churn vs. no churn). A score of 0.5 means random guessing, while 1.0 means perfect separation.
- **Qini curve-** Evaluates uplift models by showing the incremental benefit (e.g., churn reduction) achieved when targeting members ranked by predicted uplift.
It helps identify the optimal fraction of the population to target, compared to random outreach.

By the Qini curve we could see that the XGBoost model gave the best results and it showed that we should send an outreach to the first 50% members in the test set. When we filtered out already outreached members (because we assume that we don't want to outreach the same member twice), we received n=765 (out of ~1500 potential members).

---

## 📂 Project Structure


    deliverables/
    ├── Churn uplift solution.ipynb # Main notebook (end-to-end workflow)
    └── prioritized_outreach_members.csv # Ranked list of members recommended for outreach


    data/
    └── (all CSVs and schema files provided with the assignment)

    - Readme.md # This file

---

## 🚀 How to Run

1. Open the main notebook (`Churn uplift solution.ipynb`).
2. (Optional) Consider running it inside a **virtual environment**. If wanted you can create one by running in your cmd:
     ```bash
     # Create a new virtual environment
     python3 -m venv .venv
     ```
     Then simply choose it as your Python kernel.
3. All dependencies are installed automatically with `pip install` commands inside the notebook. After you run the pip install cells please restart your kernel (some packages require a restart).
4. Run all the rest of the notebook.


The notebook runs the full workflow:
 - Exploration of the provided datasets (web_visits, app_usage, claims, churn_labels)
 - Preprocessing and feature engineering
 - Model training and evaluation using uplift modeling with a T-learner approach
 - Evaluation with ROC AUC and Qini curves
 - Export of the ranked outreach list

 ---

## 📊 Deliverables

- **Notebook:** `Churn uplift solution.ipynb` — complete workflow, methodology, and evaluation.  
- **CSV:** `prioritized_outreach_members.csv` — ranked list of members **who did not receive outreach historically**, with:
- `member_id`
- `prioritization_score` (the uplift score)
- `rank`

## 📝 Notes

**Thank you for an interesting assignment and for considering me for this role!**