# Team Fortress 2 - Who can use chat?
### This is the final project for eecs 398 - practical data science

## Introduction
The game Team Fortress 2 (TF2), developed by Valve is an old and loved game that has been plagued by a botting problem. Some people might be familiar
with a viral online campaign to get Valve's attention in addressing the problem in recent years, but the plague has been around for at least a
decade. Some years ago, Valve disabled all chat communication for Free-To-Play (F2P) players in an effort to combat the abuse of the chat by botters.
This is a controversial feature that does clamp down on the chat abuse somewhat, but limits the game significantly for f2p players, like preventing
them from calling for `MEDIC!` when they are low on health. In a recent update, Valve began granting chat permissions to *some* f2p players, but
did not disclose the requirements, probably for good reason to discourage botters from finding a loophole. The youtuber Shounic put out a survey
to collect data on communications access and see if there were any factors that may point towards chat privileges. The [dataset](https://www.youtube.com/watch?v=ATzcWmuPfsA&t=1s) used in this project is the survey results published by Shounic.

<div style="position: relative; width: 100%; height: 0; padding-bottom: 56.25%;">
    <iframe src="https://www.youtube.com/embed/hYZHd7h1cLI" 
            style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;" 
            frameborder="0" allowfullscreen>
    </iframe>
</div>

### The million dollar question: What account characteristics determine whether or not a TF2 player can use chat after the update?

This question is important because examining Valve's metrics will give insight into what they believe distinguishes a bot from a real player. Botting isn't a problem unique to TF2, and verifying that somebody is a real person is becoming a more and more relevant problem in today's world, with the prevalence of AI.
Instead of using a CAPTCHA or any singular metric, modern tools will have to look at a variety of metrics and signals to determine if somebody is "real".


### The dataset has **12** columns and **3587** rows, corresponding to 3587 survey responses. Some important columns are:

`what do you have permission to do after the update?`
- Contains 4 unique strings, corresponding to levels of chat permissions: *text chat, no voice commands*, *no text chat, voice commands*, *text chat, voice commands*, *no text chat, no voice commands*
- This column will be the dependent variable that I will attempt to predict in this project

**Promising Features**

`how old is your steam account?`
- Contains a string in the format of *number* + *years old*, representing an integer number of years old a respondee's steam account is. (Steam is the platform that tf2 is on)

`how many hours do you have in tf2`
- Integer number of hours respondee has on tf2

`have you spent money on this steam account?`
- String that is "Yes" or "No" corresponding to if respondee has spent money on their steam account

`do you have a verified email on the account?`
- String that is "Yes" or "No" corresponding to respondee's verified email status

| what do you have permission to do after the update?   | how old is your steam account?   |   how many hours do you have in tf2? | have you spent money on this steam account?   | do you have a verified email on the account?   |
|:------------------------------------------------------|:---------------------------------|-------------------------------------:|:----------------------------------------------|:-----------------------------------------------|
| i can text chat, and i can use voice commands         | 6 years old                      |                                 3300 | Yes                                           | Yes                                            |
| i can text chat, and i can use voice commands         | 9 years old                      |                                 4025 | Yes                                           | Yes                                            |
| i can text chat, and i can use voice commands         | 1 year old                       |                                  330 | Yes                                           | Yes                                            |
| i can text chat, and i can use voice commands         | 3 years old                      |                                 1000 | Yes                                           | Yes                                            |
| i can text chat, and i can use voice commands         | 4 years old                      |                                  351 | Yes                                           | Yes                                            |



## Data Cleaning and Exploratory Data Analysis

### Cleaning Steps
1. **Hours played**: Removed extreme outliers (>20,000 hours) and the entire row for those entries (since those are just people messing with the survey)
2. **Account age**: Converted text values to numeric years (e.g., "6 years old" → 6.0)
3. **Target variable**: Created binary classification

### Univariate Analysis

<iframe
    src="assets/univariate.html"
    width="800"
    height="600"
    frameborder="0">
</iframe>
A majority of the people who took this survey have some communication access. This doesn't necessarily mean that a majority of TF2 players overall have comm access however. This bias will need to be taken into account when doing the model evaluation later since it isn't close to a 50-50 split.

### Bivariate Analysis

<iframe
    src="assets/bivariate.html"
    width="800"
    height="600"
    frameborder="0">
</iframe>
The responses were separated into different buckets from their experience level, and classified based on some metrics such as F2P status, steam spending, etc. I looked at the percentage of these players that had full comm access. From these plots there are some obvious trends. Paying players have basically full communication access regardless of experience level. For F2P players, there is a trend in greater access as they play for more hours. Similar trends are seen for players that have spent money elsewhere in steam, with a positive correlation between hours played and comm access rate across the board for every graph. Account security features such as 2FA and security level (calculated by summing up incidence of 2FA, email verified, phone verified) have a positive correlation with higher security → higher rate of chat access.

It should be noted that there is a noticeable drop in comm access for the expert group of respondees. This can possibly be attributed to the small sample size (110) and misidentification from the respondees.


### Interesting Aggregates

| F2P?, Paid Steam User? |   Access_Rate |   Avg_Hours |   Avg_Security_Score |   Sample_Size |
|:-----------------------|--------------:|------------:|---------------------:|--------------:|
| ('No', 'No')           |         0.658 |    1729.37  |                1.711 |            38 |
| ('No', 'Yes')          |         0.99  |    1581.73  |                2.812 |          1939 |
| ('Yes', 'No')          |         0.453 |     537.589 |                1.844 |           919 |
| ('Yes', 'Yes')         |         0.777 |     514.349 |                2.289 |           646 |

This table groups by `are you free to play?` and `have you spent money on this steam account?`. It aggregates the groupby object by the taking the mean of the `target` column, a new binary classification column which is a 1 for players that have comm access, 0 for no comm access.

- It then takes the mean of the cleaned `hours_numeric` column (which had converted strings to numbers and got rid of absurdly big numbers)
- the mean of `security_score` (calculated from summed incidence of 2FA, email verified, phone verified)
- the count of the columns in each group for a sample size metric

Some takeaways are that the majority of respondees are not F2P and have spent money elsewhere on steam. This corresponds to an almost 100% rate of comm access. The second highest comm access rate is found with players that haven't spent any money on TF2 in particular, but have elsewhere on steam, with 78% comm access rate. Complete freebies (F2P on TF2, and no money spent elsewhere), have the lowest rate of access at 0.453, but this is still pretty high and means that after the update, almost half of complete freebies now have some amount of comm access.

### Key Findings
- Most players surveyed have some form of communication access
- Free-to-play status appears is strongly correlated to communication restrictions
- Players with communication restrictions tend to have fewer hours played


## Framing a Prediction Problem

Before moving onto the prediction problem, there is something that must be taken into account.
The overall size of the dataset (about 3000 rows) is *relatively small*, and isn't large enough to make meaningful distinctions, picking out trends across specific types of access (i.e. full acccess, text chat only, no access, text and voice).
This is why it makes the most sense as a **binary classification problem** (the groundwork for which has already been laid in the data cleaning step), where a player can either have no access, or any amount of chat access

### **Dependent Variable**: Communication access permissions after TF2 updates
- `no_access`: Players with no communication abilities
- `some_access`: Players with any form of communication (text, voice, or both)

### Why This Response Variable?
Communication restrictions significantly impact the gaming experience. Being able to predict which players might face restrictions helps understand the factors that lead to these limitations.

### Evaluation Metric: F1-Score
Classes are imbalanced, precision and recall need to be evaluated and not just accuracy.

### Predictive Model:
The goal is to predict communication restrictions for new players based on account characteristics (account age, security features, spending history). These features would be available at prediction time since those are the factors that are taken into account when the game servers decide whether or not a player has chat access.


## Baseline Model

### **Dependent Variable**: Communication access permissions after TF2 updates
- `no_access`: Players with no communication abilities
- `some_access`: Players with any form of communication (text, voice, or both)

### Three independent variables (features):
- Free to play status
- If money has been spent elsewhere in steam account
- Hours played in TF2

### Model Description
**Algorithm**: K-Nearest Neighbors (k=1)  
**Features**: 3 total features
- **Categorical (2)**: Free-to-play status, Money spent on account
- **Quantitative (1)**: Hours played in TF2

### Features
- **Categorical encoding**: OneHotEncoder with `drop='first'` to avoid multicollinearity
- **Numerical scaling**: StandardScaler for hours played (important for KNN distance calculations)
- **Pipeline**: All preprocessing steps implemented in a single sklearn Pipeline

### Performance
- **F1-Score**: 0.866
- **Accuracy**: 0.784

**Confusion Matrix:**

|Actual             | predict no_access | predict some_access|
|------------------:|------------------:|-------------------:|
|actual no_access   |   61              |                 75 |
|actual some_access |   78              |                495 |

### Model Assessment
The baseline model achieves an F1-score of 0.866, which is **decent but has room for improvement**. The model shows alright performance with some challenges in correctly identifying comm restricted players (44% precision for no_access class).

It can generalize unseen data only to the extent to which it resembles the existing data frome existing survey responses (aka not very well).
I wouldn't consider it to be too reliable, considering only about 3k responses out of millions of tf2 players. There's probably also additional biases in that everyone who responded to this survey also was a watcher of this particular youtuber, and unseen data (as in random tf2 player) would not have this trait (and related tendencies such as english speaker, etc).

False positives are especially prevalent, exceeding the number of true negative predictions, which means this model is too quick to predict `some_access`, due to the bias of the dataset being comprised of mostly respondees that have `some_access`.

A numerical based model would probably extrapolate better as it would factor in numerical trends, and not just a datapoint's relation to the training set. But for this to work, I would need a lot of quantitative features, when there really only is two.
A lot of the data is boolean, so some sort of decision tree would probably be more effective.

### Model Characteristics
KNN (k=1) makes predictions based on the single most similar player in the training data. While this can capture complex patterns, it may be overly sensitive to individual data points. The confusion matrix shows the model has difficulty distinguishing between the classes, with 75 false positives and 78 false negatives, indicating there's significant room for improvement in the final model.


## Final Model

### **Dependent Variable**: Communication access permissions after TF2 updates
- `no_access`: Players with no communication abilities
- `some_access`: Players with any form of communication (text, voice, or both)

### Independent variables (features):
**Base Features**
  1. are you free to play?
  2. have you spent money on this steam account?
  3. have you used a VPN on this steam account?
  4. do you have steam guard/2FA enabled?
  5. do you have a phone number attached to the account?
  6. do you have a verified email on the account?
  7. hours_numeric
  8. account_age_years

**Engineered Features**
1. Polynomial features on account age
2. Security Score
3. Account Trust Score

**Total number of features**: 11

### Model Description
**Algorithm**: Random Forest Classifier  
**Features**: 8 base features + 3 engineered features

### Feature Engineering
**Engineered Features Added**:
1. **Polynomial Features on Account Age**: Used `PolynomialFeatures(degree=2)` to create age and age² features, capturing non-linear relationships where very old accounts might have different trust patterns
2. **Security Score**: Custom `FunctionTransformer` that combines Steam Guard/2FA, phone number, and verified email into a single security metric (0-3 scale), then standardized
3. **Account Trust Score**: Custom `FunctionTransformer` combining spending history, VPN usage, and account age into a trust metric, then standardized

**Engineered feature description**:
- **Polynomial age features**: Captures diminishing returns of account age - the difference between 1 and 5 years matters more than 10 vs 14 years
- **Security score**: Aggregates multiple security indicators that likely correlate with account trustworthiness
- **Trust score**: Combines behavioral indicators (spending, VPN usage) with account maturity

### Algorithm Selection
**Random Forest Classifier** was chosen over the baseline KNN because:
- Better handles mixed data types (categorical + numerical)
- More robust to outliers than KNN
- Better performance on tabular data

### Hyperparameter Tuning
**Method**: GridSearchCV with 3-fold cross-validation  
**Parameters Tuned**:
- `n_estimators`: [100, 200] - Number of trees in the forest
- `max_depth`: [10, 15, None] - Maximum depth to prevent overfitting
- `min_samples_split`: [2, 5] - Minimum samples required to split a node

**Best Parameters Found**:
- `n_estimators`: 100
- `max_depth`: 10  
- `min_samples_split`: 2

### Performance
- **F1-Score**: 0.885
- **Accuracy**: 0.814

**Confusion Matrix:**

|Actual             | predict no_access | predict some_access|
|------------------:|------------------:|-------------------:|
|actual no_access   |   70              |                 66 |
|actual some_access |   66              |                507 |

### Improvement Over Baseline
**Baseline (KNN k=1)**:
- F1-Score: 0.866
- Accuracy: 0.784

**Final Model (Tuned Random Forest)**:
- F1-Score: 0.885 (+0.019 improvement)
- Accuracy: 0.814 (+0.030 improvement)
- **Relative improvement**: 2.2% increase in F1-score

### Model Assessment
The final model achieves a marginal improvement over the baseline, with better balanced performance across both classes. The confusion matrix shows improved precision for the `no_access` class (from 44% to 51%) while maintaining strong performance on the majority class. However, the rate of false positives are still pretty high, so the model is still influenced by the bias in the dataset. The feature engineering successfully captured additional signals in the data, and the Random Forest algorithm proved more effective than KNN for this classification task.
