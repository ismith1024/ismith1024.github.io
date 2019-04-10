---
layout: post
title: Starbucks Capstone Challenge
---

![_config.yml]({{ site.baseurl }}/images/SBUX_placeholder.png)

## Project Overview

The project represents a simulation of the bahavior users of Starbucks' mobile app over a period of time in which users are sent promotional offers.  The promotional offers fall into one of three types: buy-one-get-one-free ('BOGO'), informational, and discount.  The parameters of each offer vary (some are valid for longer than others, some provide a greater reward), and the offers are made by way of different marketing channels – some combination of web, mobile, social, and email.  However, there are only ten variations of these offers in total.

The possible objectives of the exercise is quite open-ended.  My implementation is based on a high-level observation of the data and business doman (marketing):

- Although two events occur in sequence, it is not possible to claim that they are dependent on each other (no causality).  A customer might but the same latte every mornging, regardless of whether he or she was given a BOGO offer by email
- The outcome of an offer is not repeatable (no determinism).  A customer might get the same discount on two separate days, but just not feel like a coffee on one of those days.
- For this reason, there is no cause-and-effect relationship (causal determinism).  This point might be lost on someone approaching the data science domain from a software background, for example, where the same SELECT clause had better give the same result each time.

My implmentation of this project is oriented towards describing and predicting human behavior on an aggregate level.  The project can be summarized by one key question:
- Given some customer demographic,
- And given some offer parameters,
- What is the change of that demographic's number of transactions (over the offer period, relative to no offer)?
- And what is the change in the average transaction value?

I will be approaching this exercise as though I were consulting for a marketing team, and providing quantifiable information which can be used by those with financial information (product pricing, for example) to render a business decision.

I have found that both changes can be described fairly well, and predicted to some degree.  My proposed strategy is to anaylze the data two ways – descriptive statistics, to give the imaginary marketing team a set of heuristic rules for targeting customers, and a regression learner, which can predict a user's response to the propsed campaign.  The performance metrics – MSE and R^2 are used to find the best learner.

## Analysis

The data set is deceptively simple.  It contains three JSON files with a minimal number of fields.

#### Profile
"Profile" contains the information on the simulated users.  There are 17000 users in total.  A user is defined by four demographics: age, gender, income, and membership date.  Data is not necessarily complete...

![placeholder_1](https://ismith1024.github.io/images/age_raw.png)

*Fig x: Raw user age data*

... unknown age is encoded as 118 years old

 Male  | Female  | Other  | No Response
---|---|---|---
 8484  | 6129  | 212  | 2175

*Fig x: Raw gender data*


... gender contains null values, in addition any user who does not identify as male or female is tagged as "other".

![placeholder_1](https://ismith1024.github.io/images/age_raw.png)

*Fig x: Raw user age data*

... income contains null values.

I evaluated these missing data points both by way of imputing missing values, and by removing missing data from the experiment.  I found the machine learning performance was better with the removed data, but not by much.  And for the purposes of statistical significance, with a sample size of 17000, removing even thousands of records did not move the needle.

#### Portfolio

Portfolio is a straightforward record of all ten offers, contianing a hex id string, list of channels, difficulty, duration, reward, and offer type (one of 'bogo', 'informational', or 'discount')

#### Transcript

Transcript contains the time series data for all events - offer received, offer viewed, transaction, and offer completed.  The JSON objects did not lend themselves to creating a dataframe directly as not every event had the same characteristics. I built a series of functions to inspect a JSON object and place the relevant characteristics in a suitable column (for exampl, 'transaction amount').  Columns with no applicable value were encoded as -1.

## Methodology

## Preprocessing

A significant amount of data-wrangling work was necessary to transform the transcript time-series data into a table of the form | User Demographics | Offer Parameters | Change in Transaction Rate | Change in Average Transaction Value|. This required me to determine what offers each user had received, what is their duration, and what intervals of the experiment did not fall within any offer period. Once the offer periods and no-offer periods were determined, transactions and their values were used to evaluate trnasaction rates and values by offer or no-offer interval.

The profile data is missing significant sections of demographic data, and I have evaluated the impact of imputing missing values vs. omitting incomplete records.  The regression learner performed better when missing values were omitted.

I parsed the date string into a datetime object to enable processing.

As described previously, the Transcript file required a series of functions to be applied to the rows in oirder ot manipulate its values into a table with the following structure: 

As it turned out, the offers were all received within one of seven time slots:

![placeholder_1](https://ismith1024.github.io/images/offer_periods.png)

*Fig x: Offers Recevied Time Slots*

#### Transaction Record Preprocessing

A central part of the analysis is the concept of an offer interval.  For each user, I want to know: 
 - When an offer is made, how many transactions take place over the duration of the offer? 
 - What is the value of these offers?
 - How often does the user make transactions when no offer is in efffect?
 - What is the value of these offers?

By segmenting the offers, I was able to establish an A and B case for each user, and by aggregating, for each demographic segment.  

## Implementation

I initally evaluated Bayesian statistics, specifically Markov Chain Monte Carlo (MCMC), to determine the effect of a single offer event on a time series of transactions by a user.  This is descibed in the excellent e-book, " Bayesian Methods for Hackers" by Cameron Davidson-Pilon[1].  In effect, given a noisy time series, the MCMC detects if the time series transitions from one steady state to another, and provides an estimate for when that event occurred.

There were some early results that looked promising...

![placeholder_1](https://ismith1024.github.io/images/SBUX_placeholder.png)

*Fig x: Placeholder Image*

... but, on reflection I found that this application was not quire suited for a couple of reasons.
1.  The MCMC assumes a prior distribution, posterior distribution, and transition between two steady states.  Looking at the transactions over time (the spikes are offers), steady-state is not a good characterition fo the time series.  If anything, the time series should be mathematically modelled as an impulse response with corresponding exponential decay.

![placeholder_1](https://ismith1024.github.io/images/transaction_times.png)

*Fig x: Transcript events*

2.  MCMC is intended to detect the event.  For the starbucks problem, the time of the event is known with certainty.

3.  MCMC perfoms best at digging a signal out of a noisy environemnt.  The aggregated Starbucks transaction time series is quite clean.

4. The MCMC assumes two distinct periods of time, an internal A and B.  I have an internal A and B, but A is not in a contiguous time block.  This is discussed in the preprocessing step  on the no-ffer intervals.

-------

I segmented each user demographic into groups which would make sense to the Starbucks marketing team (for example, "users aged 20-30", users with income 50k-60k", "Female users"). For each user demographic, I generated a histogram showing the distribution of change in transaction rate and value, as well as recording their mean values in a master table.

These mean values, as well as demographic data and offer parameters, we used to train a Linear Regression learner.

## Statistics

I segemnted each demographic, into logical bins, and then determined the transaction rate and value for each offer.  Next, I determined the transaction rate and average vlaue for periods where no offer was valid for the same set of customers.  These two interval periods became my A and B groups.  I plottted a series of historgrams, and colected the A/B ratios in a matrix.

![placeholder_1](https://ismith1024.github.io/images/bogo_all_users.png)

`BOGO 1 mean: 0.006450805435910441`
`BOGO 2 mean: 0.006798231448249598`
`BOGO 3 mean: 0.005380281062973571`
`BOGO 4 mean: 0.006058094776529127`
`No offer mean: 0.005528949447740996`

*Fig x: Example histogram - BOGO response, Transaction Rate, All Users*

## Refinement 

I implemented the regression learner incrementally, watching for performance after each change.
I built a dataframe contianing | User demographics | Offer Parameters | Average Value Change | Transaction Rate Change |

First, I ran the raw data through a linear regressor, mostly to establish a baseline and determine the correct workflow and syntax.  Next, I scaled and normalized the data for each user demographic.  The results were, predicatbly, a great imrpovement.

I evaluated both RandomForestRegressor and AdaboostRegressor learners, but the LinearRegression model perofrmend the best.

After establishing the regression algorithm, I added offer reward, offer difficulty, and then offer channels to the dataframe.  None of these columns had a material effect on the performance of the regressoon model, which suggests to me that the inofrmation contained in these data fields does not contribute information to the system.

Finally, I observed that membership year has a non-linear correlation to average transaction value.  Users who joined in 2015 and 2016 spent noticeably more per visit than users who jioned in 2013 and 2018.  I thought of using a non-linear statistical model, but instead one-hot encoded membership year and re-ran the regressor.  The result was in improvement in average transaction vlaue with a trade-off in perfomance of trnasaction rate.  If Starbucks were interested in using htiese regresors, they may wish to consider encoding membership year separately for the separate y-values.




## Results

## Model evaluation and Validation



## Justification

## Conclusion

### Opportunities for improvement
Starbucks could improve the data set in  the following ways:
- Run the experiment with a continuous variation of reward and difficulty.  I found that the transcript set was not informative with respect to these parameters – there were only two values for the BOGO and discount offers, this made finding a relationship between difficulty and income, for example, unreliable
- Run the experiment with separable information.  Channel information was specifically bad.  For example, if all four offers of a type are avalable by web, it is not possible to measure the effect of web vs no web.  Similarly, if two offers are made by social with difficulty X, and two others not by social with dicciculry Y, it is not possible to attribute any difference to the channel, or the difficulty.
I could improve the experiment by:
- Considering the offer received.  I dismissed this value early on, but given more time to complete the exercise, the next thing I would have added would be a binary attribute "offer received" to the learner.


References
[1] Davidson-Pilon, Cameron "Bayesian Methods for Hackers", 2019, online, available: https://github.com/CamDavidsonPilon/Probabilistic-Programming-and-Bayesian-Methods-for-Hackers