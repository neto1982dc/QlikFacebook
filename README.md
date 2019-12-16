##QlikFacebookAnalyser

###Version 1.1

This app has been updated to bring in more likes and comments on your posts. It will now bring up to 200 likers or comments per post as opposed to 25
in the previous version.

It will also now ignore errors returned from the Facebook API - sometimes there is a specific post that causes an error when returned through the Facebook API. The
app will now load all posts up to that point, then continue with the next load of data, rather than simply exiting the reload process.

##Project details

A Qlik Sense app to analyse your Facebook posts

This app allows you to quickly load in your own Facebook data and analyse it. In order to use it, you’ll need the Qlik REST connector, a Facebook developer account and two extension objects, then you’ll just need your access token and to update the connections to use your own token. Full instructions are in the script. Further details on how this was put together and how to use the REST connector with the Facebook API can be found below.

Note: access tokens expire after 2 hours, so if you try to reload after this period, you’ll get an error and need to update the connections with a new token.

##Required connectors and extensions

Qlik REST Connector: You can find this and other Qlik connectors here - https://community.qlik.com/docs/DOC-8892

Media Box: http://branch.qlik.com/projects/showthread.php?434-Media-Box-for-Qlik-Sense&highlight=media+box 

qWidget: https://community.qlik.com/docs/DOC-7534

If you have issues importing the extensions in Qlik Sense 2.1.1 you may need to use the MIME type fix - http://branch.qlik.com/projects/showthread.php?648-qrs-mime-Add-mime-types-to-QRS&highlight=mime

##Project Overview

I’ve been playing with the REST connector and Facebook APIs for a few weeks and thought I’d share my findings, so that anyone else trying to use it doesn’t have to make all the same mistakes again. If you’d just like to see what your own Facebook data looks like, I have created an app that can be quickly configured to work with your own data.

###REST Connector documentation.

Firstly, the documentation for the REST connector is pretty good and the instructions on how to set up a developer account and get connected to Facebook are excellent. Follow these steps to get your Facebook Developers account set up.

http://help.qlik.com/Connectors/en-us/connectors/#../Subsystems/REST_connector_help/Content/1.0/Create-REST-connection/Connection-examples.htm

The description of how the pagination works is correct, but things can get a little fiddly with how Facebook actually works. Pagination allows the connector to step through all the pages of data that Facebook returns, effectively clicking the 'next' button until no more data is returned. REST APIs can do this in a variety of different ways, for the Facebook post data, it uses the Next URL method (i.e. there is a field in the data returned that explicitly states the URL to get the next load of data).

http://help.qlik.com/Connectors/en-us/connectors/#../Subsystems/REST_connector_help/Content/1.0/Create-REST-connection/Pagination-scenarios.htm%3FTocPath%3DQlik%2520Sense%2520connectors%7CREST%7C1.0%7CConnect%2520to%2520a%2520REST%2520data%2520source%7C_____4

###What I wanted to achieve

I wanted to create a single Qlik Sense app that brought together as much information about what I share on Facebook as possible. I wanted to bring in every Facebook post, that I made, along with the associated information, who liked it, who commented, what the comment was, any URLs shared, who I mentioned in posts, what the privacy setting of the post was etc.
With the Facebook API, it’s possible to create a single query fairly easily to bring all the information back, however, the amount of data returned gets fairly large and it seems to cause issues for the REST connector in certain circumstances.
What I learned about the REST connector

###Consistency

I guess the most important lesson learned is that the REST connector expects consistency in the data. This is especially important when using the paging parameter. If the data returned by the second, or any other page of the data is not in the same format, with the same fields, it will cause the connector to error.
This is the issue when returning large amounts of data. On some pages, there may not be likes for particular posts, or a single post may have a very large amount of comments, which seems to cause some issues in the connector. The simple way around this is to run multiple queries on Facebook, rather than trying to return all data in a single call.
I ended up running 7 queries to return all the data I wanted. The first to bring back the post details, along with pictures, URLs etc. shared (i.e. anything with a 1:1 relationship with the post), then I created separate queries to bring back the items that have multiple values per post, i.e. likes, comments etc.

###Consistency part 2

Another issue I ran in to is that, if you use the Facebook API explorer to create the query, it seems to make the first page of results structurally different to the next and remaining pages. On the first page it would have an element in the JSON for ‘posts’, but on subsequent pages of data, this element was missing, which caused the connector to error.
The simple workaround for this was to use the pagination links at the bottom of the explorer to move to the next page, then back to the first. This then presented a common format for the data through the complete dataset.

###Query Parameters

The documentation suggests using the query parameters to enter the access token for the API. The issue with this is that in the ‘next url’ field, Facebook embeds the required token. By placing the access token in the query parameters and using the URL paging method, the access token is repeated in the URL, which causes an error in the Facebook API. The solution for this is to simply hard code the first URL with the access token embedded.
Example Facebook API queries that work with the REST connector (note, you will need to add your own access token to the end of each URL)
With all of these examples, choose ‘Next URL’ as the pagination type and root/paging/next as the paging path.

**Posts and associated details**

`https://graph.facebook.com/v2.5/me/posts?fields=message,story,created_time,link,full_picture&limit=25&format=json&access_token=`

**Likes and likers of posts**

`https://graph.facebook.com/v2.5/me/posts?fields=likes%7Bname%7D&limit=25&format=json&access_token=`

**Comments and commenters**

`https://graph.facebook.com/v2.5/me/posts?fields=comments&limit=25&format=json&access_token=`

**Places (check ins)**

`https://graph.facebook.com/v2.5/me/posts?fields=place&limit=25&format=json&access_token=`

**‘With’ tags**

`https://graph.facebook.com/v2.5/me/posts?fields=with_tags&limit=25&format=json&access_token=`

**Message tags**

`https://graph.facebook.com/v2.5/me/posts?fields=message_tags&limit=25&format=json&access_token=`

**Post Privacy**

`https://graph.facebook.com/v2.5/me/posts?fields=privacy&limit=25&format=json&access_token=`
