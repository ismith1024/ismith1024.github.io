---
layout: post
title: Entertainment Venues, and how they affect Your AirBNB
---



![_config.yml]({{ site.baseurl }}/images/config.png)




There's no question that large scale, organizeddata can change how we approach our business perspetives.  But what's really interesting
is how the democratization of open data can help out with small scale operators.

To illustrate this point, I've put together some findings from a set of files compiled from AirBNB listings.  The dataset, available from <here>
contains just three major items:
 - A collection of the listings, including location and building type
 - A collection of customer reviews
 - The calendar, including listing, date, and price
 
The files are approximately a million or so rows long and go back several years.  I'm going to focus on the 2018 timeframe.
 
I've used the listings applicable to Boston, MA to see what business trends emerge when a major entertainment event is nearby, and maybe challenge some conventional wisdom:
 - How does proximity to a large event impact number of customers?
 - How does proximity to a large event impact prices being asked by AirBNB operators?
 - How happy are AirBNB customers who stayed close to the event?
 
### 1.  About the data

Doing a little research, I have decided to focus on well-attended, single, events in the Boston area.  These were:
 - Two large concerts held at TD Gardens (based on attendance figures, they were sold-out back-to-back shows) : U2 on June 21, and Pink, on April 8
 - The Boston Marathon, held on April 16.  I have located this at the finish line in C Square; the course starts well out of the city center.
 - The 2018 World Series finals, held on October 23-24 at Fenway
 
I have excluded any events at Gilette Stadium, since this is well outside of the city and is in fact closer to Providence, RI.
 
Figures like number of customers are based on number of reviews.  Using "Some Guy on the Internet" as a source, the forums suggest that around 70~75% of AirBNB customers leave reviews.  If the reviews are approximately even under all circumstances, dividing out htis ratio should give close to the real traffic figures.  But feel free to reinterpret the fingins if you don't agree with the assumptions.

Customer numbers are also based on the assumption that reviews are received within one week of the booking date.  By considering reviews over a one-week period, this also removes bias based on days of the week.  Finally, customer numbers are also adjusted for seasonal variation.  As you can see, business varies a lot over the year:

![Seasonal business](https://ismith1024.github.io/images/seasonal_business.png)
*Fig xx: Seasonal business by month (number of reviews)*
 
### 2.  Customer Numbers vs. Distance to the Event

First, I examined the number of AirBNB customer reviewss in the Boston area within one week after one of the entertainment events, versus the average value over the time of year.  There were noticeable increases.  The effects of the World Series and Boston Marathon were most pronounced, with 51% and 23% increases, respectively, in the number of reviews versus the three-month window of the event.  The U2 concerts had a +8% anomaly, while the Pink concerts had a -9% anomaly.  However, the Pink concert may have been distorted by the effects of the Boston Marathon, held one week later.

I next examined the increase in business as a function of distance from the venue.
![Business by distance](https://ismith1024.github.io/images/business_by_distance.png)
*Fig xx: Business by distance (number of reviews)*


### 3.  Customer Satisfaction vs. Distance to the Event

Despite being close to an event venue, AirBNB customers were less satisfied than usual within a 5k or so radius:

![Customer review anomaly](https://ismith1024.github.io/images/boston_marathon_sentiment_anomaly.png)
*Fig xx: Customer satisfaction anomaly (red scale = negative)*

In the attached image, red points are the locations of reviews, with a red point indicating more nagative than seasonal average.

### 4.  Average Price vs. Distance to the Event
This was the most surprising - when averaged across all of Boston, prices did not change much during the week of an entertainment event.  However, this change did vary with the distance to the event:

<Fig>

AirBNB operators seem to be discounting prices when an event is nearby.  I don't know why this is.


### 5.  Findings
As a takeaway then, we can draw three immediate conclusions:

 - As expected, AirBNBs are busier around the date of a major event.  Customers are using the service to stay when attending.
 - The AirBNBs closer to an event aren't noticeably busier than anywhere else in the city.  Customers appear to be happy to use trnasportation to get to their events
 - Customers who used AirBNBs close to (within 5k) an event were less satisfied than average in their AirBNB experience.
 - AirBNB operators seem to be discounting their rates.
<!--- % Next you can update your site name, avatar and other options using the _config.yml file in the root of your repository (shown below).

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.% -->