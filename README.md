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


