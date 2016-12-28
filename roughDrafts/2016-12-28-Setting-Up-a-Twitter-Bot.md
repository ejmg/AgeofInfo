---
layout: single
title:  "Tutorial: Setting Up a Twitter Bot Using Python and Tweepy"
date:   2016-11-06 20:42:06
categories: 
- programming 
- linux
- python
- tutorial
tags: 
- bots
- twitter
- tweepy
- github

---

In my previous [post about virtualenv and virtualenvwrapper](http://ageof.info/Setting-Up-Python3-On-Linux), I used the Tweepy module in the basic demonstration of how virtual environments work. Building off of that, I want to now create a tutorial for making a basic Twitter bot using Tweepy!

# Why Tweepy

Twitter bots, for those who don't know, are semi to fully automated scripts that engage with the Twitter's REST or streaming Application Programming Interface (API). Tweepy is what you would call an "API wrapper," which basically makes interacting with Twitter's API a lot easier and more "human friendly." Why would you want to do that? Is Twitter's API really that bad? Well, it's not that Twitter's API is bad, but that Twitter has to make an API that's incredibly flexible and detailed. They want to make a product that can do just about anything via API interaction alone but this comes at the cost of being "easy" to interact with on a more than one-off basis. Here's an example of how to send a tweet using nothing but the API itself by posting a POST request:

```python

request.post(https://api.twitter.com/1.1/statuses/update.json?status=Maybe%20he%27ll%20finally%20find%20his%20keys.%20%23peterfalk)

```

This post request posts a tweet saying "Maybe he'll finally find his keys. #peterfalk" to the account it's been authorized for. And herein lies the problem: this post request isn't too messy, but this is only after setting up OAuth2 authentication which verifies your identity and proves you can tweet from an account. An authorization using OAuth2 typically looks something like this (taken from [python-oauth2's repo](https://github.com/joestump/python-oauth2/wiki/Signing-A-Request)):

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

Okay, now that's a bit more nasty. Now, if you want to do something that's automated, the code above isn't much of a problem, but that still leaves that post request earlier. Formatting a string https url like that can be problematic, especially if you don't have your code prepared for each type of media. Better yet, why not just avoid the trouble to start with? 

