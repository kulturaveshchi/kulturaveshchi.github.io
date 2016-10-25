---
layout: post
title: Designing the data
---

The first step in thinking about the project was setting up the database schema, 
which in turn involved thinking about various features of the design of the 
overall system.

## The easy bit: Terms and collections

The point of the whole system is to make sequential tweets based on a sequence
of words. So the two straightforward parts of the database of the scheme are
a table for the terms, and one for the collections. The idea is to post tweets
based on the terms in order, to each term has an integer index (for the 
"everyword" case, where the terms are words in dictionary order, this index 
isn't necessary, because the order is inherent to the terms themselves, but
I assume the point of having multiple collections is to allow for collections
other than dictionaries, where the best order of the terms wouldn't necessarily
be alphabetical).

## Bots

The definition of a bot is also reasonably straightforward. A "bot" tweets
from a given collection, on a particular schedule. So there's a "bots" table that
stores this frequency and a template for tweets, and references a collection, as 
well as storing the index of the last term that was tweeted and the time of that
last tweet. 

I'm also going to store in the bots table an "update in progress" flag, which
will be set when a particular bot is selected to be updated by the posting 
process, and cleared once that update has been processed (if the update succeeds,
the "last updated" and "last term posted" fields will also be updated, otherwise
they will not, and so the update will be retried). I'm doing this to avoid
duplicate updates. I envision doing the actual updates on a background thread,
with another thread periodically waking up to add more updates to a queue as
they become due. So it might be the case that an update is still waiting to
be processed, or in process, when the thread wakes up to select new updates.

Using the database to coordinate like this seems kind of hacky, but I can't 
think of a better alternative. There are various job queue libraries like 
[resque](https://github.com/resque/resque), but I haven't seen one which has the 
functionality I want, of performing
a task repeatedly with a given frequency. Another alternative would be to
update the "last updated" field of the database as soon as a posting task is
kicked off, and have that task itself be responsible for retrying in the event
of failure. This would have the advantage that even if a particular worker
repeatedly fails, the bot would continue to process subsequent terms; but this
might not be the behaviour I want, because it would mean terms could be posted
out of order. I might rethink this when I come to implement the worker that
will actually be doing the posting.

The final thing the bot table contains is a reference to the details of a Twitter
account, which will be used to connect to Twitter to actually make the Tweet.

## Accounts and authentication

Because the point of the system is to post to Twitter accounts, I decided the
simplest thing to do was to delegate all authentication and authorization to
Twitter. Users will obviously have to log in with Twitter anyway to authorise
the system to post to the specified Twitter account. It would be possible for
me to maintain my own concept of account and allow users to associate one or 
more Twitter accounts with each account on everybot, but I decided that would
require additional work for no very clear payoff. 

The other issue to be decided upon is how to authenticate each request. I've 
decided to use a simple token-based authentication scheme, either in cookies
for the web-based interface, and in the Authorization header for API access. 
I'm planning on requiring https to access the site anyway (otherwise an attacker
could potentially access Twitter credentials during the OAuth login process), so
I don't need to use a challenge-response system to keep the credentials secret.
As we will already have a token conveniently supplied by Twitter, I'm planning
on re-using that token. This requires a little bit of plumbing in the model to
connect tokens (which are per-login) to accounts, but that's reasonably 
straightforward.

Next time: Designing the API.


