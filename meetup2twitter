#!/usr/bin/ruby

require 'rmeetup'
require 'twitter'

CONFIG = YAML.load(File.read("m2t.yml"))

group_url = CONFIG["MEETUP_GROUP_URL"]
twitter_username = CONFIG["TWITTER_USERNAME"]
list_name = CONFIG["LIST_NAME"]

RMeetup::Client.api_key = CONFIG["MEETUP_API_KEY"]

Twitter.configure do |config|
  config.consumer_key       = CONFIG["TWITTER_CONSUMER_KEY"]
  config.consumer_secret    = CONFIG["TWITTER_CONSUMER_SECRET"]
  config.oauth_token        = CONFIG["TWITTER_OAUTH_TOKEN"]
  config.oauth_token_secret = CONFIG["TWITTER_OAUTH_TOKEN_SECRET"]
end

group_members = RMeetup::Client.fetch(:members,{:group_urlname => group_url})
group_usernames = []

group_members.each do |member|
  if member.other_services.has_key?("twitter")
    group_usernames << member.other_services["twitter"]["identifier"][1..-1]
  end
end

group_usernames.sort!

list_members = Twitter.list_members(twitter_username, list_name)
list_usernames = list_members.collection.collect {|user| user.screen_name}.sort

while (!list_members.last?)
  list_members = Twitter.list_members(twitter_username, list_name, {:cursor => list_members.next_cursor})
  list_usernames.concat list_members.collection.collect {|user| user.screen_name}.sort
end

add_usernames = group_usernames - list_usernames
remove_usernames = list_usernames - group_usernames

puts "Meetup Group Member Count: #{group_members.length}"
puts "Meetup Group Twitter Users: #{group_usernames.length}"
puts "Current Meetup Members: #{group_usernames}"
puts "Twitter List Member Count: #{list_usernames.length}"
puts "Current Twitter Users: #{list_usernames}"
puts "Users to add: #{add_usernames}"
puts "Users to remove: #{remove_usernames}"

Twitter.list_add_members(twitter_username, list_name, add_usernames) if !add_usernames.empty?

remove_usernames.each do |username|
  Twitter.list_remove_member(twitter_username, list_name, username)
end

puts Twitter.rate_limit_status.remaining_hits.to_s + " Twitter API request(s) remaining this hour"

