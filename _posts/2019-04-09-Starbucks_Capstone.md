---
layout: post
title: Starbucks Capstone Challenge
---

![_config.yml]({{ site.baseurl }}/images/SBUX_placeholder.png)

## Project Overview

With mobile applications well into maturity as a platform, by 2019, businesses with large installed user bases can consider historic transaction records as a free source of very in-depth market research.  This allows the business to precisely identify customer segments, and analyze each demographic segment's preferred interactions with the business.  This will allow the business to offer the user an improved and custom-tailored purchasing experience, increasing customer satisfaction, and will also allow the business to sell more product as a result.

The project represents a simulation of the behavior users of Starbucks' mobile app over a period of time in which users are sent promotional offers.  The promotional offers fall into one of three types: buy-one-get-one-free ('BOGO'), informational, and discount.  The parameters of each offer vary (some are valid for longer than others, some provide a greater reward), and the offers are made by way of different marketing channels – some combination of web, mobile, social, and email.  However, there are only ten variations of these offers in total.

The possible objectives of the exercise are quite open-ended.  My implementation is based on a high-level observation of the data and business domain (marketing):

- Although two events occur in sequence, it is not possible to claim that they are dependent on each other (no causality).  A customer might buy the same latte every morning, regardless of whether he or she was given a BOGO offer by email.
- The outcome of an offer is not repeatable (no determinism).  A customer might get the same discount on two separate days, but might just not feel like having a coffee on one of those days.
- For this reason, there is no cause-and-effect relationship (causal determinism).  This point might be lost on someone approaching the data science domain from a software background, for example, where the same SELECT clause had better give the same result each time.

#### Problem Statement

My implementation of this project is oriented towards describing and predicting human behavior on an aggregate level.  The project can be summarized by one key question:

*Given a user with some demographics, who is provided with an offer with some parameters, what is the user's expected change in behavior, expressed as the product of transaction rate times average transaction value?*

I will be approaching this exercise as though I were consulting for a marketing team, and providing quantifiable information which can be used by those with financial information (product pricing, for example) to render a business decision.

#### Metrics

I have found that changes in both transaction volume and value can be described fairly well, and predicted to some degree.  My proposed strategy is to anaylze the data two ways – descriptive statistics, to give the imaginary marketing team a set of heuristic rules for targeting customers, and a regression learner, which can predict a user's response to the proposed campaign.  

The performance metrics – MSE and R-squared are used to find the best learner.  Heuristics are tricky to quantify, but they will be based on values found in the statistical analysis that are intuitively noteworthy.

## Data Exploration and Visualization

The data set is deceptively simple.  It contains three JSON files with a minimal number of fields:

**portfolio.json**
* id (string) - offer id
* offer_type (string) - type of offer ie BOGO, discount, informational
* difficulty (int) - minimum required spend to complete an offer
* reward (int) - reward given for completing an offer
* duration (int) - time for offer to be open, in days
* channels (list of strings)

**profile.json**
* age (int) - age of the customer 
* became_member_on (int) - date when customer created an app account
* gender (str) - gender of the customer (note some entries contain 'O' for other rather than M or F)
* id (str) - customer id
* income (float) - customer's income

**transcript.json**
* event (str) - record description (ie transaction, offer received, offer viewed, etc.)
* person (str) - customer id
* time (int) - time in hours since start of test. The data begins at time t=0
* value - (dict of strings) - either an offer id or transaction amount depending on the record

#### Profile.json
"Profile" contains the information on the simulated users.  There are 17000 users in total.  A user is defined by four demographics: age, gender, income, and membership date.  Data is not necessarily complete...

**Age**

![placeholder_1](https://ismith1024.github.io/images/age_raw.png)

*Fig 1: Raw user age data*

... Figure 1 shows that unknown age is encoded as 118 years old, and there are many of these users!

**Gender**

 Male  | Female  | Other  | No Response
---|---|---|---
 8484  | 6129  | 212  | 2175

*Fig 2: Raw gender data*

... Figure 2 shows how gender contains null values.  Note also that any user who does not identify as male or female is tagged as "other".

**Income**

![placeholder_1](https://ismith1024.github.io/images/income_raw.png)

`Percent of users non-responsive for income: 12.794117647058824%`

*Fig 3: Raw user income data*

... Figure 3 shows how income contains null values.

I evaluated these missing data points both by way of imputing missing values, and by removing missing data from the experiment.  I found the machine learning performance was better with the imputed data.  As for the purposes of statistical significance, with a sample size of 17000, removing even thousands of records did not show any detrimental effects.

#### Portfolio.json

Portfolio.json is a straightforward record of all ten offers, contianing a hex id string, list of channels, difficulty, duration, reward, and offer type (one of 'bogo', 'informational', or 'discount')

#### Transcript.json

Transcript.json contains the time series data for all events - offer received, offer viewed, transaction, and offer completed.  The JSON objects did not lend themselves to creating a dataframe directly as not every event had the same characteristics. I built a series of functions to inspect a JSON object and place the relevant characteristics in a suitable column (for example, 'transaction amount').  Columns with no applicable value were encoded as -1.

```
def get_offer_id(entry):
    '''
    in: an event JSON object from the transcript
    out: the id of the offer, or -1 if not there
    '''
    if 'offer_id' in entry:
        return entry['offer_id']
    elif 'offer id' in entry:
        return entry['offer id']
    else:
        return -1
    
def get_amount(entry):
    '''
    in: an event JSON object from the transcript
    out: the amount of the transaction, or -1 if not therele
    '''
    if 'amount' in entry:
        return entry['amount']
    else:
        return -1
    
def get_reward(entry):
    '''
    in: an event JSON object from the transcript
    out: the reward of the transaction, or -1 if not therele
    '''
    if 'reward' in entry:
        return entry['reward']
    else:
        return -1
```

#### Relationship between demographics

It is worthwhile to ask whether any demographics are strongly correlated.  This is important; for example if it were found that older users bought more BOGOs, but age were correlated with income and in fact it is higher earners who spent more money, then the finding regarding age would be spurious.

**Age vs Membership Period**

![placeholder_1](https://ismith1024.github.io/images/myear_age.png)

*Fig 4: Membership_year-age, gradient scatter plot with histograms*

Figure 4 shows that a majority of users are recently signed up to the app, and the mean value of around fifty-three holds in general for all lengths of membership.

**Age vs Income**

![placeholder_1](https://ismith1024.github.io/images/inc_age.png)

*Fig 5: Income-age, gradient scatter plot with histograms*

Figure 5 is interesting as it illustrates an artifact from the synthetic data.  It appears that three distributions, one with income up to 75k and all ages, one with income 50K to 105k and ages 37+, and one with income 70k to 120k and ages 50+ were generated and then merged.  This would explain the square features in the lower left corners of the highest density color gradient.

Ignoring the artifacts from the synthetic data, it appears that the intention was to have a somewhat linear dependency between age and income.  This can be seen from the color gradient.


**Gender vs Income**

![placeholder_1](https://ismith1024.github.io/images/inc_gend.png)

*Fig 6: Income-age, violin plot*

Males have a clearly lower income distribution than Females and Others.  There are many more Females represented in the top 25% of incomes.  The next 25% is disproportionatley Female and Other, the next 25% is where most of the Others and Males are.  The lowest quartile of income contains a Male majority.

Lower income Males, higher income Females, and mid income Others gravitate to the Starbucks app (in this simulation anyway).

In summary, there does not appear to be a need at this time to throw out any data series, but the dependency on gender and age with respect to income should be considered when making a final interpretation of any findings.


## Methodology

The overall objective of the project is to simulate a data science exercise in which the customer (Starbucks Marketing Team) is provided with a model to predict changes in customer behavior when provided with a propmotional offer. This will be measured by:

 - Given a user, with specific demographics,
 - And given an offer, with specific parameters,
 - What is the predicted change in the user's transaction rate before and after receiving the offer?
 - And what is the predicted change in the user's average transaction value over the duration of the offer period?

I considered the following technical means to create this predictive model:
 - Traditional Statistics
 - Regression Learning

The desired outcome is a descriptive set of heuristics on which types of offers are effective when presented to various user demograpics, and to provide a predictive model which will quantify the expected effectiveness of an offer given its type and audience.

#### Preprocessing

A significant amount of data-wrangling work was necessary to transform the transcript time-series data into a table of the form 

| User Demographics | Offer Parameters | Change in Transaction Rate | Change in Average Transaction Value|

This required me to determine what offers each user had received, what is their duration, and what intervals of the experiment did not fall within any offer period. Once the offer periods and no-offer periods were determined, transactions and their values were used to evaluate trnasaction rates and values by offer or no-offer interval.

The profile data is missing significant sections of demographic data, and I have evaluated the impact of imputing missing values vs. omitting incomplete records.  The regression learner performed better when missing values were imputed.

I parsed the date string into a datetime object to enable processing.

#### Transaction Record Preprocessing

A central part of the analysis is the concept of an offer interval.  For each user, I want to determine: 
 - When an offer is made, how many transactions take place over the duration of the offer? 
 - What is the value of these offers?
 - How often does the user make transactions when no offer is in efffect?
 - What is the value of these offers?

By segmenting the offers, I was able to establish an A and B case for each user, and by aggregating, for each demographic segment.  I built the transaction records as follows:
 - For each user, inspect the transcript to determine the number of each offer made.  (Some users are given the same offer more than once).
 - Build a dictionary of Offer: [(Time, Value)]
 - For each Offer type given to the user, inspect the transcript.  Record all transactions made between the offer time and (the offer time plus the offer duration) in that offer's dictionary.  This may need to be repeated for each instrance of a single offer.
 - For each transaction made by that user that is not in one of the offer periods, record the (time, value) in a no_offer_transactions.
 - The average transaction value ('B' case) for a given offer type to a given user is the average of all Value records in that offer type's dictionary entry
 - The transaction rate for a given offer type ('B' case) is determined using the offer duration, the number of times the offer is made to the user, and the number of transactions made during that offer type's intervals by that user
 - The baseline transaction rate ('A' case) is calculated as follows:
   - Process all offer intervals given to that user using a MERGE_INTERVALS algorithm
   - Sum the length af all merged intervals
   - Subtract from the total length of the experiment
   - Divide the number of no-offer transactions by the above time

As it turned out, the offers were all received within one of six time slots:

![placeholder_1](https://ismith1024.github.io/images/offer_periods.png)

*Fig 7a: Offers Recevied Time Slots*

![placeholder_1](https://ismith1024.github.io/images/transaction_times.png)

*Fig 7b: All transaction times*

Figure 7b illustrates two phenomena.  The first is the customer response to the six offer periods - it takes the form of an impulse response (with exponential decay), as opposed to a step response, with a logistic transition from one steady state to another.  The second phenomenon is that the last three offer periods are compressed in time and overlap each other -- the business driven by offer *k* has not declined back to its baseline state by the time that offer *k + 1* is made.

## Implementation

I segmented each user demographic into groups which would make sense to the Starbucks marketing team (for example, "users aged 20-30", "users with income 50k-60k", "Female users"). For each user demographic, I generated a histogram showing the distribution of change in transaction rate and value, as well as recording their mean values in a master table.

These mean values, as well as demographic data and offer parameters, were used to train a Linear Regression learner.

### Statistics

I segmented each demographic into logical bins, and then determined the transaction rate and value for each offer.  Next, I determined the transaction rate and average value for periods where no offer was valid for the same set of customers.  These two interval periods became my A and B groups.  I plottted a series of historgrams, and collected the A/B ratios in a matrix.

![placeholder_1](https://ismith1024.github.io/images/bogo_all_user.png)

|`BOGO 1 mean: 0.006450805435910441`|
|`BOGO 2 mean: 0.006798231448249598`|
|`BOGO 3 mean: 0.005380281062973571`|
|`BOGO 4 mean: 0.006058094776529127`|
|`No offer mean: 0.005528949447740996`|

*Fig 8: Example histogram - BOGO response, Transaction Rate, All Users*

The table of findings is too long to be placed inline, but see the appendix to this post for full details.  Notable heuristics are described as follows:

### Gender

Male users increased transaction frequency as a result of BOGO and discount offers more than Females or Others. However, Female users spent more per transaction than Males or Others as a result of BOGO or discount offers.  The BOGO plots are shown. 

![placeholder_1](https://ismith1024.github.io/images/bogo_tr_f.png)

|`Transaction rate, F, bogo_1: 0.005634396680460089`|
|`Transaction rate, F, bogo_2: 0.005967105624692851`|
|`Transaction rate, F, bogo_3: 0.004872737441403873`|
|`Transaction rate, F, bogo_4: 0.005495675598034395`|
|`Transaction rate, F, no offer: 0.005353211806736789`|

![placeholder_1](https://ismith1024.github.io/images/bogo_tr_m.png)

|`Transaction rate, M, bogo_1: 0.007023452187142654`|
|`Transaction rate, M, bogo_2: 0.007413135538135507`|
|`Transaction rate, M, bogo_3: 0.005742155732552141`|
|`Transaction rate, M, bogo_4: 0.006478328238981917`|
|`Transaction rate, M, no offer: 0.005654761369088241`|

*Fig 9: Male-Female discrepancy, transaction rates*

![placeholder_1](https://ismith1024.github.io/images/bogo_av_f.png)

|`Avg Transaction Value, F, bogo_1: 16.14544573773763`|
|`Avg Transaction Value, F, bogo_2: 15.48664369783282`|
|`Avg Transaction Value, F, bogo_3: 15.391220785472074`|
|`Avg Transaction Value, F, bogo_4: 15.621003896637408`|
|`Avg Transaction Value, F, no offer: 10.09996565897642`|

![placeholder_1](https://ismith1024.github.io/images/bogo_av_m.png)

|`Avg Transaction Value, M, bogo_1: 10.85762602097162`|
|`Avg Transaction Value, M, bogo_2: 10.518503135232647`|
|`Avg Transaction Value, M, bogo_3: 9.903239324149101`|
|`Avg Transaction Value, M, bogo_4: 9.863846278041088`|
|`Avg Transaction Value, M, no offer: 7.4150608793248445`|

*Fig 10: Male-Female discrepancy, average transaction values*


A slight decrease in transaction rate was recorded for Females as a result of Discount offers, compared to a slight increase for Males and Others. Discounts increased average transaction value significantly more for Females and Others than Males.

#### Age

The youngest users, those aged 18-30, were the most responsive to BOGO and discount in general.  BOGO offers are shown here to demonstrate this.

![placeholder_1](https://ismith1024.github.io/images/bogo_tr_age20.png)

|`Transaction Rate, age 20 - 30, bogo_1: 0.009038701697134851`|
|`Transaction Rate, age 20 - 30, bogo_2: 0.00961740230643825`|
|`Transaction Rate, age 20 - 30, bogo_3: 0.007327178790994582`|
|`Transaction Rate, age 20 - 30, bogo_4: 0.008295197356509156`|
|`Transaction Rate, age = 20 - 30, no offer: 0.006209470670028635`|

![placeholder_1](https://ismith1024.github.io/images/bogo_tr_age70.png)

|`Transaction Rate, age 70 - 80, bogo_1: 0.005635994223545941`|
|`Transaction Rate, age 70 - 80, bogo_2: 0.005855018405388468`|
|`Transaction Rate, age 70 - 80, bogo_3: 0.004974113506250731`|
|`Transaction Rate, age 70 - 80, bogo_4: 0.005289071284746735`|
|`Transaction Rate, age = 70 - 80, no offer: 0.005036440433212479`|

*Fig 11: Average transaction values - Variation by Age*

#### Income
The lowest-income customers had the highest response to discount and BOGO offers in terms of transaction volume.  I have illustrated this with discount offers.

![placeholder_1](https://ismith1024.github.io/images/inc20_disc_tr.png)

|`Transaction rate, income $20000 - $30000, discount_1: 0.006697530864197533`|
|`Transaction rate, income $20000 - $30000, discount_2: 0.006324404761904761`|
|`Transaction rate, income $20000 - $30000, discount_3: 0.006657088122605366`|
|`Transaction rate, income $20000 - $30000, discount_4: 0.008234126984126984`|
|`Transaction rate, Income = $20000 - $30000, no offer: 0.0057065606769648985`|

![placeholder_1](https://ismith1024.github.io/images/inc90_disc_tr.png)

|`Transaction rate, income $80000 - $90000, discount_1: 0.0036302064685523362`|
|`Transaction rate, income $80000 - $90000, discount_2: 0.004169033354082362`|
|`Transaction rate, income $80000 - $90000, discount_3: 0.0038488640356316066`|
|`Transaction rate, income $80000 - $90000, discount_4: 0.003610782193657642`|
|`Transaction rate, Income = $80000 - $90000, no offer: 0.004310092463914752`|

*Fig 12: Average transaction values - Variation by Age*


#### Membership date

A non-linear trend was apparent which depended on the date a member joined.  The newest and oldest memberships (those joining in 2013 or 2018) showed the least responsiveness to offers in terms of average transaction value.  The customers who spent the most in response to an offer were those who joined in 2015-2016.

 Membership Date | bogo_1  | bogo_2  | bogo_3  | bogo_4 | discount_1  | discount_2  | discount_3  | discount_4
---|---|---|---|---|---|---|---|---
2013|1.0289128217 | 1.3108907323 | 1.1004880489 | 1.0051803556 | 1.2970263108 | 1.3441169459 | 1.4216113414 | 1.3486408789
2014|1.3613533339 | 1.2131313491 | 1.3138951131 | 1.5422741782 | 1.5535381775 | 1.400929061 | 1.53353075 | 1.7482361107
2015|1.8607760224 | 1.789786318 | 1.9200245803 | 1.5945606834 | 1.6768473232 | 1.6013653731 | 1.7543372449 | 1.7783641652
2016|1.7026097382 | 1.8055138487 | 1.6188923012 | 1.6029436568 | 1.835628979 | 1.729043692 | 1.8618566462 | 1.7267409361
2017|1.4646361715 | 1.3936461882 | 1.3511012608 | 1.4652818338 | 1.576746049 | 1.5886497657 | 1.6497503595 | 1.5083676209
2018|1.3112040047 | 1.1154332998 | 1.1095564393 | 1.1256440277 | 1.5295580675 | 1.3316919312 | 1.5829053101 | 1.2879490886

*Fig 13: Change in average transaction value, by membership year*

Figure 13. illustrates this non-linear effect.

#### Reward and difficulty

I created a series of scatter plots to examine the relationship between demographic segment and transaction rate or value by reward and difficulty.

![placeholder_1](https://ismith1024.github.io/images/dif_tr_age.png)

![placeholder_1](https://ismith1024.github.io/images/rt_tr_age.png)

*Fig 14: Reward by age; and difficulty by income; change in transaction rate*

Younger and lower-income customers increased transaction rate, but independently of reward and difficuly. There was no noticeable transaction value trend correlated to either income or age.


### Refinement 

I implemented the regression learner incrementally, watching for performance after each change.
I built a dataframe which essentially consisted of augmented X and y matrices:

 | X_user_demographics | X_offer_parameters | y_Average_Value_Change | y_Transaction_Rate_Change |

First, I ran the raw data through a linear regressor, mostly to establish a baseline and determine the correct workflow and syntax.  Next, I scaled and normalized the data for each user demographic.  The results were, predicatbly, a great improvement.

I evaluated both RandomForestRegressor and AdaboostRegressor learners, but the LinearRegression model performed the best.

After establishing the regression algorithm, I added offer reward, offer difficulty, and then offer channels one at a time to the dataframe.  None of these columns had a material effect on the performance of the regression model, which suggests to me that the information contained in these data fields does not contribute information to the system.

Finally, I observed that membership year has a non-linear correlation to average transaction value.  Users who joined in 2015 and 2016 spent noticeably more per visit than users who joined in 2013 and 2018.  I thought of using a non-linear statistical model, but instead one-hot encoded membership year and re-ran the regressor.  The result was in improvement in average transaction value with a trade-off in perfomance of transaction rate.  If Starbucks were interested in using these regressors, they may wish to consider encoding membership year separately for the separate y-values.

## Results

#### Model evaluation and Validation

**Heuristic Solution**
The results of the statistical analysis are provided at the end of this post in a table, and in a data dump titled "results.csv".  The results can be sorted or filtered on offer and demographic, to produce a table of roughly ten rows that can visualize the relationship between any of the numerous dimensions of this problem.

 - Gender - Starbucks should target marketing campaigns intended to drive larger purchases towards Female users, and high transaction volume toward Male users, since it is shown that these demographics will respond in kind.
 - Income - Upper income customers spend more in general, but do not show a proportionally higher increase in transaction values than lower earners when given an offer.  The lowest-income earners are the most responsive in terms of transaction volumes when given a BOGO or discount offer.  Starbucks should bear in mind that income is correlated to both gender and age, and any spurous correlations should be ruled out before investing too heavily on a marketing campaign.
 - Age - Starbucks should direct offers to younger users, particularly those with lower income.  Both these demographics are shown to increase transaction rates when given offers.
 - Membership date - Starbucks should consider that the users who have been members for the median amount of time are the most receptive to offers.  The newest and the longest-serving app users will spend the least money.
 - Channel, Reward, Difficulty - There was no demonstrated change in behavior that depended on these variables.  Starbucks should review the way that these values are evaluated; difficulties are described in a subsequent section.

**Machine Learning Solution**

I evaluated the predictive models using the R-squared and MSE matrics, and quantified the effects of the items previously described.

Model | R^2 - Transaction Rate  | MSE - Transaction Rate | R^2 - Transaction Value | MSE - Transaction Value
---|---|---|---|---
LinearRegressor| 0.2144105603152586 | 2.3226821404539195e-05 | 0.06405297615851169 | 600.566649797936
RandomForestRegressor |  -0.01370380978036745 | 277.2203874089114 | -1.652772833721584 | 725.4611313253996
AdaboostRegressor | -0.47567024202882435 | 0.00011169151421061815 | -0.5928741397840689 | 435.6077010505166
---|---|---|---|---
Base data, drop invalid| 0.2144105603152586 | 2.3226821404539195e-05 | 0.06405297615851169 | 600.566649797936
Base data, imputed| 0.2403796150694231 | 2.2676661733016904e-05 |  0.08010667319942033 | 574.3614592610678
Normalize and Scale | 0.24282416374984062| 2.2603685540414085e-05 |  0.08019900111557599 | 574.3038117109567
Difficulty /Reward | 0.2403796150694231 | 2.2676661733016904e-05 | 0.08010667319942266 | 574.3614592610663
Channels |  0.24282416374984062 | 2.2603685540414085e-05 |  0.08019900111557599 | 574.3038117109567
One-hot Membership Date | 0.2290706546154977 | 2.3014263877263163e-05 |  0.08279156474044091 | 572.6850711641765

*Fig 15 : Incremental learner improvement (BOGO used for illustrative purposes)

#### Justification
The statistical analysis shows clear, if unsystematic, cases where one demographic segment responds more favorably when given the same offer type than others.  Starbucks marketing team is able to use these values to direct future marketing campaigns.

After comparing the model metrics, the Linear Regression worked best as a learner to predict user behavior.  The optimal parameters were:
 - Selection of algorithm: Linear Regression was suitable for the task; the AdaBoost and Random Forest regressors were not.  This step represented the greatest improvement potential
 - Impute data, as opposed to dropping invalid responses.  This step represented the next most significant improvement potiential.
 - Normalize and scale values
 - Encoding difficulty and reward, and encoding channels had little effect on the quality of outcome
 - One-hotting the membership date improved performance for transaction values and hindered performance for transaction rate.

## Conclusions

#### Reflection

**Heuristic Solution**
The results of the statistical analysis are straightforward conceptually, but dense in terms of volume of information.  Starbucks' marketing team is directed to the data dump in "findings.csv".

Since the findings are comprehensive but at times cumbersome to navigate, I would suggest an approach that the marketing team may wish to consider would be to open the results.csv file in a spreadsheet program and autofilter the data.  This has the advantage of making the findings accessible to a typical office team member who is not experienced with data science tools.

**Machine Learning Solution**
The most significant factors in creating a regression model were the selection of algorithm and the decison on how to manage invalid data entry.

Difficulty, reward, and channels had minimal impact on the results.  Starbucks should consider either omitting them from the regressor, or re-evalueating them with better rigor in a subsequent run of this experiment.

One-hotting the membership date improved performance for transaction vlaues and hindered performance for transaction rate.  Starbucks should consider training separate regressors using distinct membership date encoding to optimize performance if a machine learning technique is to be implemented for the purpose of predicting both transaction rate and value.

The regression learning was more successful on the transaction rate than change of transaction value.  R-squared approaches 25% for the most successful transaction rate for model, and 8% for value.  However, putting it in perspective, even a small improvement in predictive power is often suitable for business use.  I have seen claims that 65% accuracy on classifiers, for example, will often yield a significant business advantage.

#### Opportunities for improvement
Starbucks could improve the data set in the following ways:
- Run the experiment with a continuous variation of reward and difficulty.  I found that the transcript set was not informative with respect to these parameters - there were only two values for the BOGO and discount offers, this made finding a relationship between difficulty and income, for example, unreliable
- Run the experiment with separable information.  Channel information was specifically bad.  For example, if all four offers of a type are available by web, it is not possible to measure the effect of web vs no web.  Similarly, if two offers are made by social with difficulty X, and two others not by social with difficulty Y, it is not possible to attribute any difference to the channel, or the difficulty.

I could improve the experiment by:
- Considering whether the offer was opened.  I dismissed this value early on, but given more time to complete the exercise, the next thing I would have added would be a binary attribute "offer opened" to the learner.  I suspect that this would have improved the results.
- Revisting the non-responsive data.  I could have created a binary variable which flags non-responsive users for age, income, or gender.  The possible improvement on the regression learner is unknown.

∎

![placeholder_1](https://ismith1024.github.io/images/xkcd.png)

## Appendix - All Results

Offer | Demographic Segment | Transaction Rate | Average Value | Change in Transaction Rate | Change in Average Transaction Value
---|---|---|---|---|---
bogo_1 | gender_F | 0.0056343967 | 16.1454457377 | 1.0525263868 | 1.5985644192
bogo_2 | gender_F | 0.0059671056 | 15.4866436978 | 1.1146776627 | 1.5333362727
bogo_3 | gender_F | 0.0048727374 | 15.3912207855 | 0.9102455904 | 1.5238884275
bogo_4 | gender_F | 0.0054956756 | 15.6210038966 | 1.0266127694 | 1.5466393079
discount_1 | gender_F | 0.0050566553 | 16.4494842879 | 0.9536793763 | 1.6693554661
discount_2 | gender_F | 0.0054749266 | 16.4492722574 | 1.0325648639 | 1.6693339485
discount_3 | gender_F | 0.0050948994 | 17.1841426974 | 0.9608921717 | 1.7439113616
discount_4 | gender_F | 0.0049773211 | 15.1143101192 | 0.9387170449 | 1.5338569753
info_1 | gender_F | 0.007479543 | 12.2190099432 | 1.3971297348 | 1.1270599864
info_2 | gender_F | 0.008000551 | 11.5137538533 | 1.4944506185 | 1.0620084051
bogo_1 | gender_M | 0.0070234522 | 10.857626021 | 1.2420421886 | 1.4642666052
bogo_2 | gender_M | 0.0074131355 | 10.5185031352 | 1.3109546194 | 1.4185322692
bogo_3 | gender_M | 0.0057421557 | 9.9032393241 | 1.0154550047 | 1.3355573859
bogo_4 | gender_M | 0.0064783282 | 9.863846278 | 1.1456413129 | 1.3302448137
discount_1 | gender_M | 0.0063423399 | 11.3042768711 | 1.1238703513 | 1.5913158434
discount_2 | gender_M | 0.0058367468 | 10.3296725538 | 1.0342786327 | 1.4541196911
discount_3 | gender_M | 0.0056303811 | 11.6977819257 | 0.9977103922 | 1.6467099951
discount_4 | gender_M | 0.0059974089 | 11.2497341601 | 1.0627481525 | 1.5836378043
info_1 | gender_M | 0.0082067155 | 8.1248855759 | 1.4583215708 | 1.050794499
info_2 | gender_M | 0.0087305295 | 7.9734916314 | 1.5514025536 | 1.0312146634
bogo_1 | gender_O | 0.0069202788 | 11.8041452853 | 1.2386572945 | 1.3752218301
bogo_2 | gender_O | 0.0056294803 | 11.4776693548 | 1.007617905 | 1.3371863082
bogo_3 | gender_O | 0.0056236157 | 12.1818151993 | 1.00656821 | 1.4192216198
bogo_4 | gender_O | 0.0054374098 | 11.8704393939 | 0.9732393053 | 1.3829453122
discount_1 | gender_O | 0.0055649608 | 17.915349361 | 1.0348172631 | 2.1763715102
discount_2 | gender_O | 0.0058274544 | 14.1477549971 | 1.0836285527 | 1.7186810198
discount_3 | gender_O | 0.0053186085 | 11.9172717853 | 0.9890074876 | 1.4477200679
discount_4 | gender_O | 0.0047469559 | 11.9927306438 | 0.8827073575 | 1.4568868726
info_1 | gender_O | 0.0068908608 | 11.4761463845 | 1.3628496497 | 1.442506692
info_2 | gender_O | 0.0083559169 | 8.387351626 | 1.6526031788 | 1.0542572779
bogo_1 | income_110_120 | 0.0033463674 | 23.039387538 | 0.8092095263 | 1.5182823143
bogo_2 | income_110_120 | 0.0043107418 | 23.2586288056 | 1.0424119403 | 1.5327301871
bogo_3 | income_110_120 | 0.0033571794 | 22.5126779627 | 0.8118240556 | 1.4835724579
bogo_4 | income_110_120 | 0.0041510116 | 27.8199871318 | 1.0037864162 | 1.8333210628
discount_1 | income_110_120 | 0.003685308 | 27.5729872398 | 0.8866534768 | 1.8562494184
discount_2 | income_110_120 | 0.0043496328 | 24.7615489244 | 1.046484326 | 1.6669797287
discount_3 | income_110_120 | 0.0040150463 | 24.7766019608 | 0.9659856927 | 1.6679931188
discount_4 | income_110_120 | 0.0033430233 | 19.7198903193 | 0.8043027146 | 1.327568704
info_1 | income_110_120 | 0.0070344258 | 17.6219623984 | 1.7020761637 | 1.1489342254
info_2 | income_110_120 | 0.0066632664 | 15.1488587571 | 1.6122690278 | 0.9876903553
bogo_1 | income_20_30 | 0.0129464286 | 4.707979828 | 2.3430420728 | 1.586672137
bogo_2 | income_20_30 | 0.0106277778 | 3.9755619048 | 1.9234131125 | 1.3398343947
bogo_3 | income_20_30 | 0.0069444444 | 4.046412013 | 1.256804177 | 1.3637121293
bogo_4 | income_20_30 | 0.0072008547 | 4.3937678063 | 1.3032092543 | 1.4807771505
discount_1 | income_20_30 | 0.0066975309 | 3.3669708995 | 1.1736545431 | 1.1173266999
discount_2 | income_20_30 | 0.0063244048 | 4.6832546769 | 1.1082690818 | 1.5541344576
discount_3 | income_20_30 | 0.0066570881 | 4.435045977 | 1.1665674825 | 1.4717665917
discount_4 | income_20_30 | 0.008234127 | 4.1249 | 1.442922883 | 1.3688448881
info_1 | income_20_30 | 0.012012012 | 4.2257207207 | 2.2047156673 | 1.5637768135
info_2 | income_20_30 | 0.0118856838 | 3.625525641 | 2.1815290542 | 1.3416676844
bogo_1 | income_30_40 | 0.0092222557 | 5.332387794 | 1.3998805767 | 1.4754595115
bogo_2 | income_30_40 | 0.0096925204 | 5.3844016638 | 1.4712638007 | 1.4898516303
bogo_3 | income_30_40 | 0.0074542154 | 5.1133244981 | 1.1315031435 | 1.4148451983
bogo_4 | income_30_40 | 0.0081300783 | 5.0378236921 | 1.234094889 | 1.3939542979
discount_1 | income_30_40 | 0.0082772446 | 5.473665427 | 1.2544221892 | 1.5550817379
discount_2 | income_30_40 | 0.0074914419 | 5.6912754451 | 1.1353332384 | 1.6169052764
discount_3 | income_30_40 | 0.0069455312 | 5.4925408875 | 1.0526000939 | 1.5604443024
discount_4 | income_30_40 | 0.0072873415 | 5.680225182 | 1.1044016824 | 1.6137658697
info_1 | income_30_40 | 0.0093595158 | 3.9089574485 | 1.5210346855 | 1.0913248571
info_2 | income_30_40 | 0.0109935036 | 4.3648722958 | 1.7865774988 | 1.2186097437
bogo_1 | income_40_50 | 0.0084630064 | 6.4330004895 | 1.3632519668 | 1.4309985301
bogo_2 | income_40_50 | 0.0088068675 | 6.2051915867 | 1.418642365 | 1.3803232339
bogo_3 | income_40_50 | 0.0066405584 | 5.8931042429 | 1.0696853988 | 1.3109004924
bogo_4 | income_40_50 | 0.0078058887 | 7.0956264879 | 1.2574010485 | 1.5783973732
discount_1 | income_40_50 | 0.0075770809 | 6.6123749459 | 1.2191707648 | 1.5276718854
discount_2 | income_40_50 | 0.0069754647 | 6.1940096616 | 1.1223692513 | 1.4310160109
discount_3 | income_40_50 | 0.0069211961 | 6.9959997104 | 1.1136373016 | 1.6163015792
discount_4 | income_40_50 | 0.0074470188 | 6.8625033766 | 1.1982434544 | 1.5854596204
info_1 | income_40_50 | 0.0092808588 | 5.0320858093 | 1.5069216613 | 1.0138852819
info_2 | income_40_50 | 0.0104419192 | 5.5055490791 | 1.6954416198 | 1.1092806029
bogo_1 | income_50_60 | 0.0070644637 | 10.0020209585 | 1.200712875 | 1.5641259016
bogo_2 | income_50_60 | 0.0073404098 | 9.0023432334 | 1.2476141073 | 1.4077953131
bogo_3 | income_50_60 | 0.0057809794 | 8.5297190019 | 0.9825652319 | 1.3338858697
bogo_4 | income_50_60 | 0.006583003 | 9.3951316093 | 1.1188813178 | 1.4692199467
discount_1 | income_50_60 | 0.0061490901 | 10.9182012568 | 1.0364116108 | 1.7837400684
discount_2 | income_50_60 | 0.0058830562 | 9.35120297 | 0.9915723512 | 1.5277347462
discount_3 | income_50_60 | 0.0056858108 | 9.8585760599 | 0.9583272096 | 1.610625846
discount_4 | income_50_60 | 0.0058250753 | 9.5407948675 | 0.9817998348 | 1.5587089567
info_1 | income_50_60 | 0.0082565104 | 7.1262617817 | 1.3547207623 | 1.0887795799
info_2 | income_50_60 | 0.009034709 | 7.1142559486 | 1.4824068854 | 1.0869452793
bogo_1 | income_60_70 | 0.0068934122 | 11.7390251886 | 1.1843354536 | 1.4278324014
bogo_2 | income_60_70 | 0.0074089192 | 10.5708799233 | 1.2729030871 | 1.2857494233
bogo_3 | income_60_70 | 0.0055834802 | 11.074266106 | 0.9592801615 | 1.3469769179
bogo_4 | income_60_70 | 0.006459161 | 10.7299213764 | 1.1097281165 | 1.3050938353
discount_1 | income_60_70 | 0.0062638324 | 11.8621742407 | 1.1004898464 | 1.5206642773
discount_2 | income_60_70 | 0.0059242903 | 11.5251347134 | 1.0408358495 | 1.4774576982
discount_3 | income_60_70 | 0.0056763976 | 11.6948015787 | 0.9972836914 | 1.4992080398
discount_4 | income_60_70 | 0.0059175798 | 11.2582431395 | 1.0396568827 | 1.4432436938
info_1 | income_60_70 | 0.0085800439 | 9.4914810855 | 1.4592231525 | 1.1099661092
info_2 | income_60_70 | 0.0086238035 | 9.4490510937 | 1.4666654235 | 1.1050042015
bogo_1 | income_70_80 | 0.0051851735 | 14.7729471953 | 1.0121856982 | 1.4151171989
bogo_2 | income_70_80 | 0.0053501679 | 15.11192827 | 1.0443938723 | 1.4475885766
bogo_3 | income_70_80 | 0.0044646825 | 16.1293343581 | 0.8715403251 | 1.5450470482
bogo_4 | income_70_80 | 0.0049067426 | 13.9491289712 | 0.957833844 | 1.3362027262
discount_1 | income_70_80 | 0.0046878071 | 15.0097917488 | 0.9279422753 | 1.4373586584
discount_2 | income_70_80 | 0.0048814304 | 14.8165568216 | 0.9662696352 | 1.4188542114
discount_3 | income_70_80 | 0.0045933475 | 17.940144461 | 0.9092441741 | 1.7179733341
discount_4 | income_70_80 | 0.0047317398 | 16.1309920193 | 0.936638656 | 1.544726365
info_1 | income_70_80 | 0.0067194672 | 12.3619911618 | 1.3476064949 | 1.1175561458
info_2 | income_70_80 | 0.0073279436 | 11.4845747305 | 1.469638005 | 1.0382354189
bogo_1 | income_80_90 | 0.0037784497 | 20.0730590074 | 0.8599923251 | 1.4798146574
bogo_2 | income_80_90 | 0.0039247145 | 22.1291958717 | 0.8932828811 | 1.6313960117
bogo_3 | income_80_90 | 0.0035490566 | 20.6168955048 | 0.8077814219 | 1.5199070628
bogo_4 | income_80_90 | 0.0038909375 | 16.673111794 | 0.8855950731 | 1.2291656796
discount_1 | income_80_90 | 0.0036302065 | 23.101392254 | 0.8422572135 | 1.7975469818
discount_2 | income_80_90 | 0.0041690334 | 22.5210266573 | 0.9672723704 | 1.7523880401
discount_3 | income_80_90 | 0.003848864 | 23.6182450893 | 0.8929887393 | 1.8377639196
discount_4 | income_80_90 | 0.0036107822 | 20.5193815159 | 0.8377505179 | 1.5966376358
info_1 | income_80_90 | 0.0060436846 | 16.5893325222 | 1.328678169 | 1.2018589189
info_2 | income_80_90 | 0.0058364101 | 13.5701931743 | 1.2831097623 | 0.9831292294
bogo_1 | income_90_100 | 0.0038610257 | 24.7238110278 | 0.8667271178 | 1.839201107
bogo_2 | income_90_100 | 0.0038233589 | 23.891360742 | 0.8582716198 | 1.7772752378
bogo_3 | income_90_100 | 0.0035684086 | 21.1949335725 | 0.801040114 | 1.5766883691
bogo_4 | income_90_100 | 0.003959069 | 23.7344419324 | 0.888735982 | 1.7656020678
discount_1 | income_90_100 | 0.0033179767 | 23.166638608 | 0.7600056513 | 1.6676591983
discount_2 | income_90_100 | 0.0041485611 | 22.5269887419 | 0.9502567748 | 1.621613762
discount_3 | income_90_100 | 0.0036201525 | 24.7631668865 | 0.8292211137 | 1.7825858873
discount_4 | income_90_100 | 0.0036512001 | 21.2625610927 | 0.8363327913 | 1.5305934618
info_1 | income_90_100 | 0.0059539098 | 17.2460570233 | 1.3554302566 | 1.1050109979
info_2 | income_90_100 | 0.005710207 | 15.3211223616 | 1.2999503805 | 0.9816741698
bogo_1 | income_100_110 | 0.0035970738 | 26.5277310406 | 0.8424675628 | 1.5335878201
bogo_2 | income_100_110 | 0.0042007937 | 23.0887597619 | 0.9838642838 | 1.3347783381
bogo_3 | income_100_110 | 0.003601709 | 21.3708416957 | 0.8435531752 | 1.2354642197
bogo_4 | income_100_110 | 0.0040979776 | 23.2157992305 | 0.9597838247 | 1.3421225841
discount_1 | income_100_110 | 0.0034070933 | 27.7358438095 | 0.7932643345 | 1.8786266209
discount_2 | income_100_110 | 0.0041882209 | 24.03487354 | 0.9751321857 | 1.6279495072
discount_3 | income_100_110 | 0.0038919035 | 27.7813265818 | 0.9061414041 | 1.8817072968
discount_4 | income_100_110 | 0.0035493326 | 27.6371838745 | 0.8263815406 | 1.8719441063
info_1 | income_100_110 | 0.0060435669 | 16.0231467662 | 1.4159909556 | 0.8339285631
info_2 | income_100_110 | 0.0058434678 | 18.5591600332 | 1.3691083053 | 0.9659159892
bogo_1 | joined_2013 | 0.0122249278 | 6.9318094123 | 1.7544493408 | 1.0289128217
bogo_2 | joined_2013 | 0.0120787495 | 8.8315011001 | 1.7334706946 | 1.3108907323
bogo_3 | joined_2013 | 0.0087017699 | 7.4140133688 | 1.2488265507 | 1.1004880489
bogo_4 | joined_2013 | 0.0108591871 | 6.7719232403 | 1.5584463074 | 1.0051803556
discount_1 | joined_2013 | 0.0094726348 | 8.8700464404 | 1.3973098913 | 1.2970263108
discount_2 | joined_2013 | 0.0087651644 | 9.192087803 | 1.292950823 | 1.3441169459
discount_3 | joined_2013 | 0.0084109615 | 9.7220530635 | 1.2407022909 | 1.4216113414
discount_4 | joined_2013 | 0.0081121858 | 9.2230258763 | 1.1966298507 | 1.3486408789
info_1 | joined_2013 | 0.0116898148 | 6.4145436508 | 1.6525490308 | 0.9717475312
info_2 | joined_2013 | 0.0132127796 | 4.7719872702 | 1.8678453412 | 0.7229145362
bogo_1 | joined_2014 | 0.012133663 | 8.0779328847 | 1.7010712533 | 1.3613533339
bogo_2 | joined_2014 | 0.0129333333 | 7.1984204058 | 1.8131805344 | 1.2131313491
bogo_3 | joined_2014 | 0.0086537958 | 7.7963275785 | 1.2132134627 | 1.3138951131
bogo_4 | joined_2014 | 0.0096730693 | 9.1514722821 | 1.3561098747 | 1.5422741782
discount_1 | joined_2014 | 0.0096163764 | 8.8191703331 | 1.3646371344 | 1.5535381775
discount_2 | joined_2014 | 0.0086712802 | 7.9528344988 | 1.2305207748 | 1.400929061
discount_3 | joined_2014 | 0.0077779237 | 8.7055915916 | 1.1037466761 | 1.53353075
discount_4 | joined_2014 | 0.0082926641 | 9.9244371756 | 1.176792263 | 1.7482361107
info_1 | joined_2014 | 0.0114739924 | 6.3258654995 | 1.6761624821 | 0.9921431308
info_2 | joined_2014 | 0.0129339067 | 5.8964929282 | 1.8894320709 | 0.9248007178
bogo_1 | joined_2015 | 0.0088952599 | 17.0999430185 | 1.4538476586 | 1.8607760224
bogo_2 | joined_2015 | 0.0092233041 | 16.447570092 | 1.5074634333 | 1.789786318
bogo_3 | joined_2015 | 0.0072831122 | 17.6444185238 | 1.1903570826 | 1.9200245803
bogo_4 | joined_2015 | 0.0079329728 | 14.6535082667 | 1.2965707656 | 1.5945606834
discount_1 | joined_2015 | 0.0067894974 | 15.3152493645 | 1.102562125 | 1.6768473232
discount_2 | joined_2015 | 0.0076111369 | 14.6258455816 | 1.2359900533 | 1.6013653731
discount_3 | joined_2015 | 0.0071800144 | 16.0229926736 | 1.1659790782 | 1.7543372449
discount_4 | joined_2015 | 0.0070735858 | 16.242439174 | 1.148695894 | 1.7783641652
info_1 | joined_2015 | 0.0100208948 | 11.0075424779 | 1.5775357303 | 1.1409826245
info_2 | joined_2015 | 0.0111437307 | 13.0914498949 | 1.7542977735 | 1.3569892544
bogo_1 | joined_2016 | 0.0072436761 | 17.1772221847 | 1.2365269934 | 1.7026097382
bogo_2 | joined_2016 | 0.0079862213 | 18.2153971291 | 1.3632827014 | 1.8055138487
bogo_3 | joined_2016 | 0.005986636 | 16.3326169978 | 1.0219447963 | 1.6188923012
bogo_4 | joined_2016 | 0.007512234 | 16.1717149416 | 1.2823709939 | 1.6029436568
discount_1 | joined_2016 | 0.0064914958 | 18.4674974735 | 1.1133532496 | 1.835628979
discount_2 | joined_2016 | 0.0069719135 | 17.3951873608 | 1.1957494459 | 1.729043692
discount_3 | joined_2016 | 0.0066641924 | 18.7313630934 | 1.1429723597 | 1.8618566462
discount_4 | joined_2016 | 0.0062926044 | 17.3720202942 | 1.0792414828 | 1.7267409361
info_1 | joined_2016 | 0.0098136006 | 14.016312209 | 1.6696995366 | 1.2646470012
info_2 | joined_2016 | 0.0100321806 | 13.0392975045 | 1.7068890293 | 1.1764940906
bogo_1 | joined_2017 | 0.0058983302 | 12.7934209084 | 1.1080337709 | 1.4646361715
bogo_2 | joined_2017 | 0.0061791493 | 12.1733319377 | 1.1607871878 | 1.3936461882
bogo_3 | joined_2017 | 0.0050840997 | 11.8017071109 | 0.9550760887 | 1.3511012608
bogo_4 | joined_2017 | 0.0056369253 | 12.7990606918 | 1.0589274246 | 1.4652818338
discount_1 | joined_2017 | 0.0054454753 | 13.4019689224 | 1.0297561697 | 1.576746049
discount_2 | joined_2017 | 0.0053632054 | 13.5031477016 | 1.0141986678 | 1.5886497657
discount_3 | joined_2017 | 0.0051211492 | 14.0224883147 | 0.9684250971 | 1.6497503595
discount_4 | joined_2017 | 0.0051453494 | 12.8207684376 | 0.9730014315 | 1.5083676209
info_1 | joined_2017 | 0.007309679 | 10.1587984839 | 1.4016642985 | 1.1359940452
info_2 | joined_2017 | 0.0079989054 | 9.4182429331 | 1.533826599 | 1.0531824118
bogo_1 | joined_2018 | 0.0041022642 | 9.5528329218 | 0.8343137938 | 1.3112040047
bogo_2 | joined_2018 | 0.0041969563 | 8.1265370684 | 0.8535721547 | 1.1154332998
bogo_3 | joined_2018 | 0.0036486172 | 8.0837209497 | 0.7420515882 | 1.1095564393
bogo_4 | joined_2018 | 0.0037555686 | 8.2009277637 | 0.7638032271 | 1.1256440277
discount_1 | joined_2018 | 0.0043428647 | 10.1449779856 | 0.8903893219 | 1.5295580675
discount_2 | joined_2018 | 0.003664687 | 8.8326070209 | 0.7513469524 | 1.3316919312
discount_3 | joined_2018 | 0.0034473907 | 10.4988099937 | 0.7067961091 | 1.5829053101
discount_4 | joined_2018 | 0.0042436427 | 8.5424773525 | 0.8700464745 | 1.2879490886
info_1 | joined_2018 | 0.0054099422 | 6.6012666187 | 1.1010713103 | 0.8419219551
info_2 | joined_2018 | 0.0053041278 | 5.8358210926 | 1.0795351876 | 0.7442974489
bogo_1 | age_10_20 | 0.0091608088 | 6.1947699901 | 1.4668391219 | 1.238004861
bogo_2 | age_10_20 | 0.009517053 | 5.8157339952 | 1.5238813575 | 1.1622557363
bogo_3 | age_10_20 | 0.0069056755 | 5.0097672355 | 1.1057446164 | 1.0011858713
bogo_4 | age_10_20 | 0.0082938559 | 5.5131071736 | 1.3280216378 | 1.1017767392
discount_1 | age_10_20 | 0.0078712121 | 6.423182154 | 1.2657510848 | 1.2001467325
discount_2 | age_10_20 | 0.007698051 | 10.2348666126 | 1.2379054541 | 1.9123452252
discount_3 | age_10_20 | 0.0069708377 | 6.2179295642 | 1.1209639881 | 1.1617960803
discount_4 | age_10_20 | 0.0072217713 | 6.1023753307 | 1.1613160333 | 1.1402052188
info_1 | age_10_20 | 0.0100990854 | 4.5574369919 | 1.6300264218 | 0.9519645123
info_2 | age_10_20 | 0.0109176341 | 5.5947425373 | 1.762142939 | 1.1686385046
bogo_1 | age_20_30 | 0.0090387017 | 7.3381420349 | 1.4556315953 | 1.592173808
bogo_2 | age_20_30 | 0.0096174023 | 7.840010388 | 1.5488280431 | 1.7010653561
bogo_3 | age_20_30 | 0.0073271788 | 6.9101631915 | 1.1800005476 | 1.4993142392
bogo_4 | age_20_30 | 0.0082951974 | 7.5479193435 | 1.3358944421 | 1.6376896803
discount_1 | age_20_30 | 0.0079301809 | 7.2313915214 | 1.28920845 | 1.6021194831
discount_2 | age_20_30 | 0.0071300416 | 6.5183658431 | 1.1591299163 | 1.4441481815
discount_3 | age_20_30 | 0.0070082422 | 7.0637639315 | 1.1393290043 | 1.5649814818
discount_4 | age_20_30 | 0.0075189434 | 7.3265116378 | 1.2223536268 | 1.6231934066
info_1 | age_20_30 | 0.0103353726 | 7.1132527794 | 1.7054981699 | 1.4792753995
info_2 | age_20_30 | 0.0107466905 | 5.941729083 | 1.7733720569 | 1.2356447797
bogo_1 | age_30_40 | 0.0082150613 | 9.9228689947 | 1.3056988466 | 1.6437369323
bogo_2 | age_30_40 | 0.008654165 | 8.5242537789 | 1.3754898357 | 1.4120543932
bogo_3 | age_30_40 | 0.0065328505 | 8.8564186463 | 1.0383288841 | 1.4670779615
bogo_4 | age_30_40 | 0.0070894534 | 9.5657466676 | 1.1267951461 | 1.5845791264
discount_1 | age_30_40 | 0.0071536724 | 9.9067366039 | 1.1220385589 | 1.5421695915
discount_2 | age_30_40 | 0.0069959185 | 8.8666447274 | 1.097295184 | 1.380259759
discount_3 | age_30_40 | 0.0068049568 | 10.1618447688 | 1.0673432476 | 1.581881968
discount_4 | age_30_40 | 0.0069524228 | 8.3283174801 | 1.0904729763 | 1.2964590137
info_1 | age_30_40 | 0.0093051464 | 7.0459450457 | 1.4718316535 | 1.0582256531
info_2 | age_30_40 | 0.0099782788 | 8.3709893732 | 1.57830366 | 1.2572331517
bogo_1 | age_40_50 | 0.0061958444 | 12.192168139 | 1.1375166145 | 1.475926258
bogo_2 | age_40_50 | 0.0066215063 | 11.7086074733 | 1.2156653656 | 1.417388689
bogo_3 | age_40_50 | 0.0053324885 | 11.6970317469 | 0.9790100943 | 1.4159873862
bogo_4 | age_40_50 | 0.0058305413 | 11.1908961502 | 1.0704493431 | 1.3547170028
discount_1 | age_40_50 | 0.0058305902 | 11.3720242678 | 1.0827544273 | 1.334145775
discount_2 | age_40_50 | 0.0054848189 | 12.2111816505 | 1.0185438767 | 1.432594235
discount_3 | age_40_50 | 0.0049780891 | 12.5374094512 | 0.924442948 | 1.4708667036
discount_4 | age_40_50 | 0.0053805123 | 11.7575077325 | 0.9991738935 | 1.3793700133
info_1 | age_40_50 | 0.0074284365 | 9.7837546881 | 1.3550926298 | 1.0906607032
info_2 | age_40_50 | 0.0080139796 | 8.9371812047 | 1.4619071876 | 0.9962874835
bogo_1 | age_50_60 | 0.0057049805 | 14.8739401608 | 1.0502021296 | 1.5967041919
bogo_2 | age_50_60 | 0.0061066236 | 15.2421647415 | 1.1241386466 | 1.6362327718
bogo_3 | age_50_60 | 0.0048239995 | 14.1218981002 | 0.8880266112 | 1.5159731484
bogo_4 | age_50_60 | 0.0055187772 | 14.2724570169 | 1.0159248696 | 1.5321355136
discount_1 | age_50_60 | 0.0050988834 | 14.7196695583 | 0.9479832541 | 1.7043271545
discount_2 | age_50_60 | 0.0050684063 | 14.5760364838 | 0.9423169651 | 1.6876964993
discount_3 | age_50_60 | 0.0049712177 | 16.4726221085 | 0.9242476799 | 1.9072939819
discount_4 | age_50_60 | 0.004991118 | 14.5909397979 | 0.9279475383 | 1.6894220898
info_1 | age_50_60 | 0.0072807119 | 11.4183691091 | 1.3715321347 | 1.1482660421
info_2 | age_50_60 | 0.007715984 | 11.0371276901 | 1.4535282009 | 1.1099272416
bogo_1 | age_60_70 | 0.0058016688 | 14.950388961 | 1.1029735991 | 1.5765267577
bogo_2 | age_60_70 | 0.0059154876 | 14.1244189911 | 1.124612036 | 1.489427769
bogo_3 | age_60_70 | 0.0047680333 | 14.3463092749 | 0.9064658758 | 1.5128262218
bogo_4 | age_60_70 | 0.0055595598 | 12.9265848325 | 1.0569454766 | 1.3631154967
discount_1 | age_60_70 | 0.0052359492 | 17.0587145262 | 1.0139443001 | 1.9921517225
discount_2 | age_60_70 | 0.0051581772 | 14.941256881 | 0.9988837078 | 1.7448706692
discount_3 | age_60_70 | 0.0050914795 | 15.3050000267 | 0.9859676655 | 1.7873493409
discount_4 | age_60_70 | 0.0049347406 | 15.6056052527 | 0.9556150907 | 1.8224546367
info_1 | age_60_70 | 0.0071585648 | 10.6785748186 | 1.327229574 | 1.0816959548
info_2 | age_60_70 | 0.0078874539 | 10.2264372657 | 1.4623688291 | 1.0358962698
bogo_1 | age_70_80 | 0.0056359942 | 14.8455037281 | 1.1190431612 | 1.4072546137
bogo_2 | age_70_80 | 0.0058550184 | 13.812027628 | 1.162531054 | 1.3092879811
bogo_3 | age_70_80 | 0.0049741135 | 13.8745196461 | 0.9876248061 | 1.3152118071
bogo_4 | age_70_80 | 0.0052890713 | 15.6152062381 | 1.0501605955 | 1.4802172715
discount_1 | age_70_80 | 0.0053223945 | 15.8174462451 | 1.0472746564 | 1.7234932633
discount_2 | age_70_80 | 0.0054829322 | 15.5775916821 | 1.0788632694 | 1.6973583413
discount_3 | age_70_80 | 0.0047669886 | 16.4889128856 | 0.9379887723 | 1.7966573009
discount_4 | age_70_80 | 0.0050545853 | 14.2347317144 | 0.9945784949 | 1.5510382545
info_1 | age_70_80 | 0.0072916667 | 10.764780019 | 1.4974252361 | 0.9347025895
info_2 | age_70_80 | 0.0076264881 | 10.1346922542 | 1.5661845582 | 0.8799922597
bogo_1 | age_80_90 | 0.0056879524 | 15.8962491117 | 1.1005091149 | 1.3962055
bogo_2 | age_80_90 | 0.0057123252 | 12.6435314059 | 1.1052247748 | 1.1105115405
bogo_3 | age_80_90 | 0.0048579444 | 12.9237639666 | 0.9399185636 | 1.1351250351
bogo_4 | age_80_90 | 0.0057742244 | 13.3393584754 | 1.1172010766 | 1.1716276928
discount_1 | age_80_90 | 0.0049393342 | 15.7196475308 | 0.9712339076 | 1.4656028096
discount_2 | age_80_90 | 0.0054184982 | 13.8468417375 | 1.0654531393 | 1.2909939688
discount_3 | age_80_90 | 0.0049681818 | 18.6531278965 | 0.9769062792 | 1.7391023939
discount_4 | age_80_90 | 0.0051050518 | 14.2952385749 | 1.0038193711 | 1.3327997195
info_1 | age_80_90 | 0.0081745355 | 10.9408843247 | 1.524496981 | 1.0470917545
info_2 | age_80_90 | 0.0084494561 | 9.6788028053 | 1.5757678608 | 0.9263048863
bogo_1 | age_90_100 | 0.0052522308 | 13.130171408 | 0.9272070361 | 1.4014578501
bogo_2 | age_90_100 | 0.0066364087 | 19.7104335317 | 1.1715640739 | 2.1038066407
bogo_3 | age_90_100 | 0.0052889992 | 10.6647735278 | 0.9336979767 | 1.1383119165
bogo_4 | age_90_100 | 0.005518617 | 13.638327592 | 0.9742337614 | 1.4556962489
discount_1 | age_90_100 | 0.0046351969 | 23.2613711071 | 0.7848919203 | 2.469543918
discount_2 | age_90_100 | 0.0050634398 | 18.1308932749 | 0.8574075941 | 1.9248666387
discount_3 | age_90_100 | 0.0050836895 | 16.0913018886 | 0.8608365217 | 1.7083333794
discount_4 | age_90_100 | 0.0047284327 | 24.2506530197 | 0.8006798197 | 2.5745710516
info_1 | age_90_100 | 0.0068084839 | 9.4132028112 | 1.2802561668 | 1.0399486625
info_2 | age_90_100 | 0.0079113924 | 8.3980731364 | 1.4876452688 | 0.9277995068
bogo_1 | age_100_110 |  |  |  | 
bogo_2 | age_100_110 |  |  |  | 
bogo_3 | age_100_110 | 0.003968254 | 14.63 | 0.2857142857 | 1.5731182796
bogo_4 | age_100_110 | 0.0027777778 | 29.47 | 0.2 | 3.1688172043
discount_1 | age_100_110 | 0.0034722222 | 23.7125 | 0.4299889746 | 1.8909489633
discount_2 | age_100_110 | 0.0099206349 | 27.61 | 1.2285399275 | 2.201754386
discount_3 | age_100_110 | 0.0052083333 | 15.8944375 | 0.644983462 | 1.2674990032
discount_4 | age_100_110 | 0.0046296296 | 16.8173333333 | 0.5733186329 | 1.3410951621
info_1 | age_100_110 | 0.0130208333 | 13.373 | 1.6124586549 | 1.1585878276
info_2 | age_100_110 | 0.0034722222 | 21.935 | 0.4299889746 | 1.9003682045
bogo_no_offer | gender_F | 0.0053532118 | 10.099965659 | 1 | 1
bogo_no_offer | gender_M | 0.0056547614 | 7.4150608793 | 1 | 1
bogo_no_offer | gender_O | 0.0055869197 | 8.5834481589 | 1 | 1
bogo_no_offer | income_110_120 | 0.0041353534 | 15.17464 | 1 | 1
bogo_no_offer | income_20_30 | 0.0055254785 | 2.9672039474 | 1 | 1
bogo_no_offer | income_30_40 | 0.0065878875 | 3.6140522682 | 1 | 1
bogo_no_offer | income_40_50 | 0.0062079546 | 4.4954626816 | 1 | 1
bogo_no_offer | income_50_60 | 0.0058835579 | 6.3946392986 | 1 | 1
bogo_no_offer | income_60_70 | 0.0058204896 | 8.2215708068 | 1 | 1
bogo_no_offer | income_70_80 | 0.0051227492 | 10.4393807149 | 1 | 1
bogo_no_offer | income_80_90 | 0.0043935853 | 13.5645764199 | 1 | 1
bogo_no_offer | income_90_100 | 0.004454719 | 13.4426903802 | 1 | 1
bogo_no_offer | income_100_110 | 0.0042696881 | 17.2978232436 | 1 | 1
bogo_no_offer | joined_2013 | 0.0069679572 | 6.7370230655 | 1 | 1
bogo_no_offer | joined_2014 | 0.007132954 | 5.9337518657 | 1 | 1
bogo_no_offer | joined_2015 | 0.0061184264 | 9.1896836657 | 1 | 1
bogo_no_offer | joined_2016 | 0.0058580816 | 10.0887606827 | 1 | 1
bogo_no_offer | joined_2017 | 0.0053232405 | 8.7348798001 | 1 | 1
bogo_no_offer | joined_2018 | 0.0049169321 | 7.28554282 | 1 | 1
bogo_no_offer | age_10_20 | 0.0062452716 | 5.0038333333 | 1 | 1
bogo_no_offer | age_20_30 | 0.0062094707 | 4.6088825216 | 1 | 1
bogo_no_offer | age_30_40 | 0.0062916968 | 6.0367743764 | 1 | 1
bogo_no_offer | age_40_50 | 0.0054468166 | 8.2606892269 | 1 | 1
bogo_no_offer | age_50_60 | 0.0054322691 | 9.3154012097 | 1 | 1
bogo_no_offer | age_60_70 | 0.0052600251 | 9.4831178019 | 1 | 1
bogo_no_offer | age_70_80 | 0.0050364404 | 10.5492663397 | 1 | 1
bogo_no_offer | age_80_90 | 0.0051684737 | 11.3853219396 | 1 | 1
bogo_no_offer | age_90_100 | 0.0056645717 | 9.3689377866 | 1 | 1
bogo_no_offer | age_100_110 | 0.0138888889 | 9.3 | 1 | 1
discount_no_offer | gender_F | 0.0053022593 | 9.8537936476 | 1 | 1
discount_no_offer | gender_M | 0.0056433021 | 7.10372923 | 1 | 1
discount_no_offer | gender_O | 0.0053777232 | 8.231751462 | 1 | 1
discount_no_offer | income_110_120 | 0.0041564242 | 14.8541391941 | 1 | 1
discount_no_offer | income_20_30 | 0.0057065607 | 3.0134166667 | 1 | 1
discount_no_offer | income_30_40 | 0.006598452 | 3.5198570554 | 1 | 1
discount_no_offer | income_40_50 | 0.0062149464 | 4.3283999721 | 1 | 1
discount_no_offer | income_50_60 | 0.0059330579 | 6.1209598024 | 1 | 1
discount_no_offer | income_60_70 | 0.0056918584 | 7.8006529238 | 1 | 1
discount_no_offer | income_70_80 | 0.0050518305 | 10.4426210262 | 1 | 1
discount_no_offer | income_80_90 | 0.0043100925 | 12.8516208409 | 1 | 1
discount_no_offer | income_90_100 | 0.0043657264 | 13.8917103874 | 1 | 1
discount_no_offer | income_100_110 | 0.0042950289 | 14.7638937409 | 1 | 1
discount_no_offer | joined_2013 | 0.006779194 | 6.8387559809 | 1 | 1
discount_no_offer | joined_2014 | 0.0070468377 | 5.6768288418 | 1 | 1
discount_no_offer | joined_2015 | 0.0061579273 | 9.1333594612 | 1 | 1
discount_no_offer | joined_2016 | 0.0058305806 | 10.0605828766 | 1 | 1
discount_no_offer | joined_2017 | 0.0052881211 | 8.4997637575 | 1 | 1
discount_no_offer | joined_2018 | 0.0048774897 | 6.6326203637 | 1 | 1
discount_no_offer | age_10_20 | 0.0062186098 | 5.3519973684 | 1 | 1
discount_no_offer | age_20_30 | 0.0061512015 | 4.5136405852 | 1 | 1
discount_no_offer | age_30_40 | 0.006375603 | 6.4238956978 | 1 | 1
discount_no_offer | age_40_50 | 0.0053849609 | 8.5238243685 | 1 | 1
discount_no_offer | age_50_60 | 0.005378664 | 8.6366455641 | 1 | 1
discount_no_offer | age_60_70 | 0.0051639417 | 8.5629595044 | 1 | 1
discount_no_offer | age_70_80 | 0.0050821382 | 9.1775503751 | 1 | 1
discount_no_offer | age_80_90 | 0.0050856279 | 10.7257214765 | 1 | 1
discount_no_offer | age_90_100 | 0.0059055225 | 9.4192984127 | 1 | 1
discount_no_offer | age_100_110 | 0.0080751425 | 12.54 | 1 | 1
info_no_offer | gender_F | 0.0053535064 | 10.8414903292 | 1 | 1
info_no_offer | gender_M | 0.0056275075 | 7.7321356208 | 1 | 1
info_no_offer | gender_O | 0.0050562149 | 7.9556971545 | 1 | 1
info_no_offer | income_110_120 | 0.0041328502 | 15.3376599016 | 1 | 1
info_no_offer | income_20_30 | 0.0054483271 | 2.7022530864 | 1 | 1
info_no_offer | income_30_40 | 0.0061533875 | 3.5818458849 | 1 | 1
info_no_offer | income_40_50 | 0.0061588197 | 4.9631707837 | 1 | 1
info_no_offer | income_50_60 | 0.0060946216 | 6.5451831698 | 1 | 1
info_no_offer | income_60_70 | 0.005879871 | 8.5511449464 | 1 | 1
info_no_offer | income_70_80 | 0.0049862235 | 11.0616287226 | 1 | 1
info_no_offer | income_80_90 | 0.0045486445 | 13.8030614583 | 1 | 1
info_no_offer | income_90_100 | 0.0043926346 | 15.6071360878 | 1 | 1
info_no_offer | income_100_110 | 0.004268083 | 19.2140519878 | 1 | 1
info_no_offer | joined_2013 | 0.0070738082 | 6.6010393082 | 1 | 1
info_no_offer | joined_2014 | 0.0068453939 | 6.375960588 | 1 | 1
info_no_offer | joined_2015 | 0.0063522458 | 9.6474234064 | 1 | 1
info_no_offer | joined_2016 | 0.005877465 | 11.0831814695 | 1 | 1
info_no_offer | joined_2017 | 0.0052149998 | 8.9426511761 | 1 | 1
info_no_offer | joined_2018 | 0.0049133441 | 7.8407108623 | 1 | 1
info_no_offer | age_10_20 | 0.0061956575 | 4.7874021909 | 1 | 1
info_no_offer | age_20_30 | 0.0060600315 | 4.808606147 | 1 | 1
info_no_offer | age_30_40 | 0.006322154 | 6.6582633157 | 1 | 1
info_no_offer | age_40_50 | 0.0054818662 | 8.9704842759 | 1 | 1
info_no_offer | age_50_60 | 0.0053084515 | 9.9440100908 | 1 | 1
info_no_offer | age_60_70 | 0.0053936146 | 9.8720668887 | 1 | 1
info_no_offer | age_70_80 | 0.0048694696 | 11.516797043 | 1 | 1
info_no_offer | age_80_90 | 0.0053621198 | 10.4488305613 | 1 | 1
info_no_offer | age_90_100 | 0.0053180638 | 9.0516033636 | 1 | 1
info_no_offer | age_100_110 | 0.0080751425 | 11.5425 | 1 | 1
