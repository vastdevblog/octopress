---
layout: post
title: "An Unexpected Recommendation"
date: 2012-08-31 15:46
comments: true
categories: [analytics, recommendations, bicycles]
author: Josh Levy, Ph.D.
---

One of the most famous examples of big data analytics in e-commerce is Amazon's personalized recommendations.  During a recent amazon session, I was surprised to see a set of recommendations that were entirely uninteresting to me.   In this blog post I speculate about why the poor recommendations surfaced, and I propose additional features that could have steered the recommender towards products I would be more likely to purchase.
<!-- more -->

Amazon's item-based collaborative filtering algorithms (1) infer the similarity between two products, _X_ and _Y_, from the set of consumers who purchased _X_ and those who bought _Y_.  I recently purchased a handlebar basket to add to my bicycle.  I was expecting to see recommendations for other inexpensive bicycle accessories like bells and water-bottle cages.  Instead all of the recommendations were related to an ambulation aid called a _knee walker_.

The recommendations, shown in the screen-shots below are split into three categories: _Frequently Bought Together_, _Customers Who Bought This Item Also Bought_, and _Related Items_.

![Frequently Bought Together](/images/2012-08-31-bike-basket-fbt.png)

_Frequently Bought Together_ items are presumably products that are bought together during a single shopping session.  It is not surprising to see baskets and knee walkers purchased together.  After all, the basket + knee walker combination allows one who walks with the aid of a knee walker to carry stuff in their basket while using their hands to balance on the walker.  While I think a basket is a terrific product to recommend to purchasers of knee walkers, I think a knee walker is not a compelling product to recommend to purchasers of baskets for the following reasons:

1. There is a temporal ordering problem.  Even among consumers who purchase both a knee walker and basket, I would assume to workflow is to first find a knee walker that satisfies some set of requirements (price, rating, availability, etc.) and then to choose a basket that fits it.   It is hard to imagine a consumer in need of both products choosing to address the basket problem first.
2. There is a price mismatch.   It is easy to imagine a $20 basket being an impulse purchase for a knee-walker buyer, or to imagine a customer who wants both items forgetting to add the basket to their (virtual) cart.  In both of these settings, recommending baskets to buyers of knee walkers is probably quite profitable for Amazon.   Conversely, it is hard to imagine adding a $200 knee walker as an impulse purchase to complement a $20 basket.  The only way I can imagine this working is if the consumer just purchased a knee walker elsewhere because they didn't know knee walkers were available from Amazon.  After seeing the knee walker recommendation on the basket page they might choose to cancel the other order and purchase the combination from Amazon.   I have no access to data that would support or refute this, but my intuition is that cheaper products, even knee walker accessories like pads and leg protectors, would be more profitable recommendations than actual knee walkers. 

![Bought This Bought That](/images/2012-08-31-bike-basket-also.png)

_Customers Who Bought This Item Also Bought_ recommendations are presumably products that are bought by the same customers, although possibly spread across multiple shopping sessions.  Again I object to recommending knee walkers to purchasers of baskets, because of time ordering -- if a customer will eventually own both a basket and a knee walker, they were more likely to have purchased the knee walker and realized they needed a way to carry items, than to have purchased the basket and then realized they had no base upon which to attach it; and because of price -- the knee walker is so much more expensive than the basket.

![Related](/images/2012-08-31-bike-basket-related.png)

_Related Items_ are interesting because these recommendations are clustered by item type.   I like these better because several biking clusters are exposed, but again I'm surprised to walkers dominating the _Most Popular_ category, and to see _Mobility Aids & Equipment_ listed before the various bike accessories.

In conclusion, I was surprised when Amazon recommended that I purchase a knee walker to go with my new basket.  I speculate that item-based collaborative filters that were aware of temporal ordering of purchases and price differences between the items could have created recommendations that were more useful for me and more profitable for Amazon.


(1) Greg Linden, Brent Smith, Jeremy York, Amazon.com Recommendations: Item-to-Item Collaborative Filtering, IEEE Internet Computing, v.7 n.1, p. 76-80, January 2003