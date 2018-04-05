# Purpose and Background

### School project by Miles Vaughn and Dagatha Delgado. 

We wanted to visualize the attitude of Twitter users towards the leading U.S. Presidential nominees. We expect this dataset to grow and to eventually anticipate political outcomes in states as well as cities. 


# Methods of Collecting Data

We use StreamR to pull from Twitter's API and narrow our initial searches by relevant topic and geolocation. There are several challenges to this method, namely the 1% rule and the geolocation issues. If less than 1% of the population in the area you are mining tweet about your subject then using search terms is moot since it is extremely likely that you will only recieve irrelevant tweets. Additionally, geolocation data is rarely turned on and Twitter will attempt to link your location to keywords in your profile. This can lead to tweets being assigned coordinates hundreds if not thousands of miles away from their actual location. Additionally, StreamR also picks up on retweets and links to other tweets that are within your area of interest but are being re-communicated by people around the globe. 

Since it is the norm to pull thousands and thousands of tweets for only a few usuable pieces of information, we decided to broaden our initial search. By casting a wide net we are able to eventually pick up on smaller trends. 

# Organizing the Data

With our search terms each tweet or retweet we recieve initially has 43 variables. Since many of the terms are just taking up space and cluttering our dataset, it is better to just get rid of them as soon as possible. In an effort to maintain a high level of relevancy, we immediately discard tweets that do not contain the stem of the candidate's name (keep the stem so we can include tweets using wordplay) and as a result we have a far more reliable dataset. Since we are plotting our data, we get rid of tweets that do not have place_lat/place_lon or lat/lon values. Ideally we would keep only the lat/lon values since it is likely the most reliable location of the sender but doing so would leave us with a fraction of a percent of useable tweets. Ultimately, the most important information for our project is text, tweet, screen name, location, and the semantic analysis. 

# Sentiment Analysis

At the heart of this project, standing alongside geolocation, was our interest in text sentiment analysis. In our project's development phase, we toyed with the idea of creating a detailed and comprehensive list of keywords that would be used to reliably measure and quantify our text's sentiment towards their presidential candidates, even taking into account sarcasm, nuance, and frequent mispelings. Thank God that [Bing Liu](https://en.wikipedia.org/wiki/Bing_Liu), a computer science professor at the University of Illinois at Chicago, along with his research assistant Minqing Hu, had already created one! Anyone interested in sentiment analysis or NLP should look into the [fascinating research](https://www.cs.uic.edu/~liub/FBS/SentimentAnalysis-and-OpinionMining.pdf) he has conducted along [with his colleagues](http://www.idiap.ch/~apbelis/hlt-course/positive-words.txt). 

Our sentiment score function is similar to the one designed by [Jeffrey Breen](https://jeffreybreen.wordpress.com/tag/sentiment-analysis/) but with some additions as to include some key words used in the 2016 elections including 'Drumf' and 'Shillary.' The Twitter API also has a sort of 'built in' sentiment analysis where you  can filter for tweets that are happy, sad, or neutral. The developers over at PubNub have made a [great map](http://pubnub.github.io/tweet-emotion/) that takes a feed of tweets, determines if the sentiment is positive or negative, and then uses that information to determine the general happiness of the state. 

# Data Visualization

Our data visualization focuses around our three core values: the candidates, the sentiment score of the tweet, and the location of the tweet. 

We have provided two ways to visualize this data. First is a bar graph based on total positive sentiment; a simple way to compare all of the candidates together. Second is the location based sentiment analysis (scatter plot). This can get very messy when comparing all candidates so it may be best to examine each candidate separately.  


# Future Research



###### License

If you distribute this project in part or in full, please attribute with a link to the GitHub page. This software is available under The MIT License, reproduced below. Remember to delete collected tweets within the time frames set by the Twitter ToS. Follow the Twitter ToS.

Copyright (c) 2016 Miles Vaughn

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
