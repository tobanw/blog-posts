---
layout: post
title: "Find your community: increasing reddit user engagement"
date:   2017-06-22 12:00:00
type: post
---

[Reddit](https://www.reddit.com/) is a social media platform where people can browse or join communities called *subreddits* to submit posts as well as vote and comment on submissions.
Any user can create a subreddit — as a result, there are now [over 1 million subreddits](http://redditmetrics.com/history#tab2) spanning almost any topic imaginable.
Subreddits are based around topics such as [r/cats](https://www.reddit.com/r/cats/) or [r/datascience](https://www.reddit.com/r/datascience/).

When someone first visits the reddit homepage, they are shown the reddit *frontpage*, a post feed comprised of the currently most popular content from a curated selection of 50 *default* subreddits.
(Note: this has recently changed to showing popular posts from any subreddit, with some exceptions. However, this only affects new accounts — existing users still see the default frontpage subreddits).
Logged-in users can customize their feed by subscribing to other subreddits; otherwise they see the default frontpage content.

### Data

Data on all reddit comments is [publicly available on Google BigQuery](https://bigquery.cloud.google.com/dataset/fh-bigquery:reddit_comments) thanks to reddit user [fhoffa](https://www.reddit.com/user/fhoffa).
The data is very large and is divided into monthly tables — the table for May 2017 alone contains 80 million rows and takes up 20GB on disk.
As such, it was very important to use BigQuery for performant queries.

To access the data, I used the [Google Cloud Datalab](https://cloud.google.com/datalab/), a Jupyter notebook running on a VM instance which connects to BigQuery.
By using SQL queries to aggegate the data down to a manageable size, I could load the results into pandas dataframes.

### Analysis

I set out to find what behaviors are characteristic of engaged reddit users, and what reddit could do to increase engagement.
As reddit content is entirely user-generated, I decided to focus on commenting behavior as a proxy for adding value to the reddit experience.
Comments are an integral part of reddit — as many a redditor knows, they often produce invaluable discussions of the parent post.
I therefore used the number of monthly comments as a metric for user engagement.

The distinction between the reddit frontpage and other subreddits is the key divide in the reddit experience.
The frontpage surfaces excellent content with broad appeal and is extremely popular.
However, the frontpage default subreddits only produce 17% of all reddit comments, despite their outsized popularity.
Other subreddits vary widely, but many thriving communities are based around a specific interest, such as [r/coffee](https://www.reddit.com/r/Coffee/).

I was interested to see how user behavior differed across this divide.
Based on my personal experience with reddit, I suspected that people get the most out of reddit when they go beyond the frontpage and find subreddits related to their specific interests. 
However, given the popularity of the frontpage, it wasn't obvious that this would be true.

### Results

What I found was that redditors who participate in communities beyond the frontpage are more engaged users.
To measure this, I grouped users by the percentage of comments they made on posts outside of the frontpage default subreddits.
Then I calculated the average number of comments for each group.

The chart below shows that "frontpage-focused" users (0-20% of comments outside of frontpage subreddits) have the lowest engagement with an average of 22 comments per month.
Engagement increases to an average of 27 comments per month for the group having 60-80% of comments outside of frontpage subreddits.

(Note: I excluded the 80-100% group, which is almost entirely composed of users who never comment on frontpage subreddits.
This group is large and behaves very differently.
As my focus is on increasing engagement for frontpage-focused users, I restricted my analysis to users with at least some frontpage engagement.)

To increase my confidence that this relationship is causal, I did some additional checks to rule out alternative explanations.
My main concern was that user tenure drives both effects: users with longer tenure both comment more and participate more broadly in other communities.
To rule this out, I did the same analysis within cohorts, so that everyone would have the same tenure.

I also excluded outliers to ensure that the average wasn't being skewed by a few extremely active users.
(The chart above shows the results for the cohort that made their first comment between November and December of 2016, and is limited to users with 5-150 comments per month.)
The finding withstood these robustness checks — of course, the relationship could be an artifact of some other selection bias, but it increases my confidence in the causal interpretation.

Based on this evidence, I concluded that reddit should do more to help frontpage-focused users discover other communities.
Reddit has a very useful subreddit discovery feature, but it's not very prominent in the user interface.
To find it, you have to click on the small "more" link in the top subreddit navigation bar.

To encourage users to explore other communities, I proposed that this subreddit discovery feature be made available in the frontpage sidebar.
This way, subreddit discovery would be much more evident to frontpage visitors.

### Proposed A/B test

To evaluate the impact of this proposed sidebar feature (and to establish whether my finding is indeed a causal relationship), I designed an A/B test that could be implemented by reddit.
The goal of the experiment is to determine if this feature increases the commenting activity of frontpage-focused users.

#### Target group

The experiment targets frontpage-focused users: registered users who have ever commented, but haven't subscribed to any other subreddits (and hence still see the default frontpage).
I also restrict to users that joined in 2016.
(Though it is typical to target newly registered users, reddit is [already running](https://www.reddit.com/live/x3ckzbsj6myw/updates/bb55d54c-7f79-11e6-bf48-0eeb724eeebd) an A/B test for subreddit discovery during the onboarding flow. As such, my proposed test focuses on existing users.)

#### Duration and metrics

The test would run for 5 weeks in total.
The first week would be an adaptation period for users to try out the feature and subscribe to other subreddits.
Then, the following 4 weeks would track the number of comments for each user.

An important secondary metric to track is the number of subscriptions to other subreddits.
This reveals how effective the feature is at driving users to find other communities.

#### Power analysis

Unlike many companies, reddit is very conservative about changing the user experience and primarily [runs A/B tests](https://www.reddit.com/live/x3ckzbsj6myw/) to learn more about its users.
As such, I tailored the design of the test to reflect this caution.

I chose the test significance $$\alpha = 0.01$$ to be very conservative about false positives.
The power $$1 - \beta = 0.9$$ reflects less concern about false negatives.
This way, a positive result could be considered very trustworthy.

The minimum effect size that would plausibly justify implementing this feature would be around 5%, or 1 extra comment per month on average.
To reduce the number of users exposed to the new feature, I use an unbalanced design with a much larger control group.
To detect a 5% increase in comments, the experiment would require 7,000 users randomized into the A (or treatment) group and at least 20,000 users in the B (or control) group.
I chose a total sample size of 27,000 users because that is roughly the number of frontpage-focused users in the 2016 cohorts.

#### Impact

There are roughly 300,000 actively commenting users who only comment on the frontpage, so a 5% increase in comments would result in roughly 0.5% more total comments across all of reddit.
This effect could also grow over time as these users continue to explore other subreddits. 
Moreover, this is only considering the impact from users who are commenting at all — presumably there are far more users that never comment.

### Conclusion

Reddit's mission is "to help people discover places where they can be their true selves, and empower our community to flourish."
This fits nicely with my recommendation of making community discovery more prominent in the user interface.
