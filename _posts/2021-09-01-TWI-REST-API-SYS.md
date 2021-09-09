---
layout: post
title: Twitter Face Filter Bot REST API System
---

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
