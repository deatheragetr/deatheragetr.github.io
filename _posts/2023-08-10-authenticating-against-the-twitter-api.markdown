---
layout: post
title:  "Authenticating OAuth v1.0a against the Twitter API"
date:   2012-08-10 14:10:00 -0400
permalink: /authenticating-twitter-api-oauth-v1a
categories: twitter oauth signatures
---

# Twitter's Deprecated `POST /1.1/statuses/update.json`` Endpoint and a Workaround

Recently, Twitter integration spontaneously ceased to work in one of my apps.  Specifically, the ability to post new tweets.  Turns out the `POST /1.1/statuses/update` endpoint was deprecated in June without warning, as [others have also discovered](https://stackoverflow.com/questions/76352378/why-does-twitter-api-return-the-error-if-you-need-access-to-this-endpoint-you).  Specifically, the error I was was witnessing:

```
403 Forbidden
You currently have access to Twitter API v2 endpoints and limited v1.1 endpoints only. If you need access to this endpoint, you may need a different access level. You can learn more here: https://developer.twitter.com/en/docs/twitter-api/getting-started/about-twitter-api#v2-access-leve
```

The [`twitter` gem](https://github.com/sferik/twitter) I'm using has yet to be patched. (And probably won't for a while, given the dearth of recent activity).  And near as I could tell, there were no existing gems that supported the v2 `POST /2/tweets` endpoint with OAuth 1.0a.  So I made a quick workaround.  The most difficult part here -- and the advantage of using the `twitter` gem -- is the convoluted OAuth signature.  

```ruby
require "simple_oauth"
require "httpparty"

header = SimpleOAuth::Header.new(
  :post, 
  'https://api.twitter.com/2/tweets',
  { }, # Empty, see note*
  { 
    consumer_key: consumer_key, 
    token: access_token, 
    consumer_secret: consumer_secret, 
    token_secret: access_token_secret,
    signature_method: 'HMAC-SHA1'
  })

response = HTTParty.post(
  'https://api.twitter.com/2/tweets',
  body:  { 
   text: 'Hello world'
  }.to_json, 
  headers: {
   'content-type': 'application/json',
    authorization: header.to_s
})

```

*Note that unlike the Twitter API documentation for [authorizing requests with url-encoded params](https://developer.twitter.com/en/docs/authentication/oauth-1-0a/authorizing-a-request), the request body params for `application/json` type requests should _not_ be included when generating the OAuth signature.  You'll get a 401 if you do.

That's it.