---
layout: post
title: "Super Bowl and Anti-Super Bowl"
date: 2013-02-04 11:08
comments: true
categories: [analytics, traffic, super bowl]
author: Josh Levy, Ph.D.
---

More than 100,000,000 people tuned in to watch Super Bowl XLVII.   In this blog post we will look at how web traffic to a Vast powered automotive marketplace was affected by the Super Bowl and by a power outage that disrupted the Super Bowl for roughly 30 minutes.  

<!-- more -->

On February 5, 2012, the day of Super Bowl XLVI, Vast observed an interesting pattern in our web traffic.  In the early part of the day, traffic was significantly higher than Sunday traffic in January of that year.  As might be expected, as pre-game festivities ramped up, our traffic from traditional computers plummeted.  Traffic from iPhones remained high as car shoppers chose to watch the game and surf on their phones simultaneously.  Traffic from iPads trended near its historical average.  After the game, traffic from traditional computers rose above the historical trend again. 

![2012 Superbowl Traffic](/images/2013-02-04-superbowl-2012.png)

The 2012 traffic pattern can be seen in the graph above.  Historical ranges are drawn as shaded areas with a dashed line indicating average trend.  Super Bowl Sunday traffic is shown with solid lines, and an approximate time window for the Super Bowl is highlighted.

As Super Bowl XLVII approached, we hypothesized that we would see a similar pattern in our traffic on Sunday, February 3, 2013.  Of course, we failed to anticipate the _Anti-Super Bowl_: a half hour disruption of the game, due to a power outage at the Superdome. 

In the 2013 graph, we observe high traffic in the early part of the day that falls off before the game starts.  A substantial amount of traffic returns during the power outage. The steep rise of the graph before the power failure is an artifact of the hourly buckets we used to produce the graph.  Traffic drops again when the game resumes and again rises above the historical average after the game ends.

![2013 Superbowl Traffic](/images/2013-02-04-superbowl-2013.png)

In hindsight, these results are fairly obvious.  The Super Bowl draws the attention of 100,000,000 people.  Those people don't get much practical work done during the game, unless they use devices like iPhones.   The blackout on Sunday was an anomaly that functioned as an Anti-Super Bowl giving millions of people an unexpected opportunity to get stuff done.
