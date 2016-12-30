---
layout: single
title:  "Tutorial: Setting Up a Twitter Bot Using Python and Tweepy"
date:   2016-11-06 20:42:06
categories: 
- programming 
- python
- tutorial
tags: 
- bots
- twitter
- tweepy

---

In my previous [post about virtualenv and virtualenvwrapper](http://ageof.info/Setting-Up-Python3-On-Linux), I used the Tweepy module in the basic demonstration of how virtual environments work. Building off of that, I want to now create a tutorial for making a basic Twitter bot using Tweepy!

## Prerequisites

Beyond basic programming and an installation of Python 3.4.2 (or later), the following code should be viable for any system running on it.

## The Repo

I made a [repo on Github with code that is ready to go](https://github.com/ejmg/demoTwitterBot) with the bot this tutorial will show you how to use. Feel free to fork it, clone it, or just save it directly as a zip file (if you don't know how to use Git) on your PC. 

# Why Tweepy

Twitter bots, for those who don't know, are semi to fully automated scripts that engage with Twitter's REST or streaming Application Programming Interface (API). [Tweepy](https://github.com/tweepy/tweepy) is what you would call an "API wrapper," a set of commands and functions that makes interacting with Twitter's API a lot easier and more "human friendly." 

Why would you want to do that? Is Twitter's API really that bad? Well, it's not that Twitter's API is bad, but that Twitter has to make an API that's incredibly flexible and detailed. They want to make a product that can do just about anything via API interaction alone but this comes at the cost of being "easy" to interact with on a more than one-off basis. Here's an example of how to send a tweet using nothing but the API itself by posting a POST request:

```python
request.post(https://api.twitter.com/1.1/statuses/update.json?status=Maybe%20he%27ll%20finally%20find%20his%20keys.%20%23peterfalk)
```

This post request makes a tweet saying "Maybe he'll finally find his keys. #peterfalk" to the account it's authorized for. Now herein lies another problem: this post request isn't too messy, but this is only after setting up an OAuth2 authentication which verifies your identity and proves you can tweet from an account. An authorization using OAuth2 looks something like this (taken from [python-oauth2's repo](https://github.com/joestump/python-oauth2/wiki/Signing-A-Request)):

```python
params = {
    'oauth_version': "1.0",
    'oauth_nonce': oauth.generate_nonce(),
    'oauth_timestamp': str(int(time.time())),
    'user': 'joestump',
    'photoid': 555555555555
}

# Set up instances of our Token and Consumer. The Consumer.key and 
# Consumer.secret are given to you by the API provider. The Token.key and
# Token.secret is given to you after a three-legged authentication.
token = oauth.Token(key="tok-test-key", secret="tok-test-secret")
consumer = oauth.Consumer(key="con-test-key", secret="con-test-secret")

# Set our token/key parameters
params['oauth_token'] = token.key
params['oauth_consumer_key'] = consumer.key

# Create our request. Change method, etc. accordingly.
req = oauth.Request(method="GET", url=url, parameters=params)

# Sign the request.
signature_method = oauth.SignatureMethod_HMAC_SHA1()
req.sign_request(signature_method, consumer, token)
```

Okay, now that's a bit more nasty. Now, if you want to do something that's automated, the code above isn't much of a problem, but that still leaves that post request earlier. Formatting a string https url like that can be problematic, especially if you don't have your code prepared for each type of media or text character (emojis? different languages?). Better yet, since you are programming (whether veteran or not), you want to be lazy, right? Simply put, why do more work when the work has already been done for you by someone else? 

Enter Tweepy: tweepy is a module that makes interacting with Twitter's API much easier than the stuff above. In fact, it makes it a total breeze and that's why I'm going to show you how to make a bot using it.

# How to Make a Twitter Bot

First, we have to set up your account (or make a new account, if you want) so that it can give permission to your bot to tweet. If you are okay with using your current account and don't want to learn what twitter aliases are, skip to step two.

## Step 0: Set up an account

Rather than making a separate email to make a new twitter account, use an **account alias.** This saves you the trouble of having to create another email account (which has its own password), and managing which twitter accounts go with which email, and instead simplifies your bots' email to a simple alias of yours. Say your email is `example@example.com`. To make an alias with Twitter, go to their [sign up page](https://twitter.com/signup) like you would normally, and use your email with `+someAliasHere` as the email you use as shown:

![signup]({{ site.url}}/assets/images/posts/2016-12-28/signup.png)

From there, setup your twitter account as desired (accounts you want to follow, AVI, bio, settings, etcetera). You should receive a notification in your email account saying you created a twitter account as if it was your first twitter. Congrats!

## Step 1: Give App Permissions

Now that you have an account you can use to run a bot on, you have to authorize it so that the bot can use it. Make sure you are logged in with the account you want to run the bot on and go to [https://apps.twitter.com/](https://apps.twitter.com/). You should immediately see a page saying you don't have any apps and a button asking if you want to create one. Click it and fill it in with the information you want (none of this really matters for a simple bot). For my demo bot which I ran on my main account, I filled it in as follows:

![create-app]({{ site.url }}/assets/images/posts/2016-12-28/create-app.png)

Accept the user agreement and click okay. The page will immediately change and present you with your app dashboard. Switch over to the tab called `keys and access tokens`, where you will find the authorization information your bot will need. This comes in the form of `Consumer Key`, `Consumer Secret`, `Access Token`, and `Access Token Secret`. To get the latter two, you will have to click the button at the bottom of the page to generate them (you will not have them by default, don't worry when you don't see them). I've boxed in red the relevant information below:

![auth-info]({{ site.url}}/assets/images/posts/2016-12-28/auth-info.png)

In case it isn't obvious by the picture and the warning Twitter gives you, **do not share these keys with anyone**. While the non-secret keys/tokens are technically okay to know publicly, it doesn't hurt to be on the safe side. Now, keep this page open or save these codes somewhere as you will need them in just a moment. **ALSO**, your access token may have a dash in it - **don't** delete it. It was intentional and is a part of the token itself. Just copy and paste the info as given.

## Step 2: Hello World!

If you haven't forked/cloned/zip'd my [`demoTwitterBot` repo from Github yet](https://github.com/ejmg/demoTwitterBot), I highly recommend you do. It has code that is ready to deploy so that once you create your `secret.py` file in a moment, you are able to run the bot. Even if you plan on going line by line with me in this demo, it would help to have my code just in case something doesn't work on your end outside of something not being right in your development environment. It also has commentary throughout it to help you understand what is happening so that you don't ultimately need to use this guide when you go back to look at it!

### Step 2.0: Install tweepy

Making sure to use Python 3.4.2 or newer, create a virtualenv (or don't, but you should) and install tweepy using `pip3 install tweepy`. If you are using my repo, you can also just use `pip3 install -r requirements.txt`, but since it's only one module in this case, it really doesn't make a difference. It should only take a few seconds for the module to install and your terminal should show output confirming its installation:

![tweepy]({{ site.url}}/assets/images/posts/2016-12-28/tweepy.png)

### Step 2.1: Create a `secret.py` file

Remember those authorization tokens I told you to save earlier? We're going to need them in order to twee with your bot. Make a `secret.py` file in your working directory and insert `Consumer Key` as `CONSUMER_KEY`, `Consumer Secret` as `CONSUMER_SECRET`, and so forth as shown below with X's in place of the codes given:

![secret]({{ site.url }}/assets/images/posts/2016-12-28/secret.png)

Save the file and move on. If you are using Github (and you didn't fork/clone my repo and your repo isn't private) you should make a .gitignore file with `secret.py` in it. You wouldn't want the world to see your tokens!

### Step 2.2: Create `bot.py`

Create a `bot.py` file and add the following lines of code:

```python
from secret import ACCESS_SECRET, ACCESS_TOKEN, CONSUMER_KEY, CONSUMER_SECRET
import tweepy as ty
import random
```
What these lines do is the following:

1. Import the variables you declared in `secret.py` which we did in order to not reveal their contents in the following code
2. Import the `tweepy` module as `ty`, an alias that just allows you to write shorter code when calling `tweepy` modules
3. Import the random module from Python's standard library, a pseudo random number generator,  which I'll explain why in just a moment.

Speaking of #1 and #2, let's now make a method that authorizes your bot via `tweepy`. Write the following code:

```python
def setTwitterAuth():
    auth = ty.OAuthHandler(CONSUMER_KEY, CONSUMER_SECRET)
    auth.set_access_token(ACCESS_TOKEN, ACCESS_SECRET)
    api = ty.API(auth)
    return api
```

What this code does is simple: we create a function with `def`, named it `setTwitterAuth()`, and tell it to return a single variable `api`. From within this function, we use `tweepy`'s `OAuthHandler()` function to tell Twitter who you are and `set_access_token()` to give authorization of your app to use your account with read/write privileges. With this access set, we then use the `auth` object created to create an `api` object with `API()`, which from here on is a key part of your bot as it allows you access to all of the API's various functions.

We are really close to finishing the first portion of your bot! The next thing we need to do is make a simple function to say Hello World with! While you don't have to make this a function, I did for demonstration purposes. Type the following:

```python
def tweetHelloWorld(api):
    api.update_status("This is an automated tweet"
                      " using a bot! Hello, World! #{}"
                      .format(random.randint(0, 10000)))
```

What you just did was create another function, `tweetHelloWorld()` which takes your `api` object as an argument. With it, you perform the most fundamental action with Twitter's API...you tweeted(!) with the `update_status()` method by passing it a string. Notice how I formatted the string to include a random number from 0 to 10000? I did this because you may end up testing this bot a few times and Twitter blocks duplicate tweets from within a certain time window. This will (reasonably) make sure you don't get a tweet blocked while testing it out. You will see this done with every tweet from here on but feel free to exclude it.

Finally, let's make your main driver for the script. Type the following:

```python
if __name__ == "__main__":
    api = setTwitterAuth()
    tweetHelloWorld(api)
```

That's it! Assuming everything is configured right on your end, save your file and tweet using your bot with the following command:

`python3 bot.py`

Test it out!

![helloworld]({{ site.url}}/assets/images/posts/2016-12-28/helloworld.png)

## Step 3: Retrieving Account Information and Searching Your Tweets

Congrats on your first bot tweet, but what else can we do? Turns out there's a lot, which is why I highly recommend you [read Tweepy's documentation](read Tweepy's documentation). That said, I'll introduce you to a few more concepts. Add the following to your bot and run it again:

```python
if __name__ == "__main__":
    api = setTwitterAuth()
    tweetHelloWorld(api)
    user = api.me()
    print(user)
```

What we just did is retrieve the `user` object associated with the `api` object we authenticated, aka your bot's account. There's a lot of interesting info stored within this object which is why I had it printed to your terminal, which you should totally check out. Here's what mine looks like:

![user]({{ site.url}}/assets/images/posts/2016-12-28/user.png)

There are a lot of fields within your `user` object, such as your background image url, your last tweet, your bio, your user image url, whether you have 2FA enabled (whoops), and more. For now, let's checkout your screen_name by entering the following into your driver:

```python
    print(user.screen_name)
```

This should print out into your terminal your "user handle", the unique name your account has with the infamous @ symbol with Twitter. Notice, however, that Twitter does not include the actual "@" with your `screen_name`. You should keep this in mind for when you try to @ people with your bot (as we will do later).

Now add this into your driver:

```python
    api.update_status("I have {} followers and follow {} accounts! #{}"
                      .format(user.followers_count, user.friends_count,
                              random.randint(0, 10000)))
```

Your bot should now tweet again with the number of followers you have and the number accounts you follow by using the `followers` and `friends_count` fields of your `user` object. Neat! But let's go a step further. Let's use your `user` object to pull a tweet from your own timeline. Let's make another function by typing the following:

```python
def getTimeline(api, user):
    tweets = api.user_timeline(user.screen_name, count=100)
    print(tweets[0])
    return tweets
```

Go ahead and add this line to your driver as well:

```python
    getTimeline(api, user)
```

This function is simple: it uses the `user` object given to retrieve the last 100 tweets from its timeline using tweepy's `user_timeline()` function. We passed it your `user.screen_name` so it knew which timeline to look at and told it to only get 100 tweets using `count=100`. The tweets are returned to us in the form of a normal Python list which makes looking at them pretty easy. I have your print one to terminal so you can see the fields it holds. Like a `user` object, it contains a lot of information worth looking at if you want to do more work with Twitter's API. Now that we have a list of your last 100 tweets, let's do something with them. Create the following function:

```python
def getLastTweet(api, user):
    api.update_status("My last tweet is as"
                      " follows...#{}".format(random.randint(0, 10000)))
    tweets = getTimeline(api, user)
    tweets = [tweet.text for tweet in tweets]

    try:
        if len(tweets) == 0:
            api.update_status("This shouldn't be possible...#{}!"
                              .format(random.randint(0, 10000)))
        else:
            tweet = tweets[0]
            api.update_status(tweet[:14] + "...wait, my last tweet was about"
                              " tweeting my last tweet, do'oh! #{}".
                              format(random.randint(0, 10000)))
    except ty.RateLimitError:
        print("You've hit the API limit! Try your bot in about an hour.")
    except ty.TweepError as e:
        print("You've hit another error. This could be a lot of things, but "
              "I'll leave that to you to debug. The error is {}".format(e))
```

As before, add the following to your driver:

```python
    getLastTweet(api, user)
```

This may look like a lot of code but it really isn't. Here's what you are doing:

1. You tweet "My last tweet is as follows..."
2. You then call the `getTimeline()` method you just typed earlier, retrieving the 100 last tweets from your timeline
3. You then do a list comprehension, taking only the `text` field from each tweet and remaking the list
4. You then enter a try-catch block which contains the following:
    A. An if-else block which checks to the size of the list and tweets if it isn't empty
    B. an tweepy.RateLimitError catch block, which catches the error and tweets to terminal that you've gone over the API limit for the given hour
    C. A general tweepy.TweepError catch block, which catches a multitude of errors and thus just prints the error type.
    
In case you haven't already noticed, by tweeting "My last tweet is as follows..." before retrieving your last 100 tweets via `getTimeline()`, your last tweet, by default, is that tweet just given ;) Furthermore, I put in the try-catch block to show you the built in error capabilities of tweepy. Rather than just leaving you to your own devices, tweepy has some built in data types that help you figure out what is going on with your code and mitigate a problem if detected. Finally, if you didn't already, you now know what a try-catch block looks like. Congrats!

## Step 4: Search Twitter and Replying to Tweets

Good job on making it this far, we are almost done. Before I let you go, let's try two more things: searching twitter and replying to tweets. Type of the following, first another method and then the following line into your driver:

```python
def searchTweet(api, searchTerm):
    searchResults = [status for status in ty.
                     Cursor(api.search, q=searchTerm).items(100)]
    return searchResults
```

```python
    searchResults = searchTweet(api, "\"Hello World \"")
```

What we just did was simple: We searched Twitter for the explicit string "Hello World" and used the built in `Cursor` iterator in the tweepy module. Let's break it down:

First off, notice how I escaped the quotes around "Hello World" so that when you searched it, it wasn't just `Hello World` but instead was `"Hello World"`, get it? without using the paired `\"`, you would look up all tweets that contained any combination of words so long as it had `hello` and `world` somewhere in the tweet. By searching `"Hello World"` instead, you look for only the tweets which have those two words consecutive somewhere within the tweet.

Secondly, I used a special iterator type made by tweepy's creators. This iterator, `Cursor`, allows you to look up large quantities of tweets once in a cohesive and easily manageable way. While it isn't obvious with the demonstrated code, if you look more into mining Twitter, you will see the benefits. [Again, read the tweepy docs for more info](http://docs.tweepy.org/en/v3.5.0/index.html)! After we retrieve the results using that Cursor, however, I go ahead and turn the iterated tweets into a simple list once again. In fact, I do it all at once with a single list comprehension if you look closer.

Almost done! Now let's take a random tweet from those search results and tweet back! Type up the following as before, first a method and then a line for your driver:

```python
def replyHelloWorld(api, searchResults):
    randomTweet = searchResults[random.randint(0, len(searchResults) - 1)]
    tweet = ("@{} This is a demo search for 'hello world' with a bot, hello"
             " world! #{}".format(randomTweet.user.screen_name,
                                  random.randint(0, 10000)))
    tweetID = randomTweet.id
    api.update_status(tweet, tweetID)
```

```python
    replyHelloWorld(api, searchResults)
```

You probably can read the code by now, but just to be clear: You take a random tweet from the search results list, you then make a string, `tweet`, that is composed of the tweet's message, you retrieve the unique `id` of that tweet, and then you respond to that tweet using the `update_status()` method but with the added argument of `tweetID` just mentioned. What we did then was create our message as desired and then tweet it at a specific tweet rather than just on your plain old timeline. This is also the difference between just @'ing someone out of nowhere and responding to **a specific tweet**, which this function should of done. Some times it doesn't work, however, because when you use search, it doesn't just look at tweets - it also looks at individuals bios and quoted tweets. Try it out yourself and see what I mean!

## Step 5: Experiment!

Congrats, you did it! If everything ran as expected, every time you run the bot, you should get a string of tweets like these:

![tweets]({{ site.url }}/assets/images/posts/2016-12-28/tweets.png)

Now that you have your bot running, you can experiment. I tried to modularize my code to what extent it could be done, so go ahead and tweak at it and see what happens. Try searching for different terms, looking up different information, see how many tweets you can pull at once, and, hell, figure out what the Twitter API's limit is! ;) 

I hope you enjoyed this guide. If you have any input, edits, or just want to say hi, feel free to email me or @ me on twitter, preferably with your bot!
