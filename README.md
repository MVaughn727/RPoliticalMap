# RPoliticalMap
Using R to map tweet sentiment of politicians by region
#load packages

library(twitteR)
library(ROAuth)
require(RCurl)
library(stringr)
library(tm)
library(ggmap)
library(dplyr)
library(plyr)
library(tm)
library(streamR)
library(wordcloud)  

api_key = "xxx" 
api_secret = "xxxx" 
access_token = "xxxx" 
access_token_secret = "xxxxx" 

setup_twitter_oauth(api_key,api_secret,access_token, access_token_secret)

#Hillary and Trump tweets
Tweets <- searchTwitter("#Trump OR #Trump2016 OR #DonaldTrump OR #Hillary OR #Hillary2016 OR #HillaryClinton", n=500, lang="en")

Text <- sapply(Tweets, function(x) x$getText())
Corpus <- Corpus(VectorSource(Text)) #what to use for csv files
Corpus <- tm_map(Corpus, removePunctuation)
Corpus <- tm_map(Corpus, tolower)
Corpus <- tm_map(Corpus, function(x)removeWords(x,stopwords()))
Corpus <- Corpus(VectorSource(Text))
PTD <- tm_map(Corpus, PlainTextDocument)

wordcloud(PTD, scale=c(4,.5),min.freq=3,max.words=Inf,
          random.order=TRUE, random.color=FALSE, rot.per=.1,
          colors="black",ordered.colors=FALSE,use.r.layout=FALSE,
          fixed.asp=TRUE)
          
#4. Develop & assess the models.
#https://www.cs.uic.edu/~liub/FBS/sentiment-analysis.html
#Sentiment Analysis- Negative and Positive Words List

Pos = scan('filelocation/Big Data/twitter/positive-words.txt', what='character', comment.char=';')
Neg = scan('filelocation/Big Data/twitter/negative-words.txt', what='character', comment.char=';')

NegWords = c(Neg, 'wtf', 'drumpf', 'shillary')

#Sentiment Test

#sample = "Trump says there will be a recession if he's not elected. But there will be a Depression if he is!"
#result = score.sentiment (sample, Pos, NegWords)

#https://jeffreybreen.wordpress.com/tag/sentiment-analysis/


  score.sentiment = function(sentences, Pos, NegWords, .progress='none')
  {
    require(plyr)
    require(stringr)
    
    # we got a vector of sentences. plyr will handle a list
    # or a vector as an "l" for us
    # we want a simple array ("a") of scores back, so we use 
    # "l" + "a" + "ply" = "laply":
    scores = laply(sentences, function(sentence, Pos, NegWords) {
      
      # clean up sentences with R's regex-driven global substitute, gsub():
      sentence = gsub('[[:punct:]]', '', sentence)
      sentence = gsub('[[:cntrl:]]', '', sentence)
      sentence = gsub('\\d+', '', sentence)
      # and convert to lower case:
      sentence = tolower(sentence)
      
      # split into words. str_split is in the stringr package
      word.list = str_split(sentence, '\\s+')
      # sometimes a list() is one level of hierarchy too much
      words = unlist(word.list)
      
      # compare our words to the dictionaries of positive & negative terms
      pos.matches = match(words, Pos)
      neg.matches = match(words, NegWords)
      
      # match() returns the position of the matched term or NA
      # we just want a TRUE/FALSE:
      pos.matches = !is.na(pos.matches)
      neg.matches = !is.na(neg.matches)
      
      # and conveniently enough, TRUE/FALSE will be treated as 1/0 by sum():
      score = sum(pos.matches) - sum(neg.matches)
      
      return(score)
    }, Pos, NegWords, .progress=.progress )
    
    scores.df = data.frame(score=scores, text=sentences)
    return(scores.df)
  }

# taking sentiment of the tweets
result = score.sentiment(WI_Trmp, Pos, NegWords)
result$score

#finding total
Tweet.Scores = score.sentiment(Text, Pos, NegWords, .progress='text')


#5. Evaluate finding from the models.

#6. Deploy results.


#location searches

N=200  # tweets to request from each query
S=125  # radius in miles

lats=c(43.052222, 43.066667, 44.513333, 42.582222, 42.726111, 44.266667, 43.011667, 44.024167, 44.816667, 42.683889) 
lons=c(-87.955833, -89.4,-88.015833, -87.845556, -87.805833, -88.4, -88.231667, -88.561111, -91.5, -89.016389)

#cities=(Milwaukee, Madison, Green Bay, Kenosha, Racine, Appleton, Waukesha, Oshkosh, Eau Claire, Janesville)

WI_Trump=do.call(rbind,lapply(1:length(lats), function(i) searchTwitter("#Trump OR #Trump2016 OR #DonaldTrump", since= "2016-03-20", lang="en",n=N,resultType="recent", geocode=paste(lats[i],lons[i],paste0(S,"mi"),sep=","))))

#Latitude and longitude of each tweet, the tweet itself, how many 
#times it was re-twitted and favorited, the date and time it was twitted, etc.

WI_Trumplat=sapply(WI_Trump, function(x) as.numeric(x$getLatitude()))
WI_Trumplat=sapply(WI_Trumplat, function(z) ifelse(length(z)==0,NA,z))  

WI_Trumplon=sapply(WI_Trump, function(x) as.numeric(x$getLongitude()))
WI_Trumplon=sapply(WI_Trumplon, function(z) ifelse(length(z)==0,NA,z))  

WI_Trumpdate=lapply(WI_Trump, function(x) x$getCreated())
WI_Trumpdate=sapply(WI_Trumpdate,function(x) strftime(x, format="%Y-%m-%d %H:%M:%S",tz = "UTC"))

WI_Trumptext=sapply(WI_Trump, function(x) x$getText())
WI_Trumptext=unlist(WI_Trumptext)

isretweet=sapply(WI_Trump, function(x) x$getIsRetweet())
retweeted=sapply(WI_Trump, function(x) x$getRetweeted())
retweetcount=sapply(WI_Trump, function(x) x$getRetweetCount())

favoritecount=sapply(WI_Trump, function(x) x$getFavoriteCount())
favorited=sapply(WI_Trump, function(x) x$getFavorited())

data=as.data.frame(cbind(tweet=WI_Trumptext,date=WI_Trumpdate,lat=WI_Trumplat,lon=WI_Trumplon,
                         isretweet=isretweet,retweeted=retweeted, retweetcount=retweetcount,
                         favoritecount=favoritecount,favorited=favorited))

#Exclude tweets that do not have lat/lon on

data=filter(data, !is.na(lat),!is.na(lon))
lonlat=select(data,lon,lat)

#Get full address of each tweet

result <- do.call(rbind, lapply(1:nrow(lonlat),function(i) revgeocode(as.numeric(lonlat[i,1:2]))))

Tweet.Scores = score.sentiment(WI_Trump, Pos,
                               NegWords, .progress='text')
