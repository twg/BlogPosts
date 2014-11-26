# Pulling Down a User's Twitter Feed

On a recent project we needed to create a news feed that mixed our client's blog posts, twitter feed, and instagram posts into one unified timeline. After building out the timeline feature, I realized that we still needed to import our client's historical Twitter data – something that I've never done before.

It turned out to be a fairly simple task thanks to the awesome [Twitter gem](https://github.com/sferik/twitter), but I thought I would document it here anyway in the hopes that it could save another developer time.

### A Couple Notes...

* Twitter only allows you to pull down approximately 3200 tweets
* You'll need to visit [apps.twitter.com](https://apps.twitter.com/) and get your credentials setup
* This code returns an array of JSON encoded tweets (`tweet.attrs.to_json`) 
* We are not considering the API limitations of Twitter, but this could be easily adapted using the example outlined [here](https://github.com/sferik/twitter/blob/master/examples/AllTweets.md)
* Feel free to comment or contribute to the Gist posted [here](https://github.com/sferik/twitter/blob/763eaf69fcd158f83fc838c5ebe0306e6937e4bc/examples/RateLimiting.md)

### The Code

```ruby
require 'twitter'
 
CONFIG = {
  consumer_key:        "YOUR_CONSUMER_KEY",
  consumer_secret:     "YOUR_CONSUMER_SERCRET",
  access_token:        "YOUR_ACCESS_TOKEN",
  access_token_secret: "YOUR_ACCESS_TOKEN_SECRET"
}
 
class AllTweets
 
  attr_accessor :tweets
 
  def initialize(username)
    @username = username
    @tweets = []
    @max_id = nil
    self.get_tweets
  end
 
  def client
    Twitter::REST::Client.new do |config|
      config.consumer_key        = CONFIG[:consumer_key]
      config.consumer_secret     = CONFIG[:consumer_secret]
      config.access_token        = CONFIG[:access_token]
      config.access_token_secret = CONFIG[:access_token_secret]
    end
  end
 
  def get_tweets
    options = {count: 200}
    options[:max_id] = @max_id if @max_id
    fetched_tweets = client.user_timeline(@username, options)
    fetched_tweets.each do |tweet|
      if @max_id.nil? || @max_id >= tweet.id
        @max_id = tweet.id - 1
      end
      @tweets.push(tweet.attrs.to_json)
    end
 
    if fetched_tweets.length == 0
      puts "Done, fetched #{@tweets.length} tweets"
      return @tweets
    else
      puts "Tweet Count: #{@tweets.length}"
      get_tweets
    end
  end
 
end
 
AllTweets.new('twg').tweets
#=> Returns an array of ~3200 JSON encoded tweets
```