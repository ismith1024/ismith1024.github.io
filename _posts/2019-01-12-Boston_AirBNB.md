---
layout: post
title: Entertainment Venues - how they affect your AirBNB
---

![_config.yml]({{ site.baseurl }}/images/config.png)


There's no question that large scale, organized data can change how we approach our business perspetives.  But what's really interesting is how the democratization of open data can help out with small scale operators.

To illustrate this point, I've put together some findings from a set of files compiled from AirBNB listings.  The dataset, available from insideairbnb contains just three major items:
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
 - The Boston Marathon, held on April 16.  I have located this at the finish line in Copley Square; the course starts well out of the city center.
 - The 2018 World Series finals, held on October 23-24 at Fenway
 
I have excluded any events at Gilette Stadium, since this is well outside of the city and is in fact closer to Providence, RI.
 
Figures like number of customers are based on number of reviews.  Using various AirBNB forums as a source, the consensus suggests that around 70~75% of AirBNB customers leave reviews.  If the reviews are approximately even under all circumstances, dividing out htis ratio should give close to the real traffic figures.  But feel free to reinterpret the findings if you don't agree with the assumptions.

Customer numbers are also based on the assumption that reviews are received within one week of the booking date.  By considering reviews over a one-week period, this also removes bias based on days of the week.  Finally, customer numbers are also adjusted for seasonal variation.  As you can see, business varies a lot over the year:

![Seasonal business](https://ismith1024.github.io/images/seasonal_guests.png)

*Fig 1: Seasonal business by month (number of reviews)*
 
### 2.  Customer Numbers vs. Distance to the Event

First, I examined the number of AirBNB customer reviewss in the Boston area within one week after one of the entertainment events, versus the average value over the time of year.  There were noticeable increases.  The effects of the World Series and Boston Marathon were most pronounced, with 51% and 23% increases, respectively, in the number of reviews versus the three-month window of the event.  The U2 concerts had a +8% anomaly, while the Pink concerts had a -9% anomaly.  However, the Pink concert may have been distorted by the effects of the Boston Marathon, held one week later.

I next examined the increase in business as a function of distance from the venue.
![Business by distance](https://ismith1024.github.io/images/guests_by_distance.png)
*Fig 2: Business by distance (number of reviews)*

Intuitively, I expected there to be some variation in business increase, reflecting more guiests closer to the event venue.  However, the data did not support this hypothesis.  

From the perspective of an AirBNB operator, this would be encouraging: AirBNBs in the suburbs stand to benefit as much as those downtown.  For this market, guests attending large events are clearly willing to use transporation to get there.

### 3.  Customer Satisfaction vs. Distance to the Event

Despite being close to an event venue, AirBNB customers were less satisfied than usual within approximately 5 km:

![Customer review anomaly](https://ismith1024.github.io/images/boston_marathon_sentiment_anomaly.png)
*Fig 3: Customer satisfaction anomaly (red scale = negative)*

In the above image (this one corresponding to the Boston Marathon), red points are the locations of reviews, with a red point indicating more nagative than seasonal average.

The trend for all events I examined can be seen below:
![Customer review anomaly, all](https://ismith1024.github.io/images/sent_by_distance.png)
*Fig 4: Customer satisfaction anomaly by distance*

The negative trend flattens out after 5 km.


### 4.  Average Price vs. Distance to the Event
An initial inspection of price as a function of distance to events showed counterintuitively that prices were lower closer to the event venue.  
![Price anomaly, all](https://ismith1024.github.io/images/price_dist_wrong.png)
*Fig 5: Price anomaly by distance*

On closer inspection, it became clear that the results were skewed - condos have a lower list price than houses, and my entertainment events were held in neighborhoods with a higher density of condos!  This demonstrates an important point, that data analysis requires an understanding of the business domain.

Once I corrected for the neighborhood composition, the actual trend stands out.  Here is the example for prices during the World Series:
![Price anomaly, world series](https://ismith1024.github.io/images/price_map.png)
*Fig 6: Price anomaly map, World Series (blue scale = higher price)*


### 5.  Findings
As a takeaway then, we can draw three immediate conclusions:

 - As expected, AirBNBs are busier around the date of a major event.  Customers are using the service to stay when attending.
 - The AirBNBs closer to an event aren't noticeably busier than anywhere else in the city.  Customers appear to be happy to use trnasportation to get to their events
 - AirBNB operators aren't afraid to increase their rates
 - Customers who used AirBNBs close to (within 5 km) an event were less satisfied than average in their AirBNB experience.

From the perspective of an AirBNB operator, the data predicts that business will pick up when there is a large event ocurring in the city where you are located, and that suburban operators are in general not increasing prices to take advantage of this.  There is a business opportunity to increase revenue.

From the perspective of somebody who is travelling to attend one of these events, AirBNBs are an attractive option to stay at.  Be prepared for higher prices, and take note that guests are less happy when staying close to their event.  You might be happier staying a little further away at either a better price.  (Now, if only we had Uber data to check transortation usage...)
