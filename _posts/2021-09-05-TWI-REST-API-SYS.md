---
layout: post
title: Twitter Face Filter Bot REST API System
---

### Twitter API and Tweet Retrieval

The REST API system was built using Spring Boot in Java. The basic tweet retrieval code makes use of the Twitter API through the use of the Java library Twitter4J. Once configuring and authenticating with the Twitter API is done, a search is completed on Twitter using the Query class in Twitter4J as follows.

```
try {
  // Create query object, perform query
  Query q = new Query("@FaceFilterBot");
  QueryResult r = null;
  r = t.search(q);

  // Collect tweets into an ArrayList called statuses
  for (Status status : r.getTweets()) {
    statuses.add(status);
  }
} catch (TwitterException e) {
  e.printStackTrace();
}

```

Once all the tweets found are added into an ArrayList any media components within the tweets are able to be extracted using this getMediaURL() function I wrote.

```
public ArrayList<String> getMediaURL(Status s) {
  ArrayList<String> media = new ArrayList<>();
  String mediaURL;

  if (s.getMediaEntities().length != 0) {
    for (int i = 0; i < s.getMediaEntities().length; i++){
      if (!Arrays.asList(s.getMediaEntities()[i].getVideoVariants()).isEmpty()) {
        mediaURL = s.getMediaEntities()[i].getVideoVariants()[0].getUrl();
        //System.out.println(mediaURL);
      } else {
        mediaURL = s.getMediaEntities()[i].getMediaURL();
        //System.out.println(mediaURL);
      }

      media.add(mediaURL);
    }

    return media;
  } else {
    ArrayList<String> returnArr = new ArrayList<String>();
    returnArr.add("Text Only");

    return returnArr;
  }
}
```

This function loops through the media entities in a status using the `getMediaEntities()` function in the tweet passed through as Status s. It then checks to see if the current media entity being looped over is a video using by checking if `getVideoVariants()` returns an empty array to determine if the media currently being checked over has a video variants array. If it does not then it is not a video so can be retrieved using `s.getMediaEntities()[i].getMediaURL()`. Otherwise it retrieves the URL using the `getURL()` function on the first video variant.

Once these URLs are retrieved all of the Status objects are turned into objects of the Tweet class which I wrote. As can be viewed in the TwitterFaceFilterBot Github repository. This object is what is used to structure the final JSON Structured data that the REST API returns.

The Tweet class consists of a simple constructor that builds a basic object which holds the data from each tweet that the bot may need. This data comes down to the following:

 - Text content.
 - Text content with the link to the tweet removed. (When the Twitter API returns a tweet with media it returns the tweet with a shrunken link to the actual tweet attached to the end of the text content). This link is removed using the following piece of code which simply returns the substring of the text up until the start of "https://".
 ```
 public String removeLink(String tweet) {
      try {
        return tweet.substring(0, tweet.indexOf("https://")).trim();
      } catch (IndexOutOfBoundsException e) {
        return tweet;
      }
 }
 ```
 -The display name of the person who tweeted @FaceFilterBot
 -The username of the person who tweeted @FaceFilterBot
 -The ArrayList of mediaURLs found in the tweet by the getMediaURL function discussed earlier.

The following set of functions are the functions used in converting the Status Objects into Tweet Objects. It is pretty self explanatory however, a basic overview is the statusToTweet function creates and returns a tweet object for every status passed in through the status parameter. the `getMediaType(mediaURLs.get(0))` function is a simple function I wrote which simply finds out the type of media the tweet includes by checking the first URL in the mediaURLs ArrayList and checks for a bunch of different file extensions within the URL to determine whether the bot is dealing with photos, videos, or no media. Plain text (no media) is returned as 0, where as photos and videos are 1 and 2 respectively. It checks if there is any media: `if (_mediaType == 0)` and if so it creates a Tweet object with null a null mediaLinks Object. Otherwise it creates a Tweet object containing the necessary mediaURLs.

```
public Tweet statusToTweet(Status status) {
   ArrayList<String> mediaURLs = getMediaURL(status);
   int _mediaType = getMediaType(mediaURLs.get(0));

   Tweet tweet;
   if (_mediaType == 0) {
     tweet = new Tweet(status.getText(), status.getText(), status.getUser().getScreenName(), status.getUser().getName());
   } else {
     tweet = new Tweet(status.getText(), removeLink(status.getText()), status.getUser().getScreenName(), status.getUser().getName(), getMediaURL(status));
   }

   return tweet;
 }
```

`statusesToTweets()` takes every single status retrieved from the twitter API and converts them into the Tweet Object using the `statusToTweet()` and returns an `ArrayList<Tweet>` containing all the new Tweet objects. `mostRecentStatusToTweet()` converts just the most recent status received or the one in the first index of the retrieved statuses.


```
 public ArrayList<Tweet> statusesToTweets() {
   ArrayList<Tweet> tweets = new ArrayList<>();
   for (Status status: statuses) {
     Tweet tw = statusToTweet(status);
     tweets.add(tw);
   }
   return tweets;
 }

 public Tweet mostRecentStatusToTweet() {
  return statusToTweet(statuses.get(0));
 }
```
